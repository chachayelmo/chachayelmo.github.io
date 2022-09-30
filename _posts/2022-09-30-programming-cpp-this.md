---
published: true
title:  "[Programming] C++ this call"
excerpt: "C++에 대해 알아보기, this"

categories:
  - cpp
tags:
  - [C++, cpp, this]

toc: true
toc_sticky: true
 
date: 2022-09-30
last_modified_at: 2022-09-30
---

## 1. this
- 요약
  1. 멤버함수는 인자로 this를 가짐
  2. static 멤버함수는 인자로 this를 가지지 않음
  3. static 멤버함수에서 member 변수나 함수를 사용

## 2. 코드로 보는 this
### 2.1. 멤버 함수의 호출원리
- 멤버 함수는 인자로 this가 추가
- static 멤버 함수는 인자로 this가 추가되지 않음

```cpp
#include <iostream> 
using namespace std; 

class Point 
{ 
  int x; 
  int y; 
public: 
  Point(int a = 0, int b = 0) : x(a), y(b) {} 
  void setter(int a, int b) // 멤버함수는 this를 포함
  // setter(Point* this , int a, int b) 
  { 
    x = a; // this->x = a
    y = b; // this->y = b
  }

  static void foo(int a) // static 멤버 함수는 this를 포함하지 않음
  // foo(int a)
  {
    x = a;  // this->x = a로 변경해야 하는데 this가 없음, <span style="color:red">error</span>
            // static 멤버함수에서는 멤버 데이타에 접근이 안됨
  } 
};

int main() 
{ 
    Point::foo(10); // static 멤버함수는 객체없이 호출가능 
    Point p1, p2; 
    p1.setter(1, 2); // set(&p1, 10, 20) 의 원리
                     /*
                      push 20 
                      push 10  진짜 인자는 스택으로 
                      mov ecx, &p1 // 객체는 레지스터에 
                      call Point::set
                     */
    getchar(); 
}
```

### 2.2. 일반 함수 포인터와 멤버 함수 포인터
- 일반함수 포인터에 멤버 함수의 주소를 담을 수 없음 (객체가 없기 때문에)
- 일반함수 포인터에 static 멤버 함수의 주소는 담을 수 있음
- 멤버 함수 포인터를 만들고 사용하는 방법

```cpp
class Point 
{ 
public: 
    void Foo(int a) {    cout << "Point::Foo" << endl; } 
    static void Foo2(int a) { cout << "Point::Foo2" << endl; } 
}; 
  
void foo(int a) { cout << "foo" << endl; } 
  
int main() 
{ 
    void(*f1)(int) = &foo;  // ok 
//    void(*f2)(int) = &Point::Foo; // error 
    void(*f2)(int) = &Point::Foo2; // ok 
  
    // 멤버 함수 포인터를 만드는 방법 
    void(Point::*f3)(int) = &Point::Foo; //  
  
//  f3(0); // error. 객체가 없다. 
  
    Point p; 
//  p.f3(0); // error. f3이라는 멤버함수를 찾게된다. 
  
//  p.*f3(0); // .* : 연산자 우선순위 문제, error
  
    (p.*f3)(0);  // ok. 멤버 함수 포인터를 사용해서 함수를 호출하는 방법

    Point* pp = &p; 
    (pp->*f3)(0);
}
```

### 2.3. 1st static 멤버함수에서 this 사용 방법 
- 기존 함수를 static 멤버함수로 사용해야 함
- void pointer를 가진 경우 static 멤버 함수인 스레드 함수에서 this를 사용할 수 있게 하기 위해 인자로 전달

```cpp
#include <iostream> 
#include "Windows.h"
#include <conio.h> 
/* 기존 함수
DWORD __stdcall threadProc(void* p) 
{ 
    return 0; 
} 
int main() 
{ 
    CreateThread(0, 0, threadProc, "A", 0, 0); 
} 
*/ 

// 라이브러리 Base 클래스
class Thread 
{ 
public: 
  void run() {
    CreateThread(0, 0, threadProc, this, 0, 0); // this를 포함하여 전달
  } 

  static DWORD __stdcall threadProc(void* p) 
  { 
    Thread* self = static_cast<Thread*>(p); // self가 this, 모든 멤버함수 이용 가능
    self->Main(); // Main(self) 
//      Main(); error
    return 0; 
  } 

  virtual void Main() = 0; // Main(Thread* this) 
};

// 라이브러리 사용자 클래스 
class MyThread : public Thread 
{ 
  int data; 
public: 
  MyThread(int d) : data(d) {} 
  virtual void Main()  
  {  
      cout << "MyThread Main" << endl;  
      data = 10; 
  } 
}; 
  
int main() 
{ 
    MyThread t(10); 
    t.run(); // 새로운 스레드 생성후, Main 가상함수실행 
  
    _getch(); 
}
```

### 2.4. static 멤버함수에서 this 사용 방법 2nd
- 자료구조에 this를 보관해서 사용

```cpp
#define USING_GUI 
#include "cppmaster.h" // ec_set_timer 
#include <iostream> 
#include <string> 
using namespace std; 
  
/* 기존 함수
void foo(int id) 
{ 
    cout << "foo : " << id << endl; 
} 
int main() 
{ 
    int n1 = ec_set_timer(500,  foo); 
    int n2 = ec_set_timer(1000, foo); 
  
    ec_process_message(); 
} 
*/ 

#include <map> 
class Clock;  // 전방선언 
map<int, Clock*> this_map; // 중요!, this를 보관하기 위한 자료구조
  
class Clock 
{ 
  string name; 
public: 
  Clock(string s) : name(s) {} 

  void Start(int ms)  
  { 
    int id = ec_set_timer(ms, timerRoutine); 
    this_map[id] = this; // this 포인터를 자료구조에 보관
  } 

  static void timerRoutine(int id) 
  { 
    Clock* self = this_map[id]; 
    cout << self->name << endl; 
  } 
}; 
  
int main() 
{ 
  Clock c1("A"); 
  Clock c2("\tB"); 

  c1.Start(500);
  c2.Start(1000); 
  ec_process_message(); 
}
```

## 참고
codenuri 강석민 강사 강의 내용기반으로 정리한 내용입니다.  
[코드누리](https://github.com/codenuri)

<br>

    This is personal diary for study documents.
    Please comment if I'm wrong or missing something else 😄. 

[Top](#){: .btn .btn--primary }{: .align-right}