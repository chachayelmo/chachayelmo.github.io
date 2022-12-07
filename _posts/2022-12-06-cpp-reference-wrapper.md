---
published: true
title:  "[Programming] C++ reference_wrapper"
excerpt: "C++에 대해 알아보기, reference_wrapper"

categories:
  - Cpp
tags:
  - [C++, Cpp, ReferenceWrapper]

toc: true
toc_sticky: true
 
date: 2022-12-06
last_modified_at: 2022-12-06
---

## 1. reference_wrapper
- C++ 참조는 개념적으로 const, 대입 연산 시 참조가 이동하지 않고 참조가 가리키는 값이 이동
- reference_wrapper
  - C++11 이동 가능한 참조, 대입 연산시 시 참조가 이동!

## 2. 코드로 알아보기
- [web_compiler](https://godbolt.org/) 에서 확인

### 2.1. 문제점

```cpp
#include <iostream> 
using namespace std; 

int main() 
{ 
  int n1 = 10; 
  int n2 = 20; 
  int& r1 = n1; 
  int& r2 = n2; 

  r2 = r1;

  std::cout << n1 << std::endl; // 10 
  std::cout << n2 << std::endl; // 10 
  std::cout << r1 << std::endl; // 10 
  std::cout << r2 << std::endl; // 10 
}
```

### 2.2. reference_wrapper

```cpp
#include <iostream> 
using namespace std; 
  
template<typename T> struct xreference_wrapper 
{ 
  T* obj; 
  xreference_wrapper(T& t) : obj(&t) {} 

  // T& 으로의 암시적 변환을 허용해야 함
  // 변환 연산자 
  operator T&() { return *obj; } 
}; 
  
int main() 
{ 
  int n1 = 10; 
  int n2 = 20;  
  xreference_wrapper<int> r1 = n1; 
  xreference_wrapper<int> r2 = n2;     
    
  int& r3 = r1; // reference_wrapper는 진짜 참조와도 호환됨. 
                // r1.operator int&() 
  r2 = r1; 

  std::cout << n1 << std::endl; // 10 
  std::cout << n2 << std::endl; // 20 
  std::cout << r1 << std::endl; // 10 
  std::cout << r2 << std::endl; // 10 
}
```

```cpp
#include <iostream> 
#include <functional> 
using namespace std; 
  
void foo(int& a) { a = 100; } 
  
// 아래 함수가 bind 처럼 인자를 값으로 받고 있습니다. 
template<typename F, typename T>  
void valueForwarding(F f, T arg) // T(int*) arg -> int& 로 변환되어야 함 
                                 //reference wrapper는 포인터인데 참조로 암시적 형변환이 가능  
{ 
  f(arg); 
} 
  
// reference_wrapper를 만드는 도움함수 
template<typename T> 
reference_wrapper<T> xref(T& obj) 
{ 
  return reference_wrapper(obj); 
} 
  
int main() 
{ 
  int x = 0; 

  // 복사
  // valueForwarding(&foo, x);

  // 메모리 정보를 보내기 위해 주소를 보냄 
  // valueForwarding의 2번째 인자가 포인터인데 참조로 변환될 수 없어서 에러 
  // valueForwarding(&foo, &x); // error

  // reference_wrapper<int> r = x; 
  // valueForwarding(&foo, r); // ok

  // valueForwarding(&foo, reference_wrapper<int>(x)); // ok 
  valueForwarding(&foo, xref(x)); 

  std::cout << x << std::endl; 
}
```

```cpp
#include <iostream> 
#include <functional> 
#include <thread>  
using namespace std; 
void foo(int a, int& b) { b = 100; } 
  
int main() 
{ 
  int n = 0; 
  //thread t(&foo, 1, n);     // error 
  thread t(&foo, 1, ref(n));  // ok 

  t.join(); 
}
```

## 참고
[cppreference](https://en.cppreference.com/w/cpp/types/is_abstract)

<br>

    This is personal diary for study documents.
    Please comment if I'm wrong or missing something else 😄. 

[Top](#){: .btn .btn--primary }{: .align-right}