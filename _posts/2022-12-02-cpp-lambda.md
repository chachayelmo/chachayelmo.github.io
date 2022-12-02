---
published: true
title:  "[Programming] C++ 람다표현식"
excerpt: "C++에 대해 알아보기, lambda"

categories:
  - Cpp
tags:
  - [C++, Cpp, Lambda]

toc: true
toc_sticky: true
 
date: 2022-12-02
last_modified_at: 2022-12-02
---

## 1. 람다표현식
- 람다 기본 모양과 사용법
  - 람다 함수는 아래와 같이 대괄호[], 소괄호(), 중괄호{}, 소괄호() 이런 모양으로 생김
  - 여기서 생략이 가능한 건 소괄호들

```
[캡쳐] (매개변수) { 함수 동작 } (호출인자)
첫 번째 [] : 캡처
두 번째 () : 매개변수 선언 부분 (생략 가능 - 매개변수 필요 없을 때)
세 번째 {} : 함수 동작 부분
네 번째() : 함수 호출 시 인자 (생략 가능 - 호출 시에만 사용)
```

## 2. 코드로 알아보기
- [web_compiler](https://godbolt.org/) 에서 확인

### 2.1. 람다표현식

```cpp
#include <iostream> 
using namespace std; 
  
void foo(int(*f)(int, int)) 
{ 
  f(1, 2); 
} 
  
int add(int a, int b) { return a + b; } 
  
int main() 
{ 
  foo(add); 
    
  // [] : lambda introducer 
  // 람다표현식이 시작됨을 알리는 기호 
  // 람다표현식 : 익명의 함수를 만드는 표기법 
  foo([](int a, int b) { return a + b; }); 

  add(1, 2); 

  int n =    [](int a, int b) { return a + b; }(1, 2); 

  // 함수 이름 또는 주소가 필요한 곳에 함수 구현 자체를 표기하는 문법 
}
```

```cpp
#include <iostream> 
#include <algorithm> 
#include <vector> 
using namespace std; 
  
bool foo(int n) { return n % 3 == 0; } 
  
int main() 
{ 
  vector<int> v = { 1,2,3,4,5,6,7,8,9,10 }; 

  // 3 검색 
  auto p1 = find(v.begin(), v.end(), 3); 

  // 첫번째 나오는 3의 배수를 찾고 싶다. 
  auto p2 = find_if(v.begin(), v.end(), foo); 
    
  // 람다표현식의 주된 용도 
  // 알고리즘 함수에 조건자 함수로 전달하는 경우가 아주 많다. 
  auto p3 = find_if(v.begin(), v.end(),  
                    [](int n) { return n & 3 == 0; }); 
}
```

### 2.2. Closure (클로저)
- 람다표현식이 만든 임시객체

```cpp
#include <algorithm> 
#include <functional>  // less<>, greater<>등의 함수객체
using namespace std; 
  
inline bool cmp1(int a, int b) { return a < b; } 
  
int main() 
{ 
  int x[10] = { 1,3,5,7,9,2,4,6,8,10 }; 

  // 1. 함수를 사용한 정책 변경 
  sort(x, x + 10, cmp1); 

  // 2. 함수객체를 사용한 정책 변경 
  less<int> f; 
  sort(x, x + 10, f); 
  sort(x, x + 10, less<int>()); // 임시객체로 전달 

  // 3. C++11 부터는 람다 표현식을 사용가능
  sort(x, x + 10, [](int a, int b) { return a < b; }); 

  // 위 한줄을 보고 컴파일러는 아래 코드를 생성
  // 람다표현식 : 함수객체를 만드는 표현식 
  // 클로져(closure) : 람다표현식이 만든 임시객체 
  class ClosureType 
  { 
  public: 
      inline bool operator()(int a, int b) const 
      { 
          return a < b; 
      } 
  }; 

  sort(x, x + 10, ClosureType()); 
}
```

### 2.3. 람다표현식 활용

```cpp
#include <iostream> 
using namespace std; 
  
int main() 
{ 
  // auto  변수에 담아서 함수 처럼 사용가능 
  auto f = [](int a, int b) { return a < b; }; 
          // class ClosureType{ ... }; ClosureType(); 
  int n = f(1, 2); 

  // 다음 중 에러는? f2, lvalue
  auto        f    = [](int a, int b) { return a < b; }; 
  //auto&        f2    = [](int a, int b) { return a < b; }; 
  auto&&        f3    = [](int a, int b) { return a < b; }; 
  const auto& f4    = [](int a, int b) { return a < b; }; 
}
```

```cpp
#include <iostream> 
  
int main() 
{ 
  // 1. 정확한 모양. 후위반환표기법 
  auto f1 = [](int a, int b) -> int { return a + b; }; 

  // 2. 리턴문장이 하나라면 생략 가능. 
  auto f2 = [](int a, int b) { return a + b; }; 

  // 3. 리턴 문장이 2개이상인 경우 - 같은 타입이면 생략가능 
  auto f3 = [](int a, int b) { if (a == 1) return a; else return b; }; 


  // 4. 다른 타입일경우 - 반드시 리턴 타입을 표시해야 됨
  auto f4 = [](int a, double b) -> double { if (a == 1) return a; else return b; }; 
}
```

### 2.4. 인라인 치환

```cpp
#include <iostream> 
#include <functional> 
using namespace std;   
  
int main() 
{ 
  // auto일 경우에만 람다표현식이 inline 치환 가능 
  auto f1 = [](int a, int b) { return a + b; }; 
  int(*f2)(int, int) = [](int a, int b) { return a + b; }; 
  function<int(int,int)> f3 = [](int a, int b) { return a + b; }; 

  // 인라인 치환성
  f1(1, 2);   // 인라인 치환 됨. 절대 f1을 다른 함수로 바꿀 수 없음 
              // 치환되는 이유 
              // f1이 가진 주소가 아닌 f1의 타입 기반 호출(컴파일시간에) 
              // f1.operator()(1,2)로 실행 

  f2(1, 2);   // 치환 안됨. 다른 함수로 바꿀 수 있음 
              // f2가 가진 주소 기반 호출 
              // 실행시간에 메모리에서 값 꺼내기 

  f3(1, 2); //  치환 안됨. 다른 함수로 바꿀 수 있음 
}
```

```cpp
#include <iostream> 
//void foo( int(*f)(int, int) ) {} // 1. 함수 포인터, 치환 안됨 
//void foo( auto f ) {} // error, auto는 함수인자가 될 수 없음
  
template<typename T> 
void foo( T&& f ) {} // 템플릿. 인라인 치환 가능 

int main() 
{ 
  foo( [](int a, int b ) { return a + b; } ); 
  foo( [](int a, int b ) { return a - b; } ); 
}
```

### 2.5. 캡쳐 활용법

```cpp
#include <iostream> 
using namespace std; 
  
int g = 0; 
  
int main() 
{ 
  int v1 = 10; 
  int v2 = 10; 

  auto f1 = [](int a) { return a + g; };// 전역변수 사용, ok 

  //auto f2 = [](int a) { return a + v1 + v2; };// 지역변수 사용, error  

  // 지역변수를 사용하려면 캡쳐
  auto f2 = [v1, v2](int a) { return a + v1 + v2; }; 

  // error. 캡쳐된 변수는 수정할 수 없음
  //auto f3 = [v1, v2](int a) { v1 = 20; return a + v1 + v2; }; 

  // mutable 람다 : () 연산자 함수를 비상수 함수로 바꿈
  auto f3 = [v1, v2](int a) mutable { v1 = 20; return a + v1 + v2; }; 

  f3(0); 

  cout << sizeof(f3) << endl; // 8 
  cout << v1 << endl;         // 10 

  // 원리
  class ClosureType 
  { 
    int v1; 
    int v2; 
  public: 
    ClosureType(int vx, int vy) : v1(vx), v2(vy) {} // 복사 
    //inline int operator()(int a) const // 멤버 상수 함수기 때문에 멤버 변수를 바꿀 수 없음 
    inline int operator()(int a) // mutable
    { 
      v1 = 20; 
      return a + v1 + v2; 
    } 
  }; 
  auto f4 = ClosureType(v1, v2);
}
```

```cpp
#include <iostream> 
using namespace std; 
  
int main() 
{ 
  int v1 = 10; 
  int v2 = 10; 

  // capture by value 
  auto f1 = [v1, v2](int a) { return a + v1; };

  // capture variable by reference 
  auto f2 = [&v1, &v2](int a) { v1 = 100; v2 = 200; return a + v1; }; 
    
  class ClosureType 
  { 
    int &v1; 
    int &v2; 
  public: 
    ClosureType(int& vx, int& vy) : v1(vx), v2(vy) {} 
    inline int operator()(int a) const 
    { 
      v1 = 20; 
      return a + v1 + v2; 
    } 
  }; 
  auto f3 = ClosureType(v1, v2); 

  f2(0); 
} 
```

```cpp
int main() 
{ 
  int v1, v2; 
  // = : 모든 지역변수를 "값"으로 캡쳐 
  auto f1 = [=](int a) { return a + v1 + v2; }; 
  // & : 모든 지역변수를 "참조"로 캡쳐 
  auto f2 = [&](int a) { return a + v1 + v2; }; 
  auto f3 = [v1](int a) { return a + v1 + v2; }; // v1만 캡쳐(값) 
  auto f4 = [&v1](int a) { return a + v1 + v2; };// v1만 캡쳐(참조) 
    
  // 모든 지역변수는 값, v1만 참조 
  auto f5 = [=, &v1](int a) { return a + v1 + v2; }; 

  // 모든 지역변수는 참조, v1만 값
  auto f6 = [&, v1](int a) { return a + v1 + v2; }; 
}
```

```cpp
#include <iostream> 
using namespace std; 

class Test 
{ 
  int data; 
public: 
  void foo()   
  { 
    // 멤버 변수 캡쳐, 멤버변수는 this를 가짐 

    // auto f = [](int a) { return data + a; }; // error data
    //auto f = [data](int a) { return data + a; }; // error data는 지역변수가 아님

    // ok, 멤버 데이터에 접근 하려면 this 캡쳐 
    auto f = [this](int a) { data = 10; return data + a; }; 
    auto f2 = [=](int a) { return data + a; }; 

    // *this 캡쳐 : C++20 
    //auto f = [*this](int a) { data = 10; return data + a; }; 
  } 
}; 
  
int main() 
{ 
  Test t; 
  t.foo(); // foo(&t) 
}
```

## 참고
codenuri 강석민 강사 강의 내용기반으로 정리한 내용입니다.  
[코드누리](https://github.com/codenuri)  
[cppreference](https://en.cppreference.com/w/cpp/language/lambda)

<br>

    This is personal diary for study documents.
    Please comment if I'm wrong or missing something else 😄. 

[Top](#){: .btn .btn--primary }{: .align-right}