---
published: true
title:  "[C++] 완벽한 전달자(Perfect forwarding)"
excerpt: "C++에 대해 알아보기, Perfect forwarding"

categories:
  - Cpp
tags:
  - [퍼펙트포워딩, 완벽한전달자, C++, Cpp, PerfectForwarding]

toc: true
toc_sticky: true
author: chachayelmo
sitemap:
  changefreq : daily
  priority : 1.0
comment: true
date: 2022-11-23
last_modified_at: 2022-11-23
---

## 1. Perfect forwarding
- 전달 받은 인자를 다른 함수에게 완벽하게 전달하는 기술

## 2. 코드로 알아보기
- [web_compiler](https://godbolt.org/) 에서 확인

### 2.1. 문제점

```cpp
#include <iostream> 
using namespace std; 
  
void foo(int  n) { n = 10; } 
void goo(int& n) { n = 10; } 
  
template<typename F, typename T> 
void chronometry(F f, T arg)          // 1. 값 참조라서 x의 값이 변하지 않음 
void chronometry(F f, T& arg)         // 2. rvalue를 받을 수 없음 
void chronometry(F f, const T& arg)   // 3. 상수 참조라 값을 변경할 수 없음 
{ 
  // 시간 기록 
  f(arg); 
  // 걸린 시간 출력 
} 
  
int main() 
{ 
  int x = 0; 
    
  chronometry(foo, 10);
  chronometry(goo, x);
  cout << x << endl;
}
```

### 2.2. 해결책
- lvalue, rvalue 참조를 2개 제공

```cpp
#include <iostream> 
using namespace std; 
  
void foo(int  n) { n = 10; } 
void goo(int& n) { n = 10; } 
  
// lvalue 참고, rvalue 참조 2개를 제공
template<typename F> 
void chronometry(F f, int& arg) 
{ 
  f(arg); 
} 
template<typename F> 
void chronometry(F f, int&& arg) 
{ 
  f(arg); 
} 
  
int main() 
{ 
  int x = 0; 
  chronometry(foo, 10); 
  chronometry(goo, x); 

  cout << x << endl; 
}
```

### 2.3. int&& 를 넘기려면?

```cpp
// 3_완벽한전달자 
#include <iostream> 
using namespace std; 
  
void foo(int  n) { n = 10; } 
void goo(int& n) { n = 10; } 
  
void hoo(int&& n) {} 

template<typename F> void chronometry(F f, int& arg) 
{ 
  // f(arg); 
  f(static_cast<int&>(arg)); // 캐스팅이 필요없지만 해도 문제는 없음 
} 

// main                chronometry           hoo(int&) 
// 10(rvalue)  ----------------------------> ok 
// 10(rvalue)  --> int&& arg(lvalue) ------> error 캐스팅 필요! 
  
template<typename F> void chronometry(F f, int&& arg) 
{ 
  // int&& arg = 10 / 10은 rvalue, args는 lvalue 
  // f(arg);                  // error! hoo(int&) 를 찾고 있음 
  f(static_cast<int&&>(arg)); // ok 
} 
  
int main() 
{ 
  int x = 0; 

  hoo(10); // ok 
  chronometry(hoo, 10); // ?

  cout << x << endl; 
}
```

### 2.4. Perfect forwarding
- perfect forwarding을 하려면
  1. 인자를 forwarding reference로 받음
  2. 받은 인자를 다른 곳에 보낼 대는 std::forward로 묶어서 전달

```cpp
#include <iostream> 
using namespace std; 
  
void foo(int  n) { n = 10; } 
void goo(int& n) { n = 10; } 
  
void hoo(int&& n) {} 

template<typename F, typename T> 
void chronometry(F f, T&& arg) 
{ 
  // 아래 캐스팅은 lvalue를 lvalue로 
  //              rvalue를 rvalue로 캐스팅
  // 10(rvalue) -> arg(lvalue) -> rvalue 
  // x(lvalue) -> arg(lvalue) -> lvalue 
  // f(static_cast<T&&>(arg));   
  f(std::forward<T>(arg)); // 이 함수가 위처럼 캐스팅
} 
  
int main() 
{ 
  int x = 0; 

  chronometry(foo, 10); 
  chronometry(goo, x); 

  cout << x << endl; 
}
```

### 2.5. 가변인자 템플릿과 함께

```cpp
#include <iostream> 
using namespace std; 
  
void foo(int n, int b, int c) { n = 10; } 
void goo(int& n) { n = 10; } 
  
template<typename F, typename ... Types> 
void chronometry(F f, Types&& ... args) 
{ 
  f(std::forward<Types>(args)...); 
} 
  
int main() 
{ 
  int x = 0; 

  chronometry(foo, 10, 20, 30); 
  chronometry(goo, x); 

  cout << x << endl; 
}
```

### 2.6. return type일 경우
- auto 인 경우 참조를 버리기 때문에 decltype(auto) 로 받아야 됨

```cpp
#include <iostream> 
#include <string> 
#include <chrono> 
  
using namespace std; 
class stop_watch;

template<typename F, typename ... Types> 
decltype(auto) chronometry(F f, Types&& ... args) // auto인 경우 참조를 버려서 int를 반환, 따라서 decltype(auto) 
{ 
  stop_watch sw; // 생성자에서 시간 기록 
                  // 소멸자에서 수행 시간 출력 
  return f(std::forward<Types>(args)...); 
} 
  
class stop_watch 
{ 
  std::string message; 
  std::chrono::high_resolution_clock::time_point start; 
public: 
  inline stop_watch(std::string msg = "") : message(msg) { start = std::chrono::high_resolution_clock::now(); } 
  inline ~stop_watch() 
  { 
    std::chrono::high_resolution_clock::time_point end = std::chrono::high_resolution_clock::now(); 
    std::chrono::duration<double> time_span = std::chrono::duration_cast<std::chrono::duration<double>>(end - start); 
    std::cout << message << " " << time_span.count() << " seconds." << std::endl; 
  } 
}; 
  
void foo() 
{ 
  for (int i = 0; i < 10000; i++) rand(); 
} 
  
int main() 
{ 
  chronometry(foo); //시간 측정 
  return 0; 
}
```

```cpp
#include <iostream> 
using namespace std; 
  
void foo(int) {} 
void foo(double) {} 
void goo(int*) {} 
  
template<typename F, typename ... Types> 
decltype(auto) chronometry(F f, Types&& ... args) 
{ 
  return f(std::forward<Types>(args)...); 
} 
  
int main() 
{ 
  // printf("%p\n", foo); // error, 모호함 때문에 
  printf("%p\n", static_cast<void(*)(int)>(foo));
  // chronometry(foo, 10); // error, 위처럼 캐스팅 필요 

  //------------- 
  int n = 0; 
  int* p1 = 0; // ok 
  // int* p2 = n; // error 

  goo(0); // ok 
  // chronometry(goo, 0); // error, int -> int* 안됨 

  chronometry(goo, nullptr); // ok   
}
```

## 참고
codenuri 강석민 강사 강의 내용기반으로 정리한 내용입니다.  
[코드누리](https://github.com/codenuri)

<br>

    This is personal diary for study documents.
    Please comment if I'm wrong or missing something else 😄. 

[Top](#){: .btn .btn--primary }{: .align-right}