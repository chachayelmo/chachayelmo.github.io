---
published: true
title:  "[Programming] C++ traits"
excerpt: "C++에 대해 알아보기, Traits"

categories:
  - Cpp
tags:
  - [C++, Cpp, Traits]

toc: true
toc_sticky: true
 
date: 2022-11-09
last_modified_at: 2022-11-09
---

## 1. traits
- 템플릿 인자 T에 대한 속성(특질, traits)를 조사하는 기술

```cpp
template<typename T> void printv(T a) 
{ 
    if (T 가 포인터 타입 이면) 
        cout << a << " : " << *a << endl; 
    else 
        cout << a << endl; 
} 
```

## 2. 코드로 알아보기
- [web_compiler](https://godbolt.org/) 에서 확인

- traits 만드는 방법
  1. 구조체 템플릿을 만듬
  2. value = false를 넣음
  3. 조건을 만족하는 부분특수화를 만들고 value = true로 변경

```cpp
#include <iostream> 
using namespace std; 
  
template<typename T> struct IsPointer 
{ 
  // enum { value = false };  // 예전 스타일 
  static constexpr bool value = false; // C++11 
}; 
template<typename T> struct IsPointer<T*> 
{ 
  //enum { value = true }; 
  static constexpr bool value = true; 
}; 
  
template<typename T> void foo(T a) 
{ 
  if ( IsPointer<T>::value ) 
    cout << "포인터" << endl; 
  else 
    cout << "포인터 아님" << endl; 
} 
  
int main() 
{ 
  int n = 10; 
  foo(n); 
  foo(&n); 
}
```

```cpp
#include <iostream> 
using namespace std; 
  
int a;    // a는 int 타입 
double d; // d는 double 
int x[3]; // x의 타입 int[3], T[N] 
  
template<typename T> struct IsArray 
{ 
  static constexpr bool value = false; 
  static constexpr int size   = -1; 
}; 
  
template<typename T, int N> struct IsArray<T[N]> 
{ 
  static constexpr bool value = true; 
  static constexpr int size = N; 
}; 
  
template<typename T> void foo(T& a) 
{ 
  // T : int[3] 
  if (IsArray<T>::value) 
    cout << "배열 입니다. 크기는 " << IsArray<T>::size << endl; 
  else 
    cout << "배열 아님" << endl; 
} 
int main() 
{ 
  int x[3] = { 1,2,3 }; 
  foo(x);
}
```

- C++11 표준 type_traits

```cpp
#include <iostream>
#include <type_traits>
using namespace std; 

// & 있고 없음에 따라 한정적으로 밖에 사용 불가
// template<typename T> void foo(T& a) 
template<typename T> void foo(T a)
{ 
  if (is_pointer<T>::value) 
    cout << "포인터" << endl; 
  else 
    cout << "포인터아님" << endl; 
} 
int main() 
{ 
  int n = 0; 
  foo(&n);
  cout << is_pointer<int>::value << endl; 
}
```

```cpp
#include <iostream> 
#include <type_traits> 
using namespace std; 

// 포인터 일때만 사용 
template<typename T> void printv_pointer(T a) 
{ 
  cout << a << " : " << *a << endl; 
} 
// 포인터가 아닐때만 사용 
template<typename T> void printv_not_pointer(T a) 
{ 
  cout << a << endl; 
} 
template<typename T> void printv(T a) 
{ 
  if (is_pointer<T>::value)    
    printv_pointer(&a); 
  else 
    printv_not_pointer(a); 
} 
  
  
int main() 
{ 
  int    n = 10; 
  printv(n);   
}
```

- integral_constant 페이지에서 확인

```cpp
#include <iostream> 
#include <type_traits> 
using namespace std; 

// 포인터 일때만 사용 
template<typename T> void printv_imp(T a, YES) 
{ 
    cout << a << " : " << *a << endl; 
} 
  
// 포인터가 아닐때만 사용 
template<typename T> void printv_imp(T a, No)
{ 
    cout << a << endl; 
} 
  
template<typename T> void printv(T a) 
{
    // T의 포인터 여부에 따라 다른 함수 호출 
    printv_imp(a, is_pointer<T>::value );
} 

int main() 
{ 
    int    n = 10; 
    printv(n);   
};
```

## 참고
codenuri 강석민 강사 강의 내용기반으로 정리한 내용입니다.  
[코드누리](https://github.com/codenuri)

<br>

    This is personal diary for study documents.
    Please comment if I'm wrong or missing something else 😄. 

[Top](#){: .btn .btn--primary }{: .align-right}