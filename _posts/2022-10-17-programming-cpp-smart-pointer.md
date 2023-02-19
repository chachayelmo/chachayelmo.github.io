---
published: true
title:  "[C++] 스마트포인터(Smart Pointer)"
excerpt: "C++에 대해 알아보기, Smart Pointer"

categories:
  - Cpp
tags:
  - [스마트포인터, C++, Cpp, SmartPointer]

toc: true
toc_sticky: true
sitemap:
  changefreq : daily
  priority : 1.0
date: 2022-10-17
last_modified_at: 2022-10-17
---

## 1. Smart Pointer 란?
- 전제조건
    - new 사용하여 동적할당(Heap영역) 받은 메모리는 반드시 delete 키워드를 사용하여 해제
    - memory leak로부터 프로그램의 안전성을 보장하기 위해 스마트 포인터를 표준으로 제공
- Smart pointer란?
    - 포인터처럼 동작하는 클래스 템플릿으로, 사용이 끝난 메모리를 자동으로 해제

## 2. 스마트 포인터의 동작
- new 사용해 raw pointer가 실제 메모리를 가리키도록 초기화 한 후에 기본 포인터를 스마트 포인터에 대입하여 사용
- 이렇게 정의된 스마트 포인터의 수명이 다하면 소멸자는 delete 키워드를 사용하여 할당된 메모리를 자동으로 해제
- 따라서 new 키워드가 반환하는 주소값을 스마트 포인터에 대입하면 따로 메모리를 해제할 필요가 없음

## 3. 스마트 포인터의 종류
- C++11 이전에는 auto_ptr이라는 스마트 포인터를 사용하여 이 작업을 수행
- 하지만 C++11부터는 다음과 같은 새로운 스마트 포인터를 제공
    1. unique_ptr
    2. shared_ptr
    3. weak_ptr

### 3.1. unique_ptr
- unique_ptr이란?
    - 하나의 스마트 포인터만이 특정 객체를 소유할 수 있도록 객체에 소유권 개념을 도입한 스마트 포인터
- 스마트 포인터는 해당 객체의 소유권을 가지고 있을 때만 소멸자가 해당 객체를 삭제
- 소유권이 이전되면 이전 객체는 더이상 unique_ptr 객체를 소유하지 않게 재설정
- unique_ptr 인스턴스는 move() 멤버 함수를 통해 소유권을 이전할 수는 있지만 복사할 수는 없음

```cpp
#include <iostream> 
#include <memory>
using namespace std; 

class Point 
{
public: 
    Point()  { cout << "Point() " << endl; } 
    ~Point() { cout << "~Point()" << endl; } 
};

int main() 
{
    // Point 형 unique_ptr인 ptr01을 선언
    unique_ptr<Point> ptr01(new Point());

    // ptr01에서 ptr02로 소유권 이전
    auto ptr02 = move(ptr01);          

    // 대입 연산자를 이용한 복사는 error
    // unique_ptr<int> ptr03 = ptr01;

    // ptr02가 가리키고 있는 메모리 영역을 삭제
    ptr02.reset();
    // ptr01가 가리키고 있는 메모리 영역을 삭제
    ptr01.reset();
} 
```

#### 3.1.1.  make_unique()
- C++14 이후부터 제공되는 make_unique() 함수를 사용하면 unique_ptr 인스턴스를 안전하게 생성
- make_unique() 함수는 전달 받은 인수를 사용해 지정된 타입의 객체를 생성하고, 생성된 객체를 가리키는 unique_ptr을 반환
- 이 함수를 사용하면, 예외 발생에 대해 안전하게 대처
- 아래는 Person 객체를 가리키는 hong이라는 unique_ptr를 make_unique() 를 통해 생성하는 예제

```cpp
#include <iostream>
#include <memory>
using namespace std;

class Person
{
private:
    string name_;
    int age_;
public:
    Person(const string& name, int age);
    ~Person() { cout << "소멸자가 호출되었습니다." << endl; }
    void ShowPersonInfo();
};

int main(void)
{
    unique_ptr<Person> hong = make_unique<Person>("길동", 29);
    hong->ShowPersonInfo();
    return 0;
}

Person::Person(const string& name, int age) // 생성자 바디 정의
{
    name_ = name;
    age_ = age;
    cout << "생성자가 호출되었습니다." << endl;
}

void Person::ShowPersonInfo() {
    cout << name_ << "의 나이는 " << age_ << "살입니다." << endl;
}
```

### 3.2. shared_ptr
- shared_ptr란?
    - 하나의 특정 객체를 참조하는 포인터가 총 몇 개인지를 참조하는 스마트 포인터
- 참조하고 있는 스마트 포인터의 개수를 참조 횟수(reference count)
- reference count는 특정 객체에 새로운 shared_ptr이 추가될 때마다 1씩 증가하며, 수명이 다할 때마다 1씩 감소
- 따라서 마지막 shared_ptr의 수명이 다하여, 참조 횟수가 0이 되면 delete 키워드를 사용하여 메모리를 자동으로 해제

