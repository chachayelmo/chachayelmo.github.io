---
published: true
title:  "[C++] 사용자 정의 리터럴(User defined literal)"
excerpt: "C++에 대해 알아보기, User defined literal"

categories:
  - Cpp
tags:
  - [사용자, 정의, 리터럴, C++, Cpp, UserDefinedLiteral]

toc: true
toc_sticky: true
author: chachayelmo
sitemap:
  changefreq : daily
  priority : 1.0
date: 2022-10-26
last_modified_at: 2022-10-26
---

## 1. User defined literal
- 사용자 정의 Literal 함수

## 2. 코드로 알아보기
- [web_compiler](https://godbolt.org/) 에서 확인
- 연산자 재정의

```cpp
#include <iostream> 
using namespace std; 

class Meter 
{ 
  int value; 
public: 
  Meter(int n = 0) : value(n) {} 
}; 
  
Meter operator""_m(unsigned long long n) 
{ 
  cout << "operator m" << endl; 

  Meter m(n); 
  return m; 
} 
int main() 
{ 
  float f = 3.4f; 

  Meter v = 10_m; // operator""m(10) 함수를 부름
}
```

- 표준 literal 사용

```cpp
#include <iostream> 
#include <string> 
#include <complex> 
#include <thread> 
#include <chrono> 
  
// 표준 literal을 사용
using namespace std::literals;

void foo(std::string s) { std::cout << "string" << std::endl; } 
void foo(const char* s)  { std::cout << "char*" << std::endl; } 
  
int main() 
{ 
  foo("hello");   // const char* 
  foo("hello"s);  // string operator""s(const char*) 

  std::complex<double> c1(3, 0); // 3 + 0i 
  std::complex<double> c2(3);  // 3 + 0i 
  std::complex<double> c3(3i); // 0 + 3i 

  std::chrono::seconds sec = 1h + 3min + 20s; 

  std::cout << sec.count() << std::endl; 

  std::this_thread::sleep_for(10s); // 10s : seconds operator""s(10) 
}
```

## 참고
codenuri 강석민 강사 강의 내용기반으로 정리한 내용입니다.  
[코드누리](https://github.com/codenuri)

<br>

    This is personal diary for study documents.
    Please comment if I'm wrong or missing something else 😄. 

[Top](#){: .btn .btn--primary }{: .align-right}