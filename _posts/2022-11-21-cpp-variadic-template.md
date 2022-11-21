---
published: true
title:  "[Programming] C++ 가변인자 템플릿"
excerpt: "C++에 대해 알아보기, Variadic template"

categories:
  - Cpp
tags:
  - [C++, Cpp, VariadicTemplate]

toc: true
toc_sticky: true
 
date: 2022-11-21
last_modified_at: 2022-11-21
---

## 1. 가변인자 템플릿
- 템플릿을 사용해서 임의의 갯수의 인자를 받는 방법

## 2. 코드로 알아보기
- [web_compiler](https://godbolt.org/) 에서 확인

### 2.1. 기본 모양

```cpp  
// 가변인자 클래스 템플릿 
template<typename ... T> class tuple {}; 

// 가변인자 함수 템플릿 
template<typename ... Types>  
void foo(Types ... args) {} 
  
int main() 
{ 
  tuple<> t0; 
  tuple<int, double, char> t; 

  foo(10, "AA", 4.5); // Types : int, const char*, double 
                      // args : 10, "AA", 4.5 
                      // foo(int, const char*, double)인 함수가 생성 
}
```

### 2.2. Parameter pack, pack expansion
- args : Parameter pack
- Types : template parameter pack
- pack expansion : 팩 안의 요소를 , 를 사용해서 나열

```cpp
#include <iostream> 
using namespace std; 
  
void goo(int a, double d, const char* s) 
{ 
  cout << "goo" << endl; 
} 
  
template<typename ... Types>  
void foo(Types ... args ) 
{ 
  // args : parameter pack
  // Types : template paramter pack 

  // sizeof...(pack 이름) 
  cout << sizeof...(args) << endl; 
  cout << sizeof...(Types) << endl; 
  
  // goo(args); // error. args는 pack 이다. 
  goo(args...);  // goo(3, 4.5, "hello") 
}  
  
int main() 
{ 
  foo(3, 4.5, "hello"); 
}
```

```cpp
#include <iostream> 
void goo(int a, int b, int c) {} 
int hoo(int a) { return -a; } 
void print(int a, int b, int c)
{
    std::cout << a << ' ' << b << ' ' << c << '\n';
}

template<typename ... Types>  
void foo(Types ... args) 
{ 
  // goo(args); // error 
  goo(args...); // goo(1,2,3) 
  print(args...);

  goo(++args...); // goo(++E1, ++E2, ++E3) 
  print(args...);

  // goo(hoo(args...)); // error. hoo는 인자가 한개 
  goo(hoo(args)...); // goo(hoo(E1), hoo(E2), hoo(E3))
} 
  
int main() 
{ 
  foo(1, 2, 3); 
}
```

```cpp
#include <iostream> 
using namespace std; 
  
template<typename T> void print(T a) 
{ 
    cout << a << endl; 
}
  
template<typename ... Types> 
void foo(Types ... args) 
{ 
  // print(args...); // error
  // print(args)...; // error, print(E1), print(E2), print(E3) 

  // pack 은 함수인자, 배열 초기화 등에서만 확장 가능 
  // args...; // error 
  int x[] = { args... }; //ok 

  int dummy[] = { 0, (print(args), 0)... }; 
             // { (print(1), 0), print(2), print(3)} 
} 
    
int main() 
{ 
  foo(); 
  foo(1, 2, 3);
}
```

### 2.3. args 안에 요소를 꺼내는 방법
1. pack expansion
2. 재귀로 가변인자 전달된 인자 꺼내기 

```cpp
#include <tuple> 
using namespace std; 
  
template<typename ... Types> 
void foo(Types ... args) 
{ 
  // 방법 1. pack expansion 
  int arr[] = { args... }; 

  // 타입이 다르면 tuple 
  tuple<Types...> t3(args...);
}

int main() 
{ 
    foo(1, 2, 3); 
}
```

```cpp
#include <iostream> 
using namespace std; 

void foo() {} 

// 방법 2. 재귀
template<typename T, typename ... Types>  
void foo(T value, Types ... args) 
{ 
  static int cnt = 0; 
  cout << value << endl; 

  if constexpr( sizeof...(args) > 0) 
      foo(args...); // foo( 4.5, "hello") 
                    // foo("hello") 
                    // foo() 
}

int main() 
{ 
  foo(3, 4.5, "hello"); // value : 3,    args : 4.5, "hello" 
}
```

## 참고
codenuri 강석민 강사 강의 내용기반으로 정리한 내용입니다.  
[코드누리](https://github.com/codenuri)  

<br>

    This is personal diary for study documents.
    Please comment if I'm wrong or missing something else 😄. 

[Top](#){: .btn .btn--primary }{: .align-right}