---
published: true
title:  "[Programming] C++ Remove pointer"
excerpt: "C++에 대해 알아보기, Remove pointer"

categories:
  - Cpp
tags:
  - [C++, Cpp, RemovePointer]

toc: true
toc_sticky: true
 
date: 2022-11-10
last_modified_at: 2022-11-10
---

## 1. Remove pointer
- traits의 종류
  1. 타입 질의 : is_xxx<T>::value
  2. 번형 타입얻기 : xxx<T>::type

- 구조체 템플릿을 만들고 원하는 타입을 얻도록 부분 특수화시켜 "int*" 를 "int" 와 "*" 로 분리할 수 있음

## 2. 코드로 알아보기
- [web_compiler](https://godbolt.org/) 에서 확인

```cpp
#include <iostream>
#include <typeinfo>
using namespace std; 

template<typename T> struct xremove_pointer 
{ 
  using type = T; 
}; 
template<typename T> struct xremove_pointer<T*> 
{ 
  using type = T; 
};

template<typename T> struct xremove_reference 
{ 
  using type = T; 
}; 
template<typename T> struct xremove_reference<T&> 
{ 
  using type = T; 
};

int main() 
{ 
  xremove_pointer<int*>::type n; // int
  cout << typeid(n).name() << endl;

  xremove_reference<int&>::type n1; // int 
  cout << typeid(n1).name() << endl;
}
```

```cpp
#include <iostream> 
using namespace std; 

// goo 타입 : int(char, double) 
int goo(char c, double d)  
{ 
  return 0;  
} 
  
// 1. primary template 만들고 type 넣기
template<typename T> struct result_type 
{ 
  using type = T; 
}; 

// 2. 원하는 타입을 얻을수 있도록 부분 특수화
// R(A1, A2) -> int(char, double)
template<typename R, typename A1, typename A2>  
struct result_type<R(A1, A2)> 
{ 
  using type = R; // int
}; 
  
template<typename T> void foo(T& a) 
{ 
  // 함수 반환타입 조사하기.(단, 함수의 인자가 2개일때만) 
  // T:int(char, double) 
  typename result_type<T>::type n = 0;          // int  

  cout << typeid(n).name() << endl; 
} 
  
int main() 
{ 
  foo(goo); 
}
```

- C++11 부터 제공되는 헤더 : type_traits

```cpp
#include <iostream> 
#include <type_traits> // C++11 부터 제공되는 헤더 
using namespace std; 

template<typename T> void foo(T a) 
{ 
  // 포인터 여부 조사 
  bool b1 = is_pointer<T>::value; 
  bool b2 = is_pointer_v<T>;

  // 포인터를 제거한 타입 얻기
  remove_pointer_t<T> n1;
  cout << typeid(n1).name() << endl; // int
} 

int main() 
{ 
  remove_pointer<int*>::type n1; 
  cout << typeid(n1).name() << endl; // int

  int n = 0;
  foo(&n);
}
```

## 참고
codenuri 강석민 강사 강의 내용기반으로 정리한 내용입니다.  
[코드누리](https://github.com/codenuri)

<br>

    This is personal diary for study documents.
    Please comment if I'm wrong or missing something else 😄. 

[Top](#){: .btn .btn--primary }{: .align-right}