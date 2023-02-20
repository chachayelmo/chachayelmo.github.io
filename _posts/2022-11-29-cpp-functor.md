---
published: true
title:  "[C++] 함수객체(Functor)"
excerpt: "C++에 대해 알아보기, Functor"

categories:
  - Cpp
tags:
  - [C++, Cpp, Functor]

toc: true
toc_sticky: true
author: chachayelmo
sitemap:
  changefreq : daily
  priority : 1.0
comment: true
date: 2022-11-30
last_modified_at: 2022-11-30
---

## 1. Functor
- 함수객체 (function object, functor)
  - 함수처럼 사용가능한 객체
- 장점:
  1. 상태를 가지는 함수
  2. 독립된 타입성
  3. 일반 함수보다 빠름 (인라인 치환성)

## 2. 코드로 알아보기
- [web_compiler](https://godbolt.org/) 에서 확인

### 2.1. 함수객체

```cpp
#include <iostream> 
using namespace std; 

struct Plus 
{ 
  constexpr int operator()(int a, int b) const 
  { 
    return a + b; 
  } 
}; 
int main() 
{ 
  Plus p; 
  int n = p(1, 2);  
  cout << n << endl; 

  // a + b;  // a.operator+(b) 
  // a - b;  // a.operator-(b) 
  // a();    // a.operator()() 
  // a(1,2); // a.operator()(1,2) 
}
```

```cpp
#include <iostream> 
#include <bitset> 
using namespace std; 
  
template<int N> class URandom 
{ 
  bitset<N> bs; 
  bool recycle; 
public: 
  URandom(bool b = false) : recycle(b) { bs.set(); } // 모든 비트를 1로 
  int operator()() 
  { 
    if (bs.none())  // 모두 0이면 
    { 
      if (recycle) 
        bs.set();   // 다시 모든 비트 1로 
      else 
        return -1; 
    } 

    int v = -1; 
    while (!bs.test(v = rand() % N)) 
    bs.reset(v); 
    return v; 
  } 
}; 
  
// 0 ~ 9 사이의 중복되지 않은 난수를 구하는 함수 
// 일반 함수는 동작은 있지만 상태가 없다. 
/* 
int urand() 
{ 
    return rand() % 10; 
} 
*/ 
int main() 
{ 
  URandom<20> r; 
  for (int i = 0; i < 15; i++) 
    //cout << urand() << endl; 
    cout << r() << endl; 
}
```

### 2.2. inline 함수

```cpp
#include <iostream>
using namespace std;

// 핵심 1. 인라인 치환은 컴파일 시간에 이루어짐
// 핵심 2. 인라인 함수라도 함수 포인터에 담아서 사용하면 인라인 치환이 될 수 없음 
int Add1(int a, int b) { return a + b; }
inline int Add2(int a, int b) { return a + b; } 
  
int main() 
{ 
  int n1 = Add1(1, 2); // 호출  
  int n2 = Add2(1, 2); // 치환, inline

  int(*f)(int, int); 

  f = &Add2;

  int a; 
  cin >> a;
  if (a == 0) f = &Add1; 

  int n3 = f(1, 2); // 호출 ? 치환 
}
```

### 2.3. 템플릿 + 함수객체

```cpp
#include <algorithm> 
using namespace std; 
  
// 1. 일반 함수는 자신만의 타입이 없다 
//    signature가 동일한 함수는 같은 타입이다. 
  
// 2. 함수객체는 자신만의 타입이 있다 
//    signature가 동일해도 모두 다른 타입이다. 
  
struct Less    { inline bool operator()(int a, int b) { return a < b; }}; 
struct Greater { inline bool operator()(int a, int b) { return a > b; }}; 
  
//------------------------------------------------------- 
// 정책 변경이 가능하고 정책의 인라인 치환이 되는 함수 만들기 
// 템플릿 + 함수 객체로 만드는 기법 
// 단점 : 템플릿으로 사용하면 코드메모리가 2개(Less, Greater) 만들어지기 때문에  
//          코드메모리 증가 
template<typename T> void Sort(int* x, int sz, T cmp) 
{ 
  for (int i = 0; i < sz - 1; i++) 
  { 
    for (int j = i + 1; j < sz; j++) 
    { 
      if (cmp(x[i], x[j])) 
        swap(x[i], x[j]); 
    } 
  } 
} 
  
int main() 
{ 
  int x[10] = { 1,3,5,7,9,2,4,6,8,10 }; 
  Less f1; f1(1, 2); 

  Sort(x, 10, f1); 
  Greater f2; f2(1, 2); 

  Sort(x, 10, f2); 
}
```

```cpp
#include <algorithm> 
using namespace std; 
  
inline bool cmp1(int a, int b) { return a < b; } 
inline bool cmp2(int a, int b) { return a > b; } 
struct Less { inline bool operator()(int a, int b) { return a < b; } }; 
struct Greater { inline bool operator()(int a, int b) { return a > b; } }; 
  
int main() 
{ 
  int x[10] = { 1,3,5,7,9,2,4,6,8,10 }; 
  // STL의 sort()의 모든 인자는 template 입니다. 

  // 1. 비교 정책으로 일반함수를 사용할때 
  // 장점 : 정책을 교체해도 sort() 기계어는 한개이다. 
  // 단점 : 정책이 인라인 치환 될 수 없다. 왜냐하면 컴파일러가 어떤걸 선택해야 될 지 모르기 때문에 
  sort(x, x + 10, cmp1); // sort(int*, int*, bool(*)(int, int)) 생성 
  sort(x, x + 10, cmp2); // sort(int*, int*, bool(*)(int, int)) 생성 

  // 2. 비교 정책으로 함수객체를 사용할때 
  // 장점 : 정책 코드가 인라인 치환이 가능 - 속도가 빠르다. 
  // 단점 : 정책을 교체한 횟수만큼 sort() 함수 생성 
  Less    f1; 
  Greater f2; 
  sort(x, x + 10, f1); // sort(int*, int*, Less) 생성 
  sort(x, x + 10, f2); // sort(int*, int*, Greater) 생성 
}
```

## 참고
codenuri 강석민 강사 강의 내용기반으로 정리한 내용입니다.  
[코드누리](https://github.com/codenuri)

<br>

    This is personal diary for study documents.
    Please comment if I'm wrong or missing something else 😄. 

[Top](#){: .btn .btn--primary }{: .align-right}