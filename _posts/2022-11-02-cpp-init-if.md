---
published: true
title:  "[Programming] C++ Init_if"
excerpt: "C++에 대해 알아보기, init if"

categories:
  - Cpp
tags:
  - [C++, Cpp, InitIf]

toc: true
toc_sticky: true
 
date: 2022-11-02
last_modified_at: 2022-11-02
---

## 1. Init if
- if statement에 특정 구문을 넣을 수 있음

## 2. 코드로 알아보기
- [web_compiler](https://godbolt.org/) 에서 확인

```cpp
// C++17에서 추가된 문법 
int foo() { return 0; } 

int main() 
{ 
  int ret = foo(); 
  if (ret == 0) 
  { 
    //.... 
  }   
  // g++  : -std=c++17 
  // C++17 의 새로운 if, init if 라고 불리는 문법 
  if (int ret = foo(); ret == 0) 
  { 
  } 

  switch (int n = foo(); n) 
  {
    case 1: break; 
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