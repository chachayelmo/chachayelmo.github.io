---
published: true
title:  "[Programming] C++ move/forward 차이"
excerpt: "C++에 대해 알아보기, move, forward 차이"

categories:
  - Cpp
tags:
  - [C++, Cpp, Move, Forward]

toc: true
toc_sticky: true
 
date: 2022-12-29
last_modified_at: 2022-12-29
---

## 1. forward, move 파헤치기
- static_cast<T&&>(arg) - T의 타입에 따라 rvalue lvalue 캐스팅 가능

## 2. move
- 함수 인자 ( T&& obj ) : lvalue, rvalue 모두 받음
- 리턴 타입 : rvalue로 캐스팅

```cpp
template<typename T>
typename remove_reference<T>::type&&
xmove(T&& obj) // obj로 들어오는 타입은 lvalue, rvalue 모두 받음
{
  // 리턴 타입을 T의 reference를 지우고 && 로 바꿔줌, 항상 rvalue reference가 됨
  return static_cast<typename remove_reference<T>::type &&>(obj);
}
```

## 3. forward
- 함수 인자 (typename remove_reference<T>::type&) : lvalue만 받음
- 리턴 타입 : T에 따라서 lvalue or rvalue로 캐스팅

```cpp
// template<typename T> T&& xforward(T& arg)          // T 타입으로 함수를 사용하지 않으면 알아서 추론해줌, xforward(var) : ok
template<typename T> 
T&& xforward(typename remove_reference<T>::type& arg) // T 타입으로 함수를 사용하지 않으면 에러를 발생해줌, xforward(var) : error
                                                      // T의 reference를 지우고 항상 T&가 되도록 함, 즉 lvalue만 받을 수 있음
{
  // 리턴 타입은 T의 타입에 따라 rvalue lvalue 캐스팅 가능
  return static_cast<T&&>(arg);
}
```

## 4. rvalue를 인자로 가지는 forward
- 함수 인자 : rvalue를 받음
- 리턴 타입 : T에 따라서 lvalue or rvalue로 캐스팅

```cpp
template<typename T>
T&& xforward(typename remove_reference<T>::type&& arg)  // T의 reference를 지우고 항상 T&&가 되도록 함, 즉 rvalue를 받을 수 있음
{
  // 리턴 타입은 T의 타입에 따라 rvalue lvalue 캐스팅 가능
  return static_cast<T&&>(arg);
}
```

## 예제

```cpp
#include <iostream>

using namespace std;

class Test
{
  int data;
public:
  int& get() & { return data; }
  int get() && { return data; }
};

void foo(int& a) { cout << "int&" << "\n"; }
void foo(int&& a) { cout << "int&&" << "\n"; }

// move
template<typename T>
typename remove_reference<T>::type&&
xmove(T&& obj)
{
  return static_cast<typename remove_reference<T>::type &&>(obj);
}

// forward, lvalue를 인자로 받음
template<typename T>
T&& xforward(typename remove_reference<T>::type& arg)
{
  return static_cast<T&&>(arg);
}

// forward, rvalue를 인자로 받음
template<typename T>
T&& xforward(typename remove_reference<T>::type&& arg)
{
  return static_cast<T&&>(arg);
}

// foo wrapper 함수
template<typename T> void wrapper(T&& obj){
  // get 함수의 타입도 넘겨주기 위해 forward로 또 씌어줌
  // 그런데 get 함수에서는 lvalue, rvalue 모두 넘어올 수 있기 때문에
  // forward에서 lvalue뿐만 아니라 rvalue를 인자로 받는 forward도 필요!
  using Type = decltype(xforward<T>(obj).get());
  foo( xforward<Type>( xforward<T>(obj).get() ) );
}

int main()
{
  Test t;
  foo(t.get());         // int&
  foo(Test().get());    // int&&
  wrapper(t);           // int&
  wrapper(Test());      // int&&
}
```

## 참고
codenuri 강석민 강사 강의 내용기반으로 정리한 내용입니다.  
[코드누리](https://github.com/codenuri)   

<br>

    This is personal diary for study documents.
    Please comment if I'm wrong or missing something else 😄. 

[Top](#){: .btn .btn--primary }{: .align-right}