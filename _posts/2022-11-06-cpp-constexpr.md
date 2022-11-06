---
published: true
title:  "[Programming] C++ constexpr"
excerpt: "C++에 대해 알아보기, constexpr"

categories:
  - Cpp
tags:
  - [C++, Cpp, Constexpr]

toc: true
toc_sticky: true
 
date: 2022-11-06
last_modified_at: 2022-11-06
---

## 1. constexpr
- 컴파일 시간 상수

## 2. 코드로 알아보기

```cpp
#include <iostream> 
using namespace std; 
  
int main() 
{ 
  const int c = 10; // 컴파일 시간 상수 

  // int n = 10; 
  // const int c = n; // 실행시간 상수 

  // int* p = &c; // error. 상수 주소를 비상수를 가리키는 포인터에 담을수 없음
  // int* p = static_cast<int*>(&c); // error 
  int* p = const_cast<int*>(&c); // ok, 상수성을 제거하는 캐스팅 

  *p = 20; 

  cout << c  << endl; // 10 
  cout << *p << endl; // 20 
}
```

- C89 문법 : 배열의 크기는 컴파일할때 알아야 함
- C99 문법 : 배열의 크기로 변수도 가능, g++지원, vc++지원 안함

```cpp
#include <iostream> 
using namespace std; 
  
int main() 
{ 
  int s1 = 10;
  int a1[s1];

  const int s2 = 10; 
  int a2[s2]; // ok 

  const int s3 = s1; 
  int a3[s3]; // error 
} 
  
void foo(const int s) 
{ 
    int ar[s]; // error 
} 
```

- C++11 의 새로운 상수 만드는 키워드 : 컴파일 시간 상수만 만들수 있음

```cpp
#include <iostream>

int main()
{ 
  int n = 10; 
  const int c1 = n; // ok 
  const int c2 = 10;// ok 

  // C++11 의 새로운 상수 만드는 키워드
  // 컴파일 시간 상수만 만들수 있다. 
  // constexpr int c3 = n; // error 
  constexpr int c4 = 10;// ok 
} 
```

```cpp
#include <iostream>

constexpr int Add(int a, int b) 
{ 
  return a + b; 
} 

int main() 
{ 
  int a = 1, b = 2; 
  int x1[ Add(1, 2) ]; // ok 

  int x2[Add(a, b)]; // error. 배열에 크기는  실행 시 결정할수 없음
  int c = Add(a, b); // ok 
}
```

## 참고
codenuri 강석민 강사 강의 내용기반으로 정리한 내용입니다.  
[코드누리](https://github.com/codenuri)

<br>

    This is personal diary for study documents.
    Please comment if I'm wrong or missing something else 😄. 

[Top](#){: .btn .btn--primary }{: .align-right}