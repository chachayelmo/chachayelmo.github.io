---
published: true
title:  "[Programming] C++ invoke"
excerpt: "C++에 대해 알아보기, invoke"

categories:
  - Cpp
tags:
  - [C++, Cpp, Invoke]

toc: true
toc_sticky: true
 
date: 2022-12-01
last_modified_at: 2022-12-01
---

## 1. invoke
- 일반함수 포인터와 멤버함수 포인터 모두 동일한 방식으로 호출되게 하기

## 2. 코드로 알아보기
- [web_compiler](https://godbolt.org/) 에서 확인

### 2.1. 함수객체

```cpp
#include <iostream> 
using namespace std; 
  
class Test 
{ 
public: 
  void foo() &  { cout << "foo &" << endl; } 
  void foo() && { cout << "foo &&" << endl; } 

  void operator()() & { cout << "operator () &" << endl; } 
  void operator()() &&{ cout << "operator () &&" << endl; } 
}; 
  
int main() 
{ 
  Test t; 
  t.foo();      // foo() & 
  Test().foo(); // foo() && 
  t();          // operator () &
  Test()();     // operator () &&
}
```

```cpp
#include <iostream>
#include <functional>

using namespace std; 
  
class Test 
{ 
public: 
  void foo(int a) { cout << "foo" << endl; } 
}; 
  
void goo(int b) { cout << "goo" << endl; } 
  
int main() 
{ 
  void(Test::*f)(int) = &Test::foo; 

  Test t; 
  (t.*f)(0); // 멤버 함수 포인터로 호출 

  // 일반 함수 포인터라면 
  void(*f1)(int) = &goo; 
  f1(1); 

  // 일반함수 포인터와 멤버 함수 포인터 모두 동일한 방식으로 호출되게 하자 
  invoke(f1, 1);        // 일반 함수 포인터 
  invoke(f, &t, 0);    // 멤버 함수 포인터 
}
```

```cpp
#include <iostream> 
#include <functional>
using namespace std; 
  
class Test 
{ 
public: 
  void foo(int) { cout << "Test foo" << endl; } 
}; 
  
void foo(int a, int b) { cout << "func foo" << endl; } 
  
template<typename F, typename ... Types>  
decltype(auto) chronometry(F&& f, Types&& ... args) 
{ 
  // 임시객체를 받았을 때 f가 lvalue가 되기 때문에 이를 해결하기 위해 forward를 해줘야 함 
  //return std::forward<F>(f)(std::forward<Types>(args)...); 
  // invoke : C++17
  return std::invoke(std::forward<F>(f), std::forward<Types>(args)...); 
} 
  
int main() 
{ 
  chronometry(&foo, 10, 20);          // 일반함수 포인터
  Test test; 
  chronometry(&Test::foo, &test, 10); // 멤버함수 포인터
}
```

## 참고
codenuri 강석민 강사 강의 내용기반으로 정리한 내용입니다.  
[코드누리](https://github.com/codenuri)  
[cppreference](https://en.cppreference.com/w/cpp/utility/functional/invoke)

<br>

    This is personal diary for study documents.
    Please comment if I'm wrong or missing something else 😄. 

[Top](#){: .btn .btn--primary }{: .align-right}