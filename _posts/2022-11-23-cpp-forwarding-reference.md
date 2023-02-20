---
published: true
title:  "[C++] 포워딩 레퍼런스(Forwarding reference)"
excerpt: "C++에 대해 알아보기, Forwarding reference"

categories:
  - Cpp
tags:
  - [C++, Cpp, ForwardingReference]

toc: true
toc_sticky: true
author: chachayelmo
sitemap:
  changefreq : daily
  priority : 1.0
comment: true
date: 2022-11-23
last_modified_at: 2022-11-23
---

## 1. Forwarding reference
- 함수인자로
  - int&  : int 타입의 lvalue만 전달 가능
  - int&& : int 타입의 rvalue만 전달 가능
  - T&    : 임의의 타입의 lvalue만 전달 가능

- Forwarding reference, universial reference
  - lvalue를 보내면 T : int&  최종함수 f(int&)
  - rvalue를 보내면 T : int   최종함수 f(int&&)
  - T&& : 임의의 타입의 lvalue, rvalue를 모두 전달 가능!

## 2. 코드로 알아보기
- [web_compiler](https://godbolt.org/) 에서 확인

### 2.1. T& 일 경우

```cpp
#include <iostream> 
  
void f1(int&  a) {} 
void f2(int&& a) {} 
  
template<typename T> 
void f3(T& t) {} 
  
int main() 
{ 
  int n = 0; 
    
  // 1. 사용자가 타입을 명시적으로 전달하는 경우 
  f3<int>(n);     // T : int,     T& : int&,        f3(int&) 
  f3<int&>(n);    // T : int&,    T& : int& &,      f3(int&)  
  f3<int&&>(n);   // T : int&&,   T& : int&& &,     f3(int&)  

  // 2. 사용자가 타입을 지정하지 않는 경우, 
  // 컴파일러가 최대한 노력해서 함수를 만들려고 함
  // f3(10); // error. 함수를 만들 수 없음 
  f3(n);     // ok, T: int로 결정 

  return 0; 
}
```

### 2.2. T&& 일 경우


```cpp
template<typename T> 
void f4(T&& t) {} 
  
int main() 
{ 
  int n = 0; 

  // 1. 사용자가 타입을 명시적으로 전달하는 경우 
  f4<int>(10);      // T : int,        T&& : int&&,        f4(int&&) 
  f4<int&>(n);      // T : int&,       T&& : int& &&,      f4(int&)  
  f4<int&&>(10);    // T : int&&,      T&& : int&& &&,     f4(int&&)  

  // 2. 사용자가 타입을 지정하지 않는 경우 ,
  // 컴파일러가 인자를 보고 결정
  f4(n);  // ok, T: int&로 결정,   T&& : int& &&      -> f4(int&) 
  f4(10); // ok, T: int로 결정,    T&& : int&&        -> f4(int&&)      
}
```

### 2.3. 함수가 lvalue, rvaule를 받게 하는 방법

```cpp
#include <iostream> 
using namespace std; 
  
// 1. call by value 
//    특징 : 복사본이 생성된다. primitivate 타입이면 전혀 문제 될 것 없다. 
//           하지만 int가 아니라 user type 이면 문제 
void foo(int arg) {} 
  
// 2. const lvalue reference 
//    특징 : 복사본이 없다. 하지만 원본에 상수성을 추가해서 받게 된다. 
void foo(const int& arg) {} 
  
// 3. lvalue 참조와 rvalue 참조 버전으로 2개 제공 
void foo(int&) {} 
void foo(int&&) {} 
  
// 4. forwarding reference를 사용하면 3번의 함수를 자동 생성 할 수 있다. 
//    3번의 함수를 자동생성할 수 있다.
//    n을 전달 T : int&    T&& : int& 
//    10을 전달 T : int    T&& : int&& 
template<typename T> void foo(T&& arg) {} 
  
int main() 
{ 
  int n = 0; 

  foo(n); 
  foo(10); 
}
```

### 2.4. forwarding reference

```cpp
template<typename T> class Test
{
public:
  // 아래 함수의 인자는 forwarding reference 일까요 ?
  // 아래 함수는 함수 템플릿이 아닌 일반 함수
  // 클래스 템플릿의 일반함수
  void foo(T&& arg) {}

  // 아래처럼 해야지 forwarding reference
  template<typename U> void goo(U&& arg) {}
};
int main()
{
  Test<int> t;
  int n = 0;
  t.foo(n); // error
  t.foo(0); // ok
  t.goo(n); // ok
}
```

### 2.5. auto

```cpp
#include <vector>
using namespace std;

int main()
{
  int n = 0;
  auto a1 = n;      // ok, int a1 = n
  auto a2 = 0;      // ok, int a2 = 0
  auto& a3 = n;     // ok, int& a3 = n 
  auto& a4 = 0;     // error, int& a4 = 0

  // 아래 코드는 rvalue reference 일까요? forwarding reference 일까요? - forwarding reference
  auto&& a5 = n;    // n (lvalue -> auto(T) : int&
                    // auto&& : int& && -> int& a5 = n

  auto&& a6 = 0;    // 0 (rvalue -> auto(T) : int
                    // auto&& : int&& -> int&& a6 =0

  //vector<int> v(10);
  vector<bool> v(10); // bool은 꺼낼 때 rvalue가 나옴
  for (auto&& n : v)
  {
  }
}
```

## 참고
codenuri 강석민 강사 강의 내용기반으로 정리한 내용입니다.  
[코드누리](https://github.com/codenuri)

<br>

    This is personal diary for study documents.
    Please comment if I'm wrong or missing something else 😄. 

[Top](#){: .btn .btn--primary }{: .align-right}