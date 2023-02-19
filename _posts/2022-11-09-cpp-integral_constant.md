---
published: true
title:  "[C++] Integral_constant"
excerpt: "C++에 대해 알아보기, Integral constant"

categories:
  - Cpp
tags:
  - [C++, Cpp, IntegralConstant]

toc: true
toc_sticky: true
author: chachayelmo
sitemap:
  changefreq : daily
  priority : 1.0
date: 2022-11-10
last_modified_at: 2022-11-10
---

## 1. Integral constant
- int / bool 같은 primitive 타입 value를 타입으로 만드는 기술

## 2. 코드로 알아보기
- [web_compiler](https://godbolt.org/) 에서 확인

```cpp
#include <iostream> 
using namespace std; 

template<int N> struct int2type 
{ 
  enum { value = N }; 
}; 

//구글에서 "C++ Idioms"  첫번째
// 포인터 일때만 사용 
template<typename T> void printv_imp(T a, int2type<1>) 
{ 
  cout << a << " : " << *a << endl; 
} 

// 포인터가 아닐때만 사용 
template<typename T> void printv_imp(T a, int2type<0>) 
{ 
  cout << a << endl; 
} 
  
template<typename T> void printv(T a) 
{ 
  printv_imp(a,  
             int2type<is_pointer<T>::value>() ); 
} 
  
int main() 
{ 
  int    n = 10; 
  printv(&n);
  printv(n);
}
```

- int2type을 발전 시켜 만든 기술이 integral_constant

```cpp
// C++11 표준 
// int2type을 발전 시켜 만든 도구 
template<typename T, T N> struct integral_constant 
{ 
    static constexpr T value = N; 
}; 
  
integral_constant<short, 0> s0; 
integral_constant<short, 1> s1; 
integral_constant<int, 0>   n0; 
  
// true / false : 참거짓을 나타내는 값.  같은타입 
// true_type / false_type : 참 거짓을 나타내는 타입. 
typedef integral_constant<bool, true> true_type; 
typedef integral_constant<bool, false> false_type; 
  
// is_pointer를 아래 처럼 만듬
template<typename T> struct is_pointer : false_type {}; 
  
template<typename T> struct is_pointer<T*>:true_type {}; 
```

- 위 코드가 type_traits 에 있음

```cpp
#include <iostream> 
using namespace std; 

#include <type_traits> 
  
template<typename T> void printv_imp(T a, true_type) 
{ 
  cout << a << " : " << *a << endl; 
} 
template<typename T> void printv_imp(T a, false_type) 
{ 
  cout << a << endl; 
} 
template<typename T> void printv(T a) 
{ 
  printv_imp(a, is_pointer<T>());  
} 
  
int main() 
{ 
  int    n = 10; 
  printv(&n); 
  printv(n);
}
```

- T가 포인터인지 조사하는 3가지 방법

```cpp
#include <iostream>
#include <type_traits> 
using namespace std; 
  
template<typename T> void foo_imp(T a, true_type)   
{ 
  *a = 10; // ok
} 
template<typename T> void foo_imp(T a, false_type) {} 
template<typename T> void foo(T a) 
{ 
  // 방법 1. is_pointer<T>::value 를 if로 조사 
  if (is_pointer<T>::value) 
  { 
      //*a = 10; // 단, 이런 표기법은 사용할 수 없음
  } 
  else 
  { 
  } 

  // 방법 2. true_type/false_type을 사용한 함수 오버로딩 
  foo_imp(a, is_pointer<T>()); 

  // 방법 3. is_pointer<T>::value 를 if constexpr로 조사, C++17부터 가능 
  if constexpr (is_pointer<T>::value) 
  { 
    *a = 10; // ok   
  } 
} 
  
int main() 
{ 
  int n = 0; 
  foo(n); 
}
```

## 참고
codenuri 강석민 강사 강의 내용기반으로 정리한 내용입니다.  
[코드누리](https://github.com/codenuri)

<br>

    This is personal diary for study documents.
    Please comment if I'm wrong or missing something else 😄. 

[Top](#){: .btn .btn--primary }{: .align-right}