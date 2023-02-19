---
published: true
title:  "[C++] 오버라이드(override)"
excerpt: "C++에 대해 알아보기, 오버라이드"

categories:
  - Cpp
tags:
  - [오버라이드, C++, Cpp, override]

toc: true
toc_sticky: true
 
date: 2022-11-01
last_modified_at: 2022-11-01
---

## 1. Override
- 컴파일러에게 가상함수를 재정의하고 있다고 알려 주는 것
- 기반 클래스의 함수와 다른 경우 에러 발생하게 해줌

## 2. 코드로 알아보기
- [web_compiler](https://godbolt.org/) 에서 확인

```cpp
class Base 
{ 
public: 
  virtual void foo(int) {} 
  virtual void goo() const {} 
  virtual void hoo() {} 
  void koo() {} 
}; 
  
class Derived : public Base 
{ 
public: 
  virtual void foo(double)  {} 
  // virtual void goo() override {} //error
  // virtual void hooo()override {} //error
  // void koo()override {} //error
}; 
int main() 
{ 
}
```

- const int* p1 = &n; // p1을 따라가면 const
- int* const p2 = &n; // p2가 const 
- int const* p3 = &n; // p3를 따라 가면 const 

```cpp
#include <iostream> 
using namespace std; 

template<typename T> class Base 
{ 
public: 
  virtual void foo(const T a)  {    cout << "Base foo" << endl;    } 
}; 
  
class Derived : public Base<int*> 
{ 
public: 
  // virtual void foo(const int* a) // 오버라이드라 base foo를 부름
  virtual void foo(int* const a) 
  {  
    cout << "Derived foo" << endl; 
  } 
}; 
  
int main() 
{ 
  Base<int*>* p = new Derived; 
  p->foo(0);
}
```

## 참고
codenuri 강석민 강사 강의 내용기반으로 정리한 내용입니다.  
[코드누리](https://github.com/codenuri)

<br>

    This is personal diary for study documents.
    Please comment if I'm wrong or missing something else 😄. 

[Top](#){: .btn .btn--primary }{: .align-right}