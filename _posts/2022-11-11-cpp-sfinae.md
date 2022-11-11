---
published: true
title:  "[Programming] C++ SFINAE"
excerpt: "C++에 대해 알아보기, SFINAE"

categories:
  - Cpp
tags:
  - [C++, Cpp, SFINAE]

toc: true
toc_sticky: true
 
date: 2022-11-11
last_modified_at: 2022-11-11
---

## 1. SFINAE
- SFINAE = Substitution Failure is Not An Error
- 치환 실패는 에러가 아니다.

## 2. 코드로 알아보기
- [web_compiler](https://godbolt.org/) 에서 확인

```cpp
#include <iostream> 
using namespace std; 
 
// template<typename T>  
// void foo(T a)   { cout << "T" << endl;} // 2
// void foo(int a) { cout << "int" << endl; } // 1
// void foo(double a) { cout << "double" << endl; } // 3 
// void foo(char a)   { cout << "char" << endl; } // 4
void foo(...  ) { cout << "..." << endl; } // 5
  
int main() 
{ 
  foo(10);// 1. int로 정확한 타입 매칭 
          // 2. template 을 사용해서 foo(int) 생성 
          // 3. standard conversion, 표준 타입끼리의 암시적 형변환 
          // 4. 가변인자 함수 
}
```

```cpp
#include <iostream> 
using namespace std; 
  
template<typename T>  
typename T::type  foo(T a) { cout << "T" << endl; return 0; } // int::type foo(int a) 
void foo(...) { cout << "..." << endl; } 
  
int main() 
{ 
  foo(10); // 가변인자
}
```

```cpp
#include <iostream> 
using namespace std;   

// template<typename T> void foo(T a) 
// { 
//     typename T::type n; // 에러 
// } 

// SFINAE 기술을 3가지 위치에서 사용가능
// 1. 리턴 타입
template<typename T> 
typename T::type foo(T a) { cout << "T" << endl; return 0; } 
  
// 2. 함수인자에 사용 
template<typename T> 
void foo(T a, typename T::type n = 0) { } 
  
// 3. 템플릿 인자에 사용 
template<typename T,  
         typename T2 = typename T::type> 
void foo(T a) {} 
  
void foo(...) { cout << "..." << endl; } 
  
int main() 
{ 
  foo(10); 
}
```

## 참고
codenuri 강석민 강사 강의 내용기반으로 정리한 내용입니다.  
[코드누리](https://github.com/codenuri)

<br>

    This is personal diary for study documents.
    Please comment if I'm wrong or missing something else 😄. 

[Top](#){: .btn .btn--primary }{: .align-right}