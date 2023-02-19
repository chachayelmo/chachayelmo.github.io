---
published: true
title:  "[C++] Range for"
excerpt: "C++에 대해 알아보기, Range for"

categories:
  - Cpp
tags:
  - [C++, Cpp, RangeFor]

toc: true
toc_sticky: true
author: chachayelmo
sitemap:
  changefreq : daily
  priority : 1.0
date: 2022-10-24
last_modified_at: 2022-10-24
---

## 1. Range for
- 기존 for loop를 대체하는 C++11의 새로운 for loop

## 2. 코드로 알아보기
- [web_compiler](https://godbolt.org/) 에서 확인

```cpp
#include <iostream> 
#include <vector> 
using namespace std; 
  
int main() 
{ 
    vector<int> v = { 1,2,3,4,5 }; 
  
    // 전통적인 for 문 
    for (int i = 0; i < v.size(); i++) 
    { 
        cout << v[i] << endl; 
    } 
  
    // C++11 새로운 for 문 - range for 
    for (int n : v) 
        cout << n << endl; 
  
    // 위 코드가 아래처럼 컴파일러가 자동으로 만듬 
    for (auto p = begin(v); p != end(v); ++p) 
    { 
        int n = *p; 
        //----------------- 
        cout << n << endl; 
    } 
  
}
```

- begin/end 재정의

```cpp
#include <iostream> 
using namespace std; 
  
struct Point3D 
{ 
    int x, y, z; 
  
    Point3D(int a = 0, int b = 0, int c = 0) : x(a), y(b), z(c) {} 
}; 
int* begin(Point3D& pd) { return &(pd.x); } 
int* end(Point3D& pd)   { return &(pd.z) + 1; } 

int main() 
{ 
    Point3D pd(1, 2, 3); 
  
    // 아래 코드가 동작하게 하려면 begin/end를 재정의 해야 됨
    for (auto n : pd) 
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