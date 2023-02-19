---
published: true
title:  "[C++] 상수 멤버 함수(Const member function)"
excerpt: "C++에 대해 알아보기, Const member"

categories:
  - Cpp
tags:
  - [상수, 멤버, 함수, C++, Cpp, Const]

toc: true
toc_sticky: true
author: chachayelmo
sitemap:
  changefreq : daily
  priority : 1.0
date: 2022-10-15
last_modified_at: 2022-10-15
---

## 1. Const member function
- 요약
  1. const 객체는 const 멤버함수만 콜할 수 있음
  2. 그냥 객체여도 const 멤버함수를 콜할 수 있음
  3. 객체의 상태가 변하지 않는 모든 멤버함수는 반드시 상수멤버함수로 하는 것이 좋음

## 2. 코드로 알아보기
- [web_compiler](https://godbolt.org/) 에서 확인

### 2.1. 상수 객체와 멤버 함수
- 상수 객체는 상수 멤버 함수만 호출 가능
- 아래에서 print() 멤버 함수는 반드시 상수 멤버 함수 이어야 함

```cpp
#include <iostream> 
using namespace std; 
  
class Point 
{ 
public: 
    int x; 
    int y; 
  
    Point(int a = 0, int b = 0) : x(a), y(b) {} 
    void set(int a, int b) { x = a; y = b; } 
  
    void print()
    // void print() const, 상수 멤버 함수
    { 
        // x = 10;  
        // error. 상수 멤버함수에서는 모든 멤버가 상수취급
        // 값을 변경할 수 없다. 
        cout << x << ", " << y << endl; 
    } 
}; 

int main() 
{ 
  const Point p(1, 2); // 상수 객체 

  p.x = 10;     // error 
  p.set(10, 20);// error 
  p.print();    // error. 
} 
```

### 2.2. getter는 상수 멤버함수
- 상수 멤버 함수는 선택이 아닌 필수
- 객체의 상태를 변경하지 않는 모든 멤버함수는 반드시 상수 멤버 함수로 해야 함

```cpp
#include <iostream>

struct Rect 
{ 
    int xpos, ypos, width, height; 
public: 
    Rect(int x = 0, int y = 0, int w = 0, int h = 0) 
        : xpos(x), ypos(y), width(w), height(h) {} 
  
    int getArea() const { return width * height; } 
}; 
  
//void foo(Rect r) // call by value 
void foo(const Rect& r) 
{ 
    int n = r.getArea(); // ? 
} 
int main() 
{ 
    Rect r(1, 1, 10, 10); // 상수 객체 아님. 
  
    int n = r.getArea(); // ok 
    std::cout << n << "\n";
    foo(r); 
}
```

### 2.3. header, source 파일의 const
- 상수 멤버 함수에서 const 키워드는 선언과 구현에 모두 있어야 함

```cpp
// Rect.h 
class Rect 
{ 
    int x, y, w, h; 
public: 

    int getArea() const; 
};
```
```cpp
// Rect.cpp 
int Rect::getArea() const  
{ 
    return w * h; 
} 
int main() 
{ 
} 
```

## 참고
codenuri 강석민 강사 강의 내용기반으로 정리한 내용입니다.  
[코드누리](https://github.com/codenuri)

<br>

    This is personal diary for study documents.
    Please comment if I'm wrong or missing something else 😄. 

[Top](#){: .btn .btn--primary }{: .align-right}