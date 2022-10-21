---
published: true
title:  "[Programming] C++ Static assert"
excerpt: "C++에 대해 알아보기, static assert"

categories:
  - Cpp
tags:
  - [C++, Cpp, StaticAssert]

toc: true
toc_sticky: true
 
date: 2022-10-21
last_modified_at: 2022-10-21
---

## 1. Static assert
- compile time 때 assert를 해주는 기능

## 2. 코드로 알아보기
- 함수 인자는 사용하기 전에 유효성 여부를 확인하는 것이 좋음

```cpp
#include <iostream> 
#include <cassert>  
using namespace std; 
  
static_assert(sizeof(void*) == 8, 
                    "error not 64bit"); 
//static_assert(sizeof(void*) == 8); // C++17 
  
void foo(int age) 
{ 
    // assert(age > 0); // 런타임 에러
    static_assert(age > 0);  // 컴파일타임 에러
} 

int main() 
{ 
    foo(-10); 
}
```

```cpp
#include <iostream> 
#include <type_traits> 
#include <cassert>  
using namespace std; 
  
//#pragma pack(1) 
struct PACKET 
{ 
    char cmd; 
    int  data; 
}; 
  
template<typename T> void memset(T* p) 
{ 
    // T 타입에 가상함수가 있는 경우 컴파일 에러
    static_assert(!std::is_polymorphic<T>::value, 
        "Error, T has virtual function"); 
  
    memset(p, 0, sizeof(T)); 
}

class A  
{ 
    virtual void foo() {} 
};

int main() 
{ 
    A a; 
    memset(&a); 
}
```

## 참고
codenuri 강석민 강사 강의 내용기반으로 정리한 내용입니다.  
[코드누리](https://github.com/codenuri)

<br>

    This is personal diary for study documents.
    Please comment if I'm wrong or missing something else 😄. 

[Top](#){: .btn .btn--primary }{: .align-right}