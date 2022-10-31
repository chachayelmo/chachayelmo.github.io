---
published: true
title:  "[Programming] C++ Delete Default"
excerpt: "C++에 대해 알아보기, Delete Default"

categories:
  - Cpp
tags:
  - [C++, Cpp, Delete, Default]

toc: true
toc_sticky: true
 
date: 2022-10-31
last_modified_at: 2022-10-31
---

## 1. Delete default
- Delete default를 통해 복사생성자 또는 대입연산자 등을 지울 때 사용
- 사용자 정의 Literal 함수

## 2. 코드로 알아보기
- [web_compiler](https://godbolt.org/) 에서 확인

```cpp
#include <iostream>

int add(int a, int b) 
{ 
  return a + b; 
} 
// 함수를 선언만 제공 : 런타임 중에 사용 시 링크 에러 발생 
//double add(double a, double b); 
  
// 함수 삭제 문법 : 해당 함수 사용시 컴파일 에러 
double add(double a, double b) = delete; 
    
int main() 
{ 
  add(1, 2); 

  add(3.4, 2.3); 
}
```

- 컴파일러가 복사생성자와 대입연산자를 만들지 못하게 할 때 사용
- 디폴트생성자

```cpp
#include <iostream> 
#include <vector> 
using namespace std; 
  
class Point 
{ 
  int x, y; 
public: 
  Point(int a, int b) {} 
  // 컴파일러가 복사 생성자를 만들지 못하게 한다. 
  // 함수 삭제 문법은 주로 "복사생성자" "대입연산자" 
  // 를 지울때 사용 
  Point(const Point&) = delete; 
  void operator=(const Point&) = delete; 
  
// 예전에 사용했던 복사생성자를 사용하지 못하게 하는 법 - private에 선언 
//private: 
//    Point(const Point&);

//    Point() {} // 사용자가 만드는 디폴트 생성자 
  Point() = default; // 컴파일러에게 디폴트생성자를 만드는 법
}; 
  
int main() 
{ 
  Point p1(1, 2); 
// Point p2 = p1;  // Point p2(p1) error 

  Point p3; 
} 
```

## 참고
codenuri 강석민 강사 강의 내용기반으로 정리한 내용입니다.  
[코드누리](https://github.com/codenuri)

<br>

    This is personal diary for study documents.
    Please comment if I'm wrong or missing something else 😄. 

[Top](#){: .btn .btn--primary }{: .align-right}