---
published: true
title:  "[Programming] C++ is_abstract"
excerpt: "C++에 대해 알아보기, is_abstract"

categories:
  - Cpp
tags:
  - [C++, Cpp, IsAbstract]

toc: true
toc_sticky: true
 
date: 2022-12-04
last_modified_at: 2022-12-04
---

## 1. is_abstract

```cpp
template< class T >
struct is_abstract;
```

- T가 추상 클래스인 경우 멤버 상수 값을 true를 제공, 아닌 경우 false
  - 추상 클래스: 하나 이상의 순수 가상 함수를 선언 또는 상속하는 클래스

## 2. 코드로 알아보기
- [web_compiler](https://godbolt.org/) 에서 확인

```cpp
#include <iostream>
#include <type_traits>
 
struct A {
  int m;
};
 
struct B {
  virtual void foo();
};
 
struct C {
  virtual void foo() = 0;
};
 
struct D : C {};
 
int main()
{
  std::cout << std::boolalpha;
  std::cout << std::is_abstract<A>::value << '\n';  // false
  std::cout << std::is_abstract<B>::value << '\n';  // false
  std::cout << std::is_abstract<C>::value << '\n';  // true
  std::cout << std::is_abstract<D>::value << '\n';  // true
}
```

## 참고
[cppreference](https://en.cppreference.com/w/cpp/types/is_abstract)

<br>

    This is personal diary for study documents.
    Please comment if I'm wrong or missing something else 😄. 

[Top](#){: .btn .btn--primary }{: .align-right}