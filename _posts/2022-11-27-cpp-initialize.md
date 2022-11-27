---
published: true
title:  "[Programming] C++ 초기화"
excerpt: "C++에 대해 알아보기, Initialization"

categories:
  - Cpp
tags:
  - [C++, Cpp, Initialization]

toc: true
toc_sticky: true
 
date: 2022-11-27
last_modified_at: 2022-11-27
---

## 1. 초기화
- C++98 초기화의 문제점
  1. 객체의 종류에 따라 초기화 방법이 다름
  2. STL의 컨테이너를 효율적으로 초기화 하기 어려움
  3. 클래스 멤버로 있는 배열을 초기화 할 수 없음

- 해결 방법
  1. Member 초기화
  2. braced-init-list
  3. Uniform 초기화
  4. Default 초기화

- C++11 Initializer_list 함수
  - 초기화 시에 사용하기 위해 C++11에서 추가한 도구

## 2. 코드로 알아보기
- [web_compiler](https://godbolt.org/) 에서 확인

### 2.1. C++98 초기화의 문제점
```cpp
struct Point  
{ 
  int x, y;  
}; 
  
class Complex 
{ 
  int re, im; 
public: 
  Complex(int a, int b) : re(a), im(b) {} 
}; 
  
int main() 
{ 
     
  // 1. 객체의 종류에 따라 초기화 방법이 다르다. 
  int n1 = 0; 
  int n2(0); 
  int x[3] = { 1,2,3 }; 
  Point   p = { 1,1 }; 
  Complex c(1, 1); 

  // 2. STL의 컨테이너를 효율적으로 초기화 하기 어렵다. 
  vector<int> v; // 1~10으로 초기화 된 vector 만들기     
}

// 3. 클래스 멤버로 있는 배열을 초기화 할 수 없다. 
class A 
{ 
public: 
  int x[10]; // 초기화 할수 있을까 ? 
};
```

### 2.2. C++11 멤버 초기화

```cpp
#include <iostream> 
using namespace std; 
  
int x = 10; 
  
class Test 
{ 
public: 
  int a = 0; // member initializer 
  int b = ++x; // 초기화 리스트에 없을 때만 사용가능하니 쓰지말기 

  Test() {}    // ++x  
  Test(int v) : b(v) {} 
}; 

int main() 
{ 
  Test t1; 
  Test t2(3); 

  cout << x << endl; // 11 
} 
```

### 2.3. Braced init list

```cpp
#include <iostream> 
using namespace std; 
  
// braced-init-list 
// direct vs copy
struct Point { int x, y; }; 
  
class Complex 
{ 
  int re, im; 
public: 
  Complex(int a, int b) : re(a), im(b) {} 
}; 
  
int main() 
{ 
  // uniform initialization    : 일관된 초기화 
  // 중괄호 초기화                : brace init 

  // direct initialization 
  int n1{ 0 }; 
  int n2{ 0 }; 
  int x[3]{ 1,2,3 }; 
  Point   p{ 1, 1 }; 
  Complex c{ 1, 1 }; 

  // copy initialization 
  int n11       = { 0 }; 
  int n22       = { 0 }; 
  int xx[3]     = { 1,2,3 }; 
  Point   pp    = { 1, 1 }; 
  Complex cc    = { 1, 1 }; 

  int n3  = 3.4;        // ok 
  int n4  = { 3.4 };    // error 
  char cc = { 300 };    // error 
}
```

### 2.4. 초기화 활용 및 explicit 생성자

```cpp
class Point 
{ 
  int x, y; 
public: 
  Point(int a = 0, int b = 0) : x(a), y(b) {} 
}; 
  
void foo(Point pt) {} 
void goo(int n) {} 
  
int main() 
{ 
  int n = { 5 }; 

  goo({ 5 }); 

  Point pt(1, 2); 
  Point pt1 = {1, 2}; 
  foo(pt); 
  foo({ 1,2 }); 
}
```

```cpp
class Vector 
{ 
public: 
  // explicit : 직접 초기화는 가능하지만 
  //            복사 초기화는 사용할 수 없다. 
  explicit Vector(int sz) {} 
}; 
  
void foo(Vector v) {} 
  
int main() 
{ 
  // direct initialization 
  Vector v1(10); 
  Vector v2{ 10 }; 

  // copy initialization 
  Vector v3 = 10;      //error 
  Vector v4 = (10);    //error 
  Vector v5 = { 10 };  //error 

  foo(v1); 
  foo(10);            //error 
}
```

```cpp
#include <iostream> 
#include <string> 
#include <vector> 
#include <memory> 
using namespace std; 
  
void foo(string s) {} 
  
int main() 
{ 
  // string class는 explicit 생성자가 아님 
  string s1("hello"); 
  string s2 = "hello";    // ok 
  foo("hello"); 

  // vector class는 explicit 생성자 
  vector<int> v1(10); 
  vector<int> v2 = 10;    // error 

  // explicit 생성자 
  unique_ptr<int> p1(new int); 
  unique_ptr<int> p2 = new int; // error 
}
```

### 2.5. Default 초기화

```cpp
#include <iostream> 
#include <vector> 
#include <initializer_list> 
using namespace std; 
  
// value initialization vs default 
struct Point 
{ 
  int x, y; 
  Point() = default; // 컴파일러에게 너가 초기화 만들어달라는 의미 
}; 
  
int main() 
{ 
  Point p1; 
  Point p2{}; // 모든 멤버가 0으로 초기화 

  cout << p2.x << endl; 

  //int n1;        // 쓰레기값    ,               default 초기화 
  //int n2{};      // 디폴트 값(0)으로 초기화 ,    value 초기화 
}
```

```cpp
#include <iostream> 
#include <vector> 
#include <initializer_list> 
using namespace std; 
  
// aggregate 
// aggregate type : 생성자 없어도 {} 로 초기화 가능한 타입. 배열, 구초제 등 
// aggregate type이 안되는 조건 : 생성자가 있거나 가상함수가 있을 경우 
struct Point 
{ 
  int x, y; 
  Point() = default;    // aggregate 

  void foo() {}        // aggregate 
  //virtual void goo() {} // aggregate 아님 
  //Point() {}        // 이 요소 때문에 aggregate 가 아님 
  //Point(int a, int b) {} 
}; 
  
int main() 
{ 
  Point p1; 
  //Point p2(0, 0); 
  Point p2{ 0,0 }; 
  Point p3 = { 0,0 }; 
}
```

### 2.6. Initializer_list

```cpp
#include <iostream> 
#include <initializer_list> 
using namespace std; 
  
void foo(initializer_list<int> e) 
{ 
} 
  
int main() 
{ 
  initializer_list<int> e = { 1,2,3 };    // initializer_list는 stack에 연속된 주소로 저장  
                                          // vector는 연속된 주소로 heap에 저장, 
                                          // linked list는 연속된 주소가 아님 

  foo(e); 
  foo({ 1,2,3,4,5,6 }); 
}
```

```cpp
#include <iostream> 
#include <vector> 
#include <initializer_list> 
using namespace std; 
  
  
class Point 
{ 
  int x, y; 
public: 
  Point(int a, int b) : x(a), y(b) { cout << "int, int" << endl; } 
  Point(initializer_list<int>) { cout << "initializer_list" << endl; } 
};

int main() 
{ 
  // 아래 코드가 각각 어떤 생성자를 호출할지 생각해보기
  Point p0(1, 1);          // 1 
  Point p1({ 1, 1 });      // 2 
  Point p2{ 1, 1 };        // 핵심. 2번이 우선시됨, 없으면 1번 

  Point p4 = { 1, 1 };     // 2번이 우선 

  // Point p5(1, 1, 1);    // error 
  Point p6{ 1,1,1 };       // 2 
  Point p7 = { 1,1,1 };    // 2 

  // C++11부터 아래처럼 표기가 가능
  // initializer_list 때문에 가능한 표기법 
  vector<int> v = { 1,2,3,4,5,6,7,8,9 }; 

  // 아래 2줄의 차이점은? 
  vector<int> v1(10, 2);    // 10개를 2로 초기화 
  vector<int> v2{ 10, 2 };  // 2개를 10, 2로 초기화 

  //vector<int> v3 = 10;    // error 
  vector<int> v4 = { 10 };  // ok 
}
```

## 참고
codenuri 강석민 강사 강의 내용기반으로 정리한 내용입니다.  
[코드누리](https://github.com/codenuri)

<br>

    This is personal diary for study documents.
    Please comment if I'm wrong or missing something else 😄. 

[Top](#){: .btn .btn--primary }{: .align-right}