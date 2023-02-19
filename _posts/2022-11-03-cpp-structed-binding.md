---
published: true
title:  "[C++] 바인딩(Structured binding)"
excerpt: "C++에 대해 알아보기, 바인딩"

categories:
  - Cpp
tags:
  - [바인딩, C++, Cpp, StructuredBinding]

toc: true
toc_sticky: true
author: chachayelmo
sitemap:
  changefreq : daily
  priority : 1.0
date: 2022-11-03
last_modified_at: 2022-11-03
---

## 1. Structured binding
- 구조체의 모든 멤버 값을 한번에 꺼내는 기술
- 반드시 auto 타입만 가능
- 구조체와 배열도 가능
- C++17부터 가능

## 2. 코드로 알아보기
- [web_compiler](https://godbolt.org/) 에서 확인

```cpp
#include <iostream> 
using namespace std; 
  
struct Point 
{ 
  int x = 0; 
  int y = 0; 
};

int main() 
{ 
  Point p = { 1, 2 }; 

  int x = p.x; 
  int y = p.y; 

  // structured binding
  auto[x1, y1] = p;

  int arr[3] = { 1, 2, 3 }; 

  auto[a, b, c] = arr;

  auto& [r1, r2, r3] = arr; // ok
} 
```

## 참고
codenuri 강석민 강사 강의 내용기반으로 정리한 내용입니다.  
[코드누리](https://github.com/codenuri)

<br>

    This is personal diary for study documents.
    Please comment if I'm wrong or missing something else 😄. 

[Top](#){: .btn .btn--primary }{: .align-right}