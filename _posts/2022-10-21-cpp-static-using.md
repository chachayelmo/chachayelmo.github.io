---
published: true
title:  "[Programming] C++ Using"
excerpt: "C++에 대해 알아보기, Using"

categories:
  - Cpp
tags:
  - [C++, Cpp, Using]

toc: true
toc_sticky: true
 
date: 2022-10-21
last_modified_at: 2022-10-21
---

## 1. Using
- using, typedef은 용도가 비슷하며 타입의 별명을 만드는 것
- 차이
    - typedef는 template의 별명이 되지 못함
    - using은 template의 별명이 가능

## 2. 코드로 알아보기

```cpp
//typedef int DWORD; 
//typedef void(*F)(); 
  
// C++11 using 
using DWORD = int; 
using F = void(*)(); 
  
int main() 
{ 
    DWORD n; // int 
    F p;     // 함수 포인터 
}
```

- typedef는 템플릿의 별명이 불가능

```cpp
#include <set> 
#include <functional> 
#include <iostream> 
using namespace std; 

// 타입의 별명은 가능
// typedef set<int> SET; 
// typedef set<double> SETD;

// 템플릿의 별명은 에러
// template<typename T> typedef set SET;   // error 

// using은 템플릿의 별명 가능
template<typename T>
using SET = set<T>;
  
int main() 
{ 
    SET s2; //set<int> s;

    s2.insert(10); 
    s2.insert(15); 
    s2.insert(6); 

    for (auto n : s2) 
        cout << n << endl; 
}
```

## 참고
codenuri 강석민 강사 강의 내용기반으로 정리한 내용입니다.  
[코드누리](https://github.com/codenuri)

<br>

    This is personal diary for study documents.
    Please comment if I'm wrong or missing something else 😄. 

[Top](#){: .btn .btn--primary }{: .align-right}