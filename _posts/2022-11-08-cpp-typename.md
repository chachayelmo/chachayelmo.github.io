---
published: true
title:  "[Programming] C++ typename"
excerpt: "C++에 대해 알아보기, typename"

categories:
  - Cpp
tags:
  - [C++, Cpp, Typename]

toc: true
toc_sticky: true
 
date: 2022-11-08
last_modified_at: 2022-11-08
---

## 1. typename
- T::DWARD
  - DWORD "값"으로 해석
- typename T::DWORD
  - DWORD "타입"으로 해석
- typename AAA:DWORD
  - AAA라는 타입을 사용하므로 typename을 붙일 필요가 없음

## 2. 코드로 알아보기
- [web_compiler](https://godbolt.org/) 에서 확인

```cpp
#include <iostream>
#include <typeinfo>

using namespace std;

struct Test 
{ 
  enum { value = 0 }; 
  static int data; 
  typedef int DWORD; 
  struct nested {}; 
}; 
int Test::data = 0; 
  
int main() 
{ 
  int p = 1; 

  // 아래 코드에서 *의 의미는?
  Test::value * p; // enum 곱하기 p 
  cout << typeid(Test::value).name() << '\n';

  Test::data  * p; // int 곱하기 p
  cout << typeid(Test::data).name() << '\n';

  // Test::DWORD * p; // error, 포인터 변수 선언 conflict
  cout << typeid(Test::DWORD).name() << '\n';

  // Test::nested* p; // error, 포인터 변수 선언 conflict
  cout << typeid(Test::nested).name() << '\n';  
}
```

- 템플릿에 의존적으로 타입을 꺼내려면 typename을 붙여야 함

```cpp
#include <iostream> 
using namespace std; 

class AAA 
{ 
public: 
  enum { value = 10}; 
  typedef int DWORD; 
}; 
int p = 0; 
  
template<typename T> void foo(T a) 
{ 
  // T::DWORD * p;      // error, DWORD를 값으로 해석 
  typename T::DWORD* p; // DWORD를 타입으로 해석 
} 
  
int main() 
{ 
  AAA aaa; 
  foo(aaa); 
}
```

## 참고
codenuri 강석민 강사 강의 내용기반으로 정리한 내용입니다.  
[코드누리](https://github.com/codenuri)

<br>

    This is personal diary for study documents.
    Please comment if I'm wrong or missing something else 😄. 

[Top](#){: .btn .btn--primary }{: .align-right}