```cpp
#include <iostream> 
#include <memory>
using namespace std; 

class Point 
{
public: 
    Point()  { cout << "Point() " << endl; } 
    ~Point() { cout << "~Point()" << endl; } 
};

int main() 
{

    // Point 형 shared_ptr인 ptr01을 선언
    shared_ptr<Point> ptr01(new Point());
    cout << ptr01.use_count() << endl; // 1

    // 복사 생성자를 통한 ptr02 선언
    auto ptr02(ptr01);                 
    cout << ptr01.use_count() << endl; // 2

    // 대입연산자를 통한 ptr03 선언
    auto ptr03 = ptr01;
    cout << ptr01.use_count() << endl; // 3

    // 삭제
    ptr03.reset();
    cout << ptr01.use_count() << endl; // 2
    ptr02.reset();
    ptr01.reset();
    // 소멸자 ~Point() 불림
    cout << ptr01.use_count() << endl; // 0
} 
```

#### 3.2.1. make_shared()
- use_count() 는 shared_ptr 객체가 현재 가리키고 있는 리소스를 참조 중인 소유자의 수를 반환
- make_shared() 함수를 사용하면 shared_ptr 인스턴스를 안전하게 생성
- make_shared() 함수는 전달받은 인수를 사용해 지정된 타입의 객체를 생성하고, 생성된 객체를 가리키는 shared_ptr을 반환, 예외 발생에 대해 안전하게 대처가능
- 다음 예제는 Person 객체를 가리키는 hong이라는 shared_ptr를 make_shared() 함수를 통해 생성하는 예제

```cpp
#include <iostream> 
#include <memory>
using namespace std; 

class Person 
{
    string _name;
    int _age;
public: 
    Person(const string& name, int age) : _name(name), _age(age)  { cout << "Person() " << endl; } 
    ~Person() { cout << "~Person()" << endl; } 
};

int main() 
{
    shared_ptr<Person> hong = make_shared<Person>("길동", 29);
    cout << "현재 소유자 수 : " << hong.use_count() << endl; // 1
    auto han = hong;
    cout << "현재 소유자 수 : " << hong.use_count() << endl; // 2
    han.reset(); // shared_ptr인 han을 해제
    cout << "현재 소유자 수 : " << hong.use_count() << endl; // 1
    hong.reset();
    cout << "check" << '\n';
} 
```

### 3.3. weak_ptr
- weak_ptr란?
    - 하나 이상의 shared_ptr 인스턴스가 소유하는 객체에 대한 접근을 제공하지만, 소유자의 수에는 포함되지 않는 스마트 포인터
- 이렇게 서로가 상대방을 참조하고 있는 상황을 순환 참조(circular reference)라고 함
- 순환참조의 문제점
    - shared_ptr은 reference count를 기반으로 동작하는 스마트 포인터인데 서로가 상대방을 가리키는 shared_ptr를 가지고 있다면
    - 참조 횟수는 절대 0이 되지 않으므로 메모리는 영원히 해제되지 않음
- weak_ptr은 바로 이러한 shared_ptr 인스턴스 사이의 순환 참조를 제거하기 위해서 사용

```cpp
// 순환참조 예시
#include <iostream>
#include <memory>

class A {
  int *data;
  std::shared_ptr<A> other;

 public:
  A() {
    data = new int[100];
    std::cout << "자원을 획득함!" << std::endl;
  }

  ~A() {
    std::cout << "소멸자 호출!" << std::endl;
    delete[] data;
  }

  void set_other(std::shared_ptr<A> o) { other = o; }
};

int main() {
  std::shared_ptr<A> pa = std::make_shared<A>();
  std::shared_ptr<A> pb = std::make_shared<A>();

  pa->set_other(pb);
  pb->set_other(pa);
  // error, 프로그램이 끝나도 소멸자가 불리지 않음
}
```

```cpp
#include <iostream>
#include <memory>

class A {
  int *data;
  std::weak_ptr<A> other; // weak_ptr로 받음

 public:
  A() {
    data = new int[100];
    std::cout << "자원을 획득함!" << std::endl;
  }

  ~A() {
    std::cout << "소멸자 호출!" << std::endl;
    delete[] data;
  }

  void set_other(std::shared_ptr<A> o) { other = o; }
};

int main() {
  std::shared_ptr<A> pa = std::make_shared<A>();
  std::shared_ptr<A> pb = std::make_shared<A>();

  pa->set_other(pb);
  pb->set_other(pa);
  // 소멸자가 정상적으로 호출
}
```

## 참고
codenuri 강석민 강사 강의 내용기반으로 정리한 내용입니다.  
[코드누리](https://github.com/codenuri)  
[모두코드](https://modoocode.com/252)

<br>

    This is personal diary for study documents.
    Please comment if I'm wrong or missing something else 😄. 

[Top](#){: .btn .btn--primary }{: .align-right}