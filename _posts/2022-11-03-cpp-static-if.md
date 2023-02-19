---
published: true
title:  "[C++] Static_if"
excerpt: "C++에 대해 알아보기, Static if, constexpr"

categories:
  - Cpp
tags:
  - [C++, Cpp, StaticIf]

toc: true
toc_sticky: true
author: chachayelmo
sitemap:
  changefreq : daily
  priority : 1.0
date: 2022-11-03
last_modified_at: 2022-11-03
---

## 1. Static If
- 컴파일타임 때 If statement의 true, false를 판단하는 기술
- 컴파일 과정에서 If condition이 true일 경우 false statement 가 버려짐
- 반대로 If condition이 false일 경우 true statement가 버려짐

## 2. 코드로 알아보기
- [web_compiler](https://godbolt.org/) 에서 확인

```cpp
#include <iostream> 
#include <type_traits> 
using namespace std; 
  
template<typename T> void printv(T a) 
{
  // if (std::is_pointer_v<T>) // error 
  if constexpr (std::is_pointer_v<T>) // true 
    cout << a << " : " << *a << endl; 
  else 
    cout << a << endl; 
} 
int main() 
{ 
  int n = 10; 
  printv(&n); 
  printv(n); 
} 
```

## 참고
codenuri 강석민 강사 강의 내용기반으로 정리한 내용입니다.  
[코드누리](https://github.com/codenuri)

<br>

    This is personal diary for study documents.
    Please comment if I'm wrong or missing something else 😄. 

[Top](#){: .btn .btn--primary }{: .align-right}