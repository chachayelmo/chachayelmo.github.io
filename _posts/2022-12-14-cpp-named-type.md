---
published: true
title:  "[Programming] C++ NamedType"
excerpt: "C++에 대해 알아보기, NamedType"

categories:
  - Cpp
tags:
  - [C++, Cpp, NamedType]

toc: true
toc_sticky: true
 
date: 2022-12-14
last_modified_at: 2022-12-14
---

## 1. NamedType
- int, double 등의 타입이름을 사용하지 말고, 의미를 가지는 타입을 사용하자
- Width, Height, xPos, yPos 등

```cpp
#include <iostream> 
using namespace std; 
  
template<typename T, typename Parameter> class NamedType 
{ 
  T value; 
public: 
  NamedType(const T& v) : value(v) {} 
  T& get() { return value; } 
}; 
  
using Width = NamedType<int, struct WidthParameter>; 
using Height = NamedType<int, struct HeigthParameter>; 
using xPos = NamedType<int, struct xPosParameter>; 
using yPos = NamedType<int, struct yPosParameter>; 
  
void foo(Width w, Height h) {} 
  
int main() 
{ 
  // named argument idioms 
  foo(Width(10), Height(20)); // 이렇게 사용하자 

  // Objective-C 와 swift가 아래와 유사하게 사용
  //setRect(x:10, y:10, width:100, height:100); 

  Width w = 10; 
  Height h = 10; 

  // 단점. width와 height가 동일 타입. 다른 타입으로 만들어야 함 
  // w = h; 
  cout << w.get() << endl; 
}
```

```cpp
#include <iostream> 
#include <string> 
using namespace std; 
  
template<typename T, typename Parameter> class NamedType 
{ 
  T value; 
public: 
  NamedType(const T& v) : value(v) {} 
  NamedType(T&& v) : value(std::move(v)) {} 
  T& get() { return value; }                // 비상수 get 
  const T& get() const { return value; }    // 상수 get 
}; 
  
using FirstName = NamedType<string, struct stringP>; 
  
int main() 
{ 
  FirstName f1("aaa"); 
  const FirstName f3("aaa"); 
  FirstName f2(move(f1)); 
}
```

## 참고
codenuri 강석민 강사 강의 내용기반으로 정리한 내용입니다.  
[코드누리](https://github.com/codenuri)  
[NamedType](https://github.com/joboccara/NamedType)

<br>

    This is personal diary for study documents.
    Please comment if I'm wrong or missing something else 😄. 

[Top](#){: .btn .btn--primary }{: .align-right}