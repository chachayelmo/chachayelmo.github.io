---
published: true
title:  "[C++] 널포인터(nullptr)"
excerpt: "C++에 대해 알아보기, nullptr"

categories:
  - Cpp
tags:
  - [널포인터, C++, Cpp, Nullptr]

toc: true
toc_sticky: true
author: chachayelmo
sitemap:
  changefreq : daily
  priority : 1.0
date: 2022-10-24
last_modified_at: 2022-10-24
---

## 1. nullptr
- 가독성이 좋은 null
- Zero value 와 다르게 암시적 형변환이 불가능
- nullptr 은 nullptr_t 라는 타입
  - nullptr_t는 모든 타입의 포인터로 암시적 형변환 가능
## 2. 코드로 알아보기
- [web_compiler](https://godbolt.org/) 에서 확인

```cpp
#include <iostream> 
using namespace std; 
  
int main() 
{ 
  int  n1 = 0; 
  int* p1 = 0; // 0은 정수지만 포인터 암시적 형변환
  // int* p2 = 10; // error 

  int* p3 = nullptr; // 포인터 0 
  // int  n2 = nullptr; // error.  
} 
```

```cpp
#include <iostream> 
using namespace std; 
  
void foo(int n) { cout << "int" << endl; } 
void foo(double n) { cout << "double" << endl; } 
void foo(bool p) { cout << "bool" << endl; } 
void foo(void* p) { cout << "void*" << endl; } 

int main() 
{ 
  foo(0);     // int.    0은 정수형 리터럴 
  foo(0.0);   // double. 0.0은 실수형 리터럴 
  foo(false); // bool.   false은 bool 형 리터럴 

  foo(nullptr);// void*  nullptr은 pointer 리터럴 

  // nullptr_t 암시적 형변환
  nullptr_t a = nullptr; 
  void* p2 = a; 
}
```

## 참고
codenuri 강석민 강사 강의 내용기반으로 정리한 내용입니다.  
[코드누리](https://github.com/codenuri)

<br>

    This is personal diary for study documents.
    Please comment if I'm wrong or missing something else 😄. 

[Top](#){: .btn .btn--primary }{: .align-right}