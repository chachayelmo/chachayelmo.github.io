---
published: true
title:  "[Programming] C++ rvalue"
excerpt: "C++에 대해 알아보기, rvalue"

categories:
  - Cpp
tags:
  - [C++, Cpp, rvalue]

toc: true
toc_sticky: true
 
date: 2022-11-22
last_modified_at: 2022-11-22
---

## 1. rvalue
- lvalue:
  - "=" 을 기준으로 왼쪽과 오른쪽에 모두 올 수 있음
  - 이름이 있음, 단일식을 넘어서 메모리에 존재함
  - "참조"를 리턴하는 함수
  - 주소연산자로 주소를 구할 수 있음
- rvalue:
  - "=" 을 기준으로 오른쪽에만 올 수 있음
  - 이름이 없음, 단일식에서만 사용
  - "값"을 리턴하는 함수
  - 주소연산자로 주소를 구할 수 없음
  - 임시객체, literal

## 2. 코드로 알아보기
- [web_compiler](https://godbolt.org/) 에서 확인

```cpp
#include <iostream> 
using namespace std; 
  
int x = 10; 
int  foo() { return x; } 
int& goo() { return x; } 

class Point {
public:
  int x, y;
  Point(int x, int y) : x(x), y(y) {}
  void set(int x, int y) {
      x = x;
      y = y;
  }
};

int main() 
{ 
  int v1 = 10; 
  int v2 = 20; 

  v1 = 10; // v1 : lvalue, 10 : rvalue 
  10 = v2; // error 
  v2 = v1; // ok

  foo() = 20; // error, 값반환은 임시객체이기 때문에 lvalue가 될 수 없음
  goo() = 20; // ok

  cout << &v1 << endl;
  cout << &10 << endl; // error 

  const int c = 10; 
  c = 20; // c는 수정 불가능한 lvalue 

  // rvalue는 상수인가? 답은 No, 단지 왼쪽에 올 수 없을 뿐임
  Point(1, 2).x = 10; // error 
  Point(1, 2).set(10, 20); //ok 
}
```

```cpp
#include <iostream> 
using namespace std; 
  
#define PRINT_L_R_VALUE(...)                                       \ 
if (is_lvalue_reference<decltype((__VA_ARGS__))>::value)           \ 
    printf(#__VA_ARGS__ " : lvalue\n");                            \ 
else if (is_rvalue_reference<decltype((__VA_ARGS__))>::value)      \ 
    printf(#__VA_ARGS__ " : rvalue(xvalue)\n");                    \ 
else                                                               \ 
    printf(#__VA_ARGS__ " : rvalue(prvalue)\n"); 

int main() 
{ 
  int n = 0; 
  n = 10; // ok, n은 lvalue 
  //n + 1 = 20; // error, n+1은 rvalue 

  ++n = 20; // ok, 메모리, lvalue 
  // n++ = 20; // error, 이전 값, rvalue 

  PRINT_L_R_VALUE(n);   // lvalue
  PRINT_L_R_VALUE(n++); // rvalue
  PRINT_L_R_VALUE(++n); // lvalue

  const int c = 0; 
  PRINT_L_R_VALUE(c);   // lvalue
}
```

### 2.1. rvalue와 참조 문법
- non const lvalue reference(일반 참조)는 lvalue만 참조 가능
- const lvalue reference(상수 참조)는 lvalue, rvalue 모두 참조 가능
- rvalue reference는 rvalue만 참조 가능

```cpp
#include <iostream>

class Point {
public:
  int x, y;
  Point(int x, int y) : x(x), y(y) {}
  void set(int x, int y) {
    x = x;
    y = y;
  }
};

int main() 
{ 
  int n = 10; 

  // 1. non const lvalue reference(일반 참조)는 lvalue만 참조
  int& r1 = n;  // ok 
  int& r2 = 10; // error 

  // 2. const lvalue reference는 lvalue 와 rvalue 모두 참조
  const int& r3 = n;  // ok 
  const int& r4 = 10; // ok 

  // 3. rvalue reference는 rvalue 만 참조
  int&& r5 = n;  // error 
  int&& r6 = 10; // ok 
  Point&& r = Point(0, 0); // 임시객체 
}
```

### 2.2. 참조와 함수 오버로딩

```cpp
#include <iostream> 
using namespace std; 
  
// 참조와 함수 오버로딩 
void foo(int& a)       { cout << "int&" << endl; }       // 1 
void foo(const int& a) { cout << "const int&" << endl; } // 2 
void foo(int&& a)      { cout << "int&&" << endl; }      // 3 
  
int main() 
{ 
  int n = 10; 

  foo(n);  // 1번 호출, 없으면 2번 
  foo(10); // 3번 호출, 없으면 2번 

  int& r1 = n; 
  foo(r1); // 1번 호출, 없으면 2번 


  int&& r2 = 10; // 10은 rvalue지만, r2는 이름있는 변수이므로 lvalue입니다. 
                  // "named rvalue reference is lvalue" 

  // 중요!. r2는 lvalue, 주소도 있고 이름도 있음 
  foo(r2); // 1번 호출, 없으면 2번 
  foo(static_cast<int&&>(r2)); // 3번 호출 
}
```

```cpp
typedef int& LR; 
typedef int&& RR; 
  
int main() 
{ 
  int n = 10; 

  LR r1 = n;    // ok 
  RR r2 = 10;   // ok 

  // reference collapsing 이라고 함, 결론적으로 && &&의 경우만 &&
  LR& r3 = n;   // int& & -> int& 
  RR& r4 = n;   // int&& & -> int& 

  LR&& r5 = n;  // int& && -> int& 
  RR&& r6 = 10; // int&& && -> int&& 

  // 주의 직접 레퍼런스의 레퍼런스를 만들수는 없음
  // typedef 나 템플릿에서만 허용됩니다. 
  int& & r7 = n; // error 
} 
```

## 참고
codenuri 강석민 강사 강의 내용기반으로 정리한 내용입니다.  
[코드누리](https://github.com/codenuri)

<br>

    This is personal diary for study documents.
    Please comment if I'm wrong or missing something else 😄. 

[Top](#){: .btn .btn--primary }{: .align-right}