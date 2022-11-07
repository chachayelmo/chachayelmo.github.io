---
published: true
title:  "[Programming] C++ Template"
excerpt: "C++에 대해 알아보기, Template basic"

categories:
  - Cpp
tags:
  - [C++, Cpp, Template]

toc: true
toc_sticky: true
 
date: 2022-11-07
last_modified_at: 2022-11-07
---

## 1. Template
- 유사한 코드가 반복되면 함수를 만들지 말고 함수를 만드는 틀을 만듬
- 템플릿은 2번 컴파일 됨
  - 1. 템플릿 자체를 컴파일 - T와 관련없는 코드의 에러 유무 확인
  - 2. 실제 C++ 코드를 생성한 후 다시 컴파일 - T와 관련 있는 코드의 에러 유무 확인 

## 2. 코드로 알아보기
- [web_compiler](https://godbolt.org/) 에서 확인

```cpp
#include <iostream>

/* 
int square(int a) 
{ 
  return a * a; 
} 
double square(double a) 
{ 
  return a * a; 
} 
*/ 

template<typename T> // 템플릿 파라미터 
T square(T a)        // 호출 파라미터 
{ 
  return a * a; 
} 
  
int main() 
{ 
  square<int>(3); 
  square<double>(3.3); 
    
  // printf("%p\n", &square); // error, 어떤 type인지 모름
  printf("%p\n", &square<int>); // ok. 

  square(3); 
  square(3.3);  
}
```

```cpp
#include <iostream>

template<typename T> void foo(T a) 
{
  // 아래 두개의 함수는 다른 컴파일 에러를 뱉음
  // goo();  // error, there are no arguments to 'goo' that depend on a template parameter
  goo(a);
} 
int main() 
{ 
  foo(10);
}
```

## 참고
codenuri 강석민 강사 강의 내용기반으로 정리한 내용입니다.  
[코드누리](https://github.com/codenuri)

<br>

    This is personal diary for study documents.
    Please comment if I'm wrong or missing something else 😄. 

[Top](#){: .btn .btn--primary }{: .align-right}