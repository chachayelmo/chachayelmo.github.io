---
published: true
title:  "[Programming] C++ 스마트포인터 원리"
excerpt: "C++에 대해 알아보기, SmartPointer"

categories:
  - Cpp
tags:
  - [C++, Cpp, SmartPointer]

toc: true
toc_sticky: true
 
date: 2022-12-19
last_modified_at: 2022-12-19
---

## 1. 스마트포인터 사용
### 1.1. 생성자

```cpp
#include <iostream> 
using namespace std; 
  
// C++11의 스마트포인터를 사용하려면 필요한 헤더 
#include <memory> 
  
int main() 
{ 
  // 1. explicit 생성자 
  std::shared_ptr<int> p1(new int);    // ok 
  //std::shared_ptr<int> p2 = new int;    // error 

  // 스마트 포인터 오버헤드 
  std::shared_ptr<int> sp1(new int); 
  std::shared_ptr<int> sp2 = sp1; 

  cout << sizeof(sp1) << endl; // 8

  // (A) 관리객체가 생성되는 오버헤드 
  // (B) 스마트 포인터 자체의 크기가 raw pointer의 2배 
}
```

### 1.2. 멤버 접근 방법

```cpp
#include <iostream> 
#include <memory> 
using namespace std; 
  
class Car 
{ 
public: 
  void Go() { std::cout << "Go" << std::endl; } 
}; 
  
int main() 
{ 
  std::shared_ptr<Car> sp1(new Car); 
  std::shared_ptr<Car> sp2 = sp1; 

  // -> 연산 : 대상체(Car)의 멤버에 접근 
  sp1->Go(); 

  // . 연산 : 스마트 포인터 자체의 멤버에 접근 
  int n = sp1.use_count(); 
  cout << n << endl; 

  Car* p = sp1.get(); // raw pointer 꺼내기 
}                     
```

### 1.3. 삭제자 함수

```cpp
#include <iostream> 
#include <memory> 
using namespace std; 
  
void foo(void* p) { free(p); } 
  
int main() 
{ 
  // 삭제자 변경 
  shared_ptr<int>  sp1(new int); 

  // 생성자의 2번째 인자로 삭제자 함수 전달 
  // shared_ptr<void> sp2(malloc(100), foo); 
  // shared_ptr<void> sp2(malloc(100), free); 
  shared_ptr<void> sp2(malloc(100), [](void*p) { free(p); }); 
    
  // 배열 할당일 경우 
  //shared_ptr<int> sp3(new int[10], [](int *p) {delete[] p; }); 
  // C++17 부터 아래처럼 사용가능 
  shared_ptr<int[]> sp4(new int[10]); 
}
```

### 1.4. 순환 참조 문제

```cpp
#include <iostream> 
#include <string> 
#include <memory> 
using namespace std; 
struct People 
{ 
  std::string name; 

  People(std::string n) : name(n) {} 
  ~People() { cout << name << " 파괴" << endl; } 

  shared_ptr<People> bf; 
}; 
  
int main() 
{ 
  shared_ptr<People> sp1(new People("kim")); 
  shared_ptr<People> sp2(new People("lee")); 

  // 메모리 누수 
  // 이 코드는 순환 참조 현상이 있습니다. 메모리가 해지 안됨. - 중요!! 
  sp1->bf = sp2; 
  sp2->bf = sp1; 
}
```

- 해결방안으로 shared_ptr 대신 그냥 포인터를 사용한다면?

```cpp
#include <iostream> 
#include <string> 
#include <memory> 
using namespace std; 
struct People 
{ 
  std::string name; 

  People(std::string n) : name(n) {} 
  ~People() { cout << name << " 파괴" << endl; } 

  //shared_ptr<People> bf; 
  People* bf; // raw pointer. 참조계수 증가 안함. 
              // 객체 파괴 조사 못함 
}; 
  
int main() 
{ 
  shared_ptr<People> sp1(new People("kim")); 
  { 
    shared_ptr<People> sp2(new People("lee")); 

    sp1->bf = sp2.get(); 
    sp2->bf = sp1.get(); 
  } 

  // sp2가 가리키는 객체는 파괴 되었지만 raw pointer인 bf를 가지고는 확인할 방법이 없다. 
  if (sp1->bf != nullptr) 
  { 
    cout << sp1->bf->name << endl; 
  } 
}
```

- 해결방안으로 weak_ptr을 사용하면?

```cpp
#include <iostream> 
#include <string> 
#include <memory> 
using namespace std; 
struct People 
{ 
  std::string name; 

  People(std::string n) : name(n) {} 
  ~People() { cout << name << " 파괴" << endl; } 

  //People* bf; 
  weak_ptr<People> bf; // 참조계수 증가 안함, 객체 파괴 조사 가능 
}; 
  
int main() 
{ 
  shared_ptr<People> sp1(new People("kim")); 
  { 
    shared_ptr<People> sp2(new People("lee")); 

    sp1->bf = sp2; 
    sp2->bf = sp1; 
  } 

  // weak 는 자신이 가리키는 곳이 유효한지 조사할 수 있다. 
  //if (sp1->bf != nullptr) 
  if ( sp1->bf.expired() ) 
  { 
    cout << "객체 파괴" << endl; 
  } 
}
```

### 1.5. shared_ptr의 관리객체
1. strong cnt 
2. weak cnt (strong cnt + weak_ptr) 
3. 삭제자 
4. 주소지 

```cpp
#include <iostream> 
#include <memory> 
using namespace std; 
  
class Car 
{ 
public: 
    void Go() { std::cout << "Go" << std::endl; } 
    ~Car() {std::cout << "~Car" << std::endl;    } 
}; 
int main() 
{ 
  shared_ptr<Car> sp; 
  weak_ptr<Car>   wp; 
  { 
    shared_ptr<Car> p(new Car); 
  
    // sp = p;    // 참조 계수 증가 
    wp = p;        // 참조 계수 증가 안함 

    cout << p.use_count() << endl; // 2, 1 
  } 

  if (wp.expired()) 
  { 
    cout << "객체 파괴" << endl; 
  } 
  else 
  { 
    //wp를 사용해서 객체 접근 
    //wp->Go(); // 불가능, -> 연산자 없음 

    // wp를 가지고 sp를 만들어서 접근, shared_ptr은 살아있는 동안 100% 안전하게 사용가능 
    shared_ptr<Car> sp = wp.lock(); 
      
    if (sp) 
      sp->Go(); 
  } 

  cout << "------" << endl; 
}
```

### 1.6. RAII
- Resource Acquision Is Initialization
- 자원을 획득하는 것은 꼭 자원 관리 객체가 초기화 될 때 하자

```cpp
#include <iostream> 
#include <memory> 
using namespace std; 
  
int main() 
{ 
  int* p = new int; 

  //shared_ptr<int> sp1(p); 
  //shared_ptr<int> sp2(p); // sp1과는 다른 별개의 관리객체 사용 
                            // 절대 이렇게 사용하면 안됩니다. 

  // 자원을 획득하는 것은 꼭! 자원 관리 객체가 초기화 될 때 해야 한다. 
  // Resource Acquision Is Initialization 
  // RAII idioms 
  shared_ptr<int> sp1(new int); 
  shared_ptr<int> sp2(new int); 
}
```

## 참고
codenuri 강석민 강사 강의 내용기반으로 정리한 내용입니다.  
[코드누리](https://github.com/codenuri)  

<br>

    This is personal diary for study documents.
    Please comment if I'm wrong or missing something else 😄. 

[Top](#){: .btn .btn--primary }{: .align-right}