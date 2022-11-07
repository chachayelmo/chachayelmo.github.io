---
published: true
title:  "[Programming] C++ Lazy instantiation"
excerpt: "C++에 대해 알아보기, Lazy instantiation"

categories:
  - Cpp
tags:
  - [C++, Cpp, LazyInstantiation]

toc: true
toc_sticky: true
 
date: 2022-11-07
last_modified_at: 2022-11-07
---

## 1. Lazy instantiation
- 지연된 인스턴스화는 사용된 함수만 실제 C++ 코드로 생성
  - 인스턴스화 : 템플릿에서 실제 C++ 코드(함수/클래스) 가 생성되는 것 
  - 지연된 인스턴스화 : 사용된 함수만 실제 C++ 코드로 생성
- if constexpr ( 컴파일 시간 조건문 ) static_if

## 2. 코드로 알아보기
- [web_compiler](https://godbolt.org/) 에서 확인

```cpp
template<typename T> 
class A 
{ 
  T data; 
public: 
  void foo(T a) { *a = 0; } // error, 포인터만 오는지 확인할 수 없음
}; 

int main() 
{ 
  A<int> a; 
  a.foo(0); 
}
```

- 아래 코드는 웹 컴파일러에서 if contexpr 구문이 생성되지 않음을 확인할 수 있음

```cpp
#include <iostream>

template<typename T> void foo(T a) 
{ 
  // 인스턴스화
  // 실행시간 조건문, C++ 코드로 생성
  /*
  if (false)
      *a = 10; 
  */

  // if constexpr : 컴파일 시간 조건문
  // 조건이 false 인경우 C++ 코드를 생성하지 않음
  // std:c++17 
  if constexpr (false) 
    *a = 10;    
}

int main() 
{ 
  foo(0);
}
```

## 참고
codenuri 강석민 강사 강의 내용기반으로 정리한 내용입니다.  
[코드누리](https://github.com/codenuri)

<br>

    This is personal diary for study documents.
    Please comment if I'm wrong or missing something else 😄. 

[Top](#){: .btn .btn--primary }{: .align-right}