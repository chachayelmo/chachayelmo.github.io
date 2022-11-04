---
published: true
title:  "[Programming] C++ Type deduction"
excerpt: "C++에 대해 알아보기, 타입 추론 - template, auto"

categories:
  - Cpp
tags:
  - [C++, Cpp, TypeDeduction, Auto]

toc: true
toc_sticky: true
 
date: 2022-11-04
last_modified_at: 2022-11-04
---

## 1. Type deduction
### 1.1. 인자 타입이 value면, 함수 인자의 const, volatile, reference는 제거

```cpp
#include <iostream> 
using namespace std; 
  
template<typename T> void foo(T a) 
{ 
} 
  
int main() 
{ 
  int n = 10; 
  int&r = n; 
  const int c = n; 
  const int& cr = c; 
  foo(n); // T : int 
  foo(r); // T : int 
  foo(c); // T : int 
  foo(cr);// T : int 

  const char* s = "hello"; 
  foo(s); // T: const char* 

  const char* const s2 = "aaa"; 
  foo(s2); // T: const char* 
}
```

### 1.2. 인자 타입이 reference 면, 함수 인자가 가지는 참조만 제거

```cpp
#include <iostream> 
using namespace std; 

template<typename T> void foo(T& a) 
{ 
} 
int main() 
{ 
  int n = 10; 
  int&r = n; 
  const int c = n; 
  const int& cr = c; 
  foo(n); // T : int    a : int& 
  foo(r); // T : int    a : int& 
  foo(c); // T : const int  a : const int& 
  foo(cr);// T : const int  a : const int& 
}
```

### 1.3. auto 의 타입 추론은 tempalte 타입추론과 같음

```cpp
#include <iostream> 
using namespace std; 
  
int main() 
{ 
  int n = 10; 
  int& r = n; 
  const int c = n; 

  // 규칙 1. "auto a = 우변" 인경우  
  //         우변의 const, volatile, reference 모두 제거 후
  //         auto 결정
  // T    a = 함수인자
  // auto a = 우변

  auto a1 = n;// auto : int 
  auto a2 = r;// auto : int 
  auto a3 = c;// auto : int 

  // 규칙 2. "auto& a = 우변".  
  //         우변의 const, volatile 유지.  
  //         reference는 제거하고 auto 결정
  // T&    a = 함수인자 
  // auto& a = 우변 
  auto& a4 = n; // auto : int    a4 : int& 
  auto& a5 = r; // auto : int    a5 : int& 
  auto& a6 = c; // auto : const int  a6 : const int& 
  // a6 = 10; //  error 

  // auto 와 배열 문제 
  // auto  a1 = 배열 : a1은 포인터 
  // auto& a2 = 배열 : a2는 배열를 가리키는 참조 
  int x[3] = { 1,2,3 }; 

  auto  a7 = x; // int* a7 = x; 
  auto& a8 = x; // int (&a8)[3] = x; 

  cout << typeid(a7).name() << endl; // Pi
  cout << typeid(a8).name() << endl; // A3_i
}
```

### 1.4. decltype

```cpp
#include <iostream> 
using namespace std; 

int& foo() { return x; } 

int main() 
{ 
  int n = 10; 
  int& r = n; 
  const int c = 10; 
  int* p = &n; 

  // 규칙 1. () 안에 심볼의 이름만 있는 경우 
  //        해당 심볼의 선언과 완전히 동일한 타입 
  decltype(n)  d1; // int 
  decltype(r)  d2; // int&
  decltype(c)  d3; // const int
  decltype(p)  d4; // int* 

  // 규칙 2. 변수이름에 연산자등을 포함한 표현식이 있는 경우
  // 표현식이 왼쪽에 올수 있으면 : 참조타입 
  // 표현식이 왼쪽에 올수 없으면 : 값 타입 

  decltype(*p) d5; // int& 

  int x[3] = { 1,2,3 };
  decltype(x[0]) d6;  // int& 
  decltype(n + n) d7; // (n+n) = 10 이므로 int
  decltype(++n) d8;   // int&
  decltype(n++) d9;   // int

  decltype(n = 20) d10; // (n = 20) = 10; // int& 


  int y[3] = { 1,2,3 }; 

  // d와 a의 타입은 ? 
  decltype( y[0] ) d;  // int&  
  auto a = y[0]; // auto는 참조를 제거하므로 a는 int

  auto ret = foo();   // ret 타입은 int 
  ret = 20; 

  // decltype(함수호출식) : 함수호출의 결과로 나오는타입. 즉 함수 반환 타입 
  decltype( foo() ) ret2 = foo(); // int&

  // C++14 에서 나온 표현식
  // 우변으로 타입을 결정하는데 규칙은 decltype 규칙
  decltype(auto) ret3 = foo();  // int&
}
```

## 참고
codenuri 강석민 강사 강의 내용기반으로 정리한 내용입니다.  
[코드누리](https://github.com/codenuri)

<br>

    This is personal diary for study documents.
    Please comment if I'm wrong or missing something else 😄. 

[Top](#){: .btn .btn--primary }{: .align-right}