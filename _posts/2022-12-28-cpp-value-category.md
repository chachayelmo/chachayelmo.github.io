---
published: true
title:  "[Programming] C++ value 카테고리"
excerpt: "C++에 대해 알아보기, value 카테고리"

categories:
  - Cpp
tags:
  - [C++, Cpp, Value, Category]

toc: true
toc_sticky: true
 
date: 2022-12-28
last_modified_at: 2022-12-28
---

## 1. Value 카테고리
- lvalue : lvalue reference return
- rvalue : rvalue reference return, value return
	- rvalue reference return : xvalue
	- value return : prvalue

## 2. Value 별 특징
- [cppreference](https://en.cppreference.com/w/cpp/language/value_category)
- lvalue  : copy, polymorphic	  , Id
- xvalue  : move, polymorphic	  , Id
- prvalue : move, non-polymorphic , no Id

- glvalue : lvalue + xvalue
- rvalue  : xvalue + prvalue

![Capture](https://user-images.githubusercontent.com/23397039/209789357-d5704d64-5924-4d3c-9a03-3eb9fad77e59.PNG)

## 3. 코드로 알아보기
- [web_compiler](https://godbolt.org/) 에서 확인

```cpp
#include <iostream>
using namespace std;

int x = 0;
int     f1() { return x; }
int&    f2() { return x; }
int&&   f3() { return move(x); }

int main()
{
    f1() = 10; // error , value return
    f2() = 20; // ok    , lvalue ref return
    f3() = 30; // error , rvalue ref return
}
```

```cpp
#include <iostream>
using namespace std;

struct Base {
    virtual void foo() { cout << "B::foo" << "\n"; }
};

struct Derived : public Base {
	virtual void foo() { cout << "D::foo" << "\n"; }
};

Derived d;
Base    f1() { return d; }
Base&   f2() { return d; }
Base&&  f3() { return move(d); }

int main()
{
    Base b1 = f1(); // 임시객체, move
    Base b2 = f2(); // copy
    Base b3 = f3(); // move

    f1().foo(); // B::foo, value return		 , 임시객체이므로 Base foo 호출
    f2().foo(); // D::foo, lvalue ref return , 이름을 가지므로 Derived foo 호출
    f3().foo(); // D::foo, rvalue ref return , 이름을 가지므로 Derived foo 호출
}
```

## 참고
codenuri 강석민 강사 강의 내용기반으로 정리한 내용입니다.  
[코드누리](https://github.com/codenuri)   
[cppreference](https://en.cppreference.com/w/cpp/language/value_category)

<br>

    This is personal diary for study documents.
    Please comment if I'm wrong or missing something else 😄. 

[Top](#){: .btn .btn--primary }{: .align-right}