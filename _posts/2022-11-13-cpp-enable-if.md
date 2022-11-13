---
published: true
title:  "[Programming] C++ enable_if"
excerpt: "C++에 대해 알아보기, enable_if"

categories:
  - Cpp
tags:
  - [C++, Cpp, EnableIf]

toc: true
toc_sticky: true
 
date: 2022-11-13
last_modified_at: 2022-11-13
---

## 1. enable_if
```cpp
template< bool B, class T = void >
struct enable_if;
```
- B가 true이면 enable_if는 T와 같은 public member typedef type을 가짐
- 그렇지 않으면 typedef member가 없음
- 이 함수는 SFINAE를 활용하여, type traits 기반으로 후보를 제거하고 함수 오버로드, 특수화를 통해 함수를 분리
- 사용 가능한 형태
  - 추가적인 function argument (연산자 오버로드에는 적용되지 않음)
  - return type (생성자 및 소멸자에는 적용되지 않음)
  - 클래스/함수 템플릿

## 2. 코드로 알아보기
- [web_compiler](https://godbolt.org/) 에서 확인

```cpp
template<bool b, typename T = void> 
struct enable_if 
{ 
  typedef T type; 
};
// 부분 특수화 시에는 디폴트값을 표기하지 않음
// 하지만 primary의 내용대로 적용
template<typename T> 
struct enable_if<false, T> 
{ 
}; 
int main() 
{ 
  enable_if<true, int>::type n1;    // int  
  enable_if<true, double>::type n2; // double 
  enable_if<true>::type n3;         // void 
                                    // void 타입은 변수를 만들수 없으므로 error 
    
  enable_if<false, int>::type n4;   // error. 
                                    // 부분 특수화의 type 이 없음
}
```

- 모든 정수 타입에 대해서만 코드를 생성하는 템플릿

```cpp
#include <iostream> 
#include <type_traits> 
using namespace std; 

/*
// 방법 1. static_assert 
// 특징 : 조건을 만족하지 않으면 무조건 에러 
template<typename T> void foo(T a) 
{ 
    
  static_assert(is_integral<T>::value, 
                "error not integer type"); 
} 
*/ 

/*
 방법 2. enable_if 사용
 특징 : 조건을 만족하지 못하면 에러가 아니라 후보군에서 빠진다는 의미. 
        동일 이름의 다른 함수가 있다면 사용가능 
*/
template<typename T>  
typename enable_if<is_integral<T>::value>::type 
foo(T a) 
{ 
  cout << "T" << endl; 
} 
  
void foo(double) { cout << "double" << endl; } 
  
int main() 
{ 
  foo(10);    // T
  foo(3.4);   // double
  foo(3.4f);  // double
}
```

- printv 만들기 3가지 방법
  1. is_pointer, true_type, false_type
  2. enable_if
  3. if constexpr

```cpp
#include <iostream> 
#include <type_traits> 
using namespace std; 

// 방법 1. true_type/false_type 
template<typename T> void printv_imp(T v, true_type) 
{ 
  cout << v << " : " << *v << endl; 
} 
template<typename T> void printv_imp(T v, false_type) 
{ 
  cout << v << endl; 
} 
template<typename T> void printv1(T v) 
{ 
  printv_imp(v, is_pointer<T>()); 
}

// 방법 2. enable_if
template<typename T>
//typename enable_if<is_pointer<T>::value>::type 
enable_if_t<is_pointer_v<T>> 
printv2(T v) 
{ 
  cout << v << " : " << *v << endl; 
} 

template<typename T>
enable_if_t<!is_pointer_v<T>> 
printv2(T v) 
{ 
  cout << v << endl; 
} 

// 방법 3. if constexpr 
template<typename T> void printv3(T v) 
{ 
  if constexpr ( is_pointer<T>::value )         
    cout << v << " : " << *v << endl; 
  else 
    cout << v << endl; 
} 

int main() 
{ 
  int n = 10;
  printv1(n); 
  printv1(&n); 

  printv2(n); 
  printv2(&n); 

  printv3(n); 
  printv3(&n); 
}
```

## 참고
codenuri 강석민 강사 강의 내용기반으로 정리한 내용입니다.  
[코드누리](https://github.com/codenuri)
[cppreference](https://en.cppreference.com/w/cpp/types/enable_if)

<br>

    This is personal diary for study documents.
    Please comment if I'm wrong or missing something else 😄. 

[Top](#){: .btn .btn--primary }{: .align-right}