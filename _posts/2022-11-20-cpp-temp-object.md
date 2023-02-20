---
published: true
title:  "[C++] 임시객체(Temporary object)"
excerpt: "C++에 대해 알아보기, temporary object"

categories:
  - Cpp
tags:
  - [임시객체, C++, Cpp, TemporaryObject, Object]

toc: true
toc_sticky: true
author: chachayelmo
sitemap:
  changefreq : daily
  priority : 1.0
comment: true
date: 2022-11-20
last_modified_at: 2022-11-20
---

## 1. 임시객체
- 런타임 중에 잠깐 사용되는 객체로 문장의 끝(;)에서 파괴
- 이름이 없는 객체, 클래스이름() = 임시객체

## 2. 코드로 알아보기
- [web_compiler](https://godbolt.org/) 에서 확인

### 2.1. 임시객체의 수명

```cpp
#include <iostream> 
using namespace std; 
  
class Point 
{ 
  int x, y; 
public: 
  Point(int a, int b) : x(a), y(b) { cout << "Point()" << endl; } 
  ~Point()                         { cout << "~Point()" << endl; } 
}; 
int main() 
{ 
  Point p(1, 2); // 이름 있는 객체. 블럭({})의 끝에서 파괴

  Point(1, 2), cout << "X" << endl; // 이름이 없는 객체, 문장의 끝(;)에서 파괴

  cout << "---------" << endl; 
}
```

### 2.2. 임시객체를 받을 수 있는 것들
 - const &, 상수 참조
 - &&, rvalue 참조

```cpp
#include <iostream> 
using namespace std; 
  
class Point 
{ 
public: 
    int x, y; 
    Point(int a, int b) : x(a), y(b) { cout << "Point()" << endl; } 
    ~Point()                         { cout << "~Point()" << endl; } 
}; 
int main() 
{ 
  Point p(1, 2);  
    
  Point* p1 = &p;           // ok 
  Point* p2 = &Point(1, 2); // error, 임시객체는 주소를 구할 수 없음 

  p.x = 10;           // ok 
  Point(1, 2).x = 10; // error, 임시객체는 lvalue로 올 수 없음, 즉 rvalue

  Point& r1 = p;            // ok 
  Point& r2 = Point(1, 2);  // error, 임시객체를 참조로 가리킬 수 없음, 왜? 주소를 구할 수 없으므로

  // 그러나, 임시객체는 상수 참조를 가리킬 수 있음 
  const Point& r2 = Point(1, 2);  
  // 임시객체의 수명이 r2의 수명으로 연장됨 

  r2.x = 10; // error, 상수 참조 이므로 값을 넣을 수 없음 

  // C++11의 새로운 참조 (&&) 는 상수성이 없음, 임시객체를 가리킴
  Point&& r3 = Point(1, 2); 
  r3.x = 10; // ok 
}
```

### 2.3. 임시객체와 함수 인자

```cpp
#include <iostream> 
using namespace std; 
  
class Point 
{ 
public: 
  int x, y; 
  Point(int a, int b) : x(a), y(b) { cout << "Point()" << endl; } 
  ~Point() { cout << "~Point()" << endl; } 
}; 
  
// 임시객체와 함수 인자 
void foo(const Point& p)
// 함수인자로 임시객체를 사용하는 경우 일반참조는 받을 수 없음
// 상수참조로 사용해야 lvalue, rvalue 둘다 받을 수 있음
{ 
}

int main() 
{
  // 임시객체를 사용하기 않고 넘기는 방법
  // Point pt(1, 2);
  // foo(pt);

  // 임시객체를 사용한 인자 전달 
  foo(Point(1, 2)); 

  cout << "---------" << endl; 
}
```

### 2.4. 임시객체와 함수 리턴
- 지역변수는 절대로 참조반환하면 안됨
- RVO = Return Value Optimization
  - 객체를 만들고 리턴하지 말고 바로 만들면서 리턴하기
- NRVO = Named RVO
  - 객체를 만들고 리턴하여도 컴파일러가 알아서 최적화 가능

```cpp
#include <iostream> 
using namespace std; 
  
class Point 
{ 
public: 
    int x, y; 
    Point(int a, int b) : x(a), y(b) { cout << "Point()" << endl; } 
    ~Point() { cout << "~Point()" << endl; } 
  
    Point(const Point& p) { cout << "copy ctor" << endl; } 
}; 

// 지역변수는 절대로 참조반환하면 안됨
/* 
Point& foo() // 버그 
{ 
  Point p(1, 2); 
  return p;   
} 
*/ 
  
Point foo() 
{ 
  // 요즘에는 아래처럼 반환해도 컴파일러가 최적화 가능
  // NRVO ( Named RVO ) 
  //Point p(1, 2); 
  //return p;

  // 만들고 리턴하지 말고 만들면서 리턴하자 
  // RVO ( Return Value Optimization ) 
  return Point(1, 2);  
} 
  
int main() 
{ 
  Point p1(0, 0); 

  p1 = foo(); 

  cout << "---------" << endl; 
}
```

### 2.5. 임시객체
1. 사용자가 만들기도 합니다. - 함수 인자로 전달할 때 
2. 컴파일러에 의해 만들어 지기도 합니다. 
  - 함수가 값으로 반환할 때 
  - 값 타입으로 캐스팅할 때 
  - 람다표현식을 사용할 때

```cpp
#include <iostream> 
using namespace std; 
  
class Point 
{ 
public: 
  int x, y; 
  Point(int a, int b) : x(a), y(b) { cout << "Point()" << endl; } 
  ~Point() { cout << "~Point()" << endl; } 

  Point(const Point& p) { cout << "copy ctor" << endl; } 
}; 
  
// call by value : 복사본(pt)을 생성 
void goo(Point pt) {} 
  
Point p(0, 0);
  
// return by value : 복사본(임시객체)을 생성해서 반환, 리턴용 임시객체는 함수호출 구문의 끝에서 파괴
Point foo() 
{ 
  return p; 
} 
  
// return by reference 참조 반환 : 임시객체를 만들지 말라는 의미
Point& goo() 
{ 
  return p; 
} 
  
int main() 
{ 
  // foo().x = 10; // error, 임시객체는 lvalue가 될 수 없음 
  goo(p);
  goo().x = 10; // ok 
  cout << p.x << endl; 
}
```

## 참고
codenuri 강석민 강사 강의 내용기반으로 정리한 내용입니다.  
[코드누리](https://github.com/codenuri)  

<br>

    This is personal diary for study documents.
    Please comment if I'm wrong or missing something else 😄. 

[Top](#){: .btn .btn--primary }{: .align-right}