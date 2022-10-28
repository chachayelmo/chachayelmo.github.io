---
published: true
title:  "[Programming] C++ raw str"
excerpt: "C++에 대해 알아보기, Raw str"

categories:
  - Cpp
tags:
  - [C++, Cpp, RawStr]

toc: true
toc_sticky: true
 
date: 2022-10-27
last_modified_at: 2022-10-27
---

## 1. Raw str
- "\\" 정규표현식이나 디렉토리 표현에 널리 사용
- raw string은 "\" 를 특수 문자로 취급하지 말라는 의미
- "( : 문자열 시작 
- )" : 문자열 종료

## 2. 코드로 알아보기
- [web_compiler](https://godbolt.org/) 에서 확인

```cpp
#include <iostream> 
using namespace std; 

enum class COLOR { red = 1, blue = 2 }; 
  
int main() 
{ 
    COLOR c = COLOR::red; // ok 
    // int n1 = COLOR::red; // error 
}
```

## 참고
codenuri 강석민 강사 강의 내용기반으로 정리한 내용입니다.  
[코드누리](https://github.com/codenuri)

<br>

    This is personal diary for study documents.
    Please comment if I'm wrong or missing something else 😄. 

[Top](#){: .btn .btn--primary }{: .align-right}