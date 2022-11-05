---
published: true
title:  "[Programming] C++ Suffix return"
excerpt: "C++에 대해 알아보기, Suffix return"

categories:
  - Cpp
tags:
  - [C++, Cpp, Suffix, Return]

toc: true
toc_sticky: true
 
date: 2022-11-05
last_modified_at: 2022-11-05
---

## 1. Suffix return
- 후위 반환 타입 C++14 부터 auto로만 가능
- return이 참조를 유지하려면 decltype(auto)로 받아야됨, 그냥 auto는 &를 떼고 받음
- 람다표현식과 템플릿에서 주로 사용

## 2. 코드로 알아보기
- [web_compiler](https://godbolt.org/) 에서 확인

```cpp
#include <iostream> 
using namespace std; 

auto square(int n) -> int 
{ 
  return n * n; 
} 
  
int main() 
{ 
  square(3); 
}
```

```cpp
#include <iostream> 
using namespace std; 

// return 문장의 표현이 참조 타입일때 참조를 유지하려면
template<typename T1, typename T2> 
decltype(auto) mul(T1 a, T2 b)
//auto mul(T1 a, T2 b) -> C++14 에서는 auto만 표기해도 됨
{ 
  return a * b; 
} 
  
int main() 
{ 
  cout << mul(3, 4.2) << endl; 
}
```

## 참고
codenuri 강석민 강사 강의 내용기반으로 정리한 내용입니다.  
[코드누리](https://github.com/codenuri)

<br>

    This is personal diary for study documents.
    Please comment if I'm wrong or missing something else 😄. 

[Top](#){: .btn .btn--primary }{: .align-right}