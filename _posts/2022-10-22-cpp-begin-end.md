---
published: true
title:  "[C++] 일반함수 Begin/End"
excerpt: "C++에 대해 알아보기, Begin/End"

categories:
  - Cpp
tags:
  - [C++, Cpp, Begin, End]

toc: true
toc_sticky: true
author: chachayelmo
sitemap:
  changefreq : daily
  priority : 1.0
date: 2022-10-22
last_modified_at: 2022-10-22
---

## 1. Begin, End
- 일반함수 begin/end

## 2. 코드로 알아보기
- [web_compiler](https://godbolt.org/) 에서 확인
- 반복자 : 컨테이너의 요소를 가리키는 포인터 같이 동작하는 객체 

```cpp
#include <iostream> 
#include <vector> 
#include <list> 
  
int main() 
{ 
//    std::vector<int> v = { 1,2,3,4,5 }; 
//    std::list<int> v = { 1,2,3,4,5 }; 
    int v[5] = { 1,2,3,4,5 }; 
  
// 멤버함수에 begin, end가 없을 수 있음
//    auto p1 = v.begin(); 
//    auto p2 = v.end(); 
  
    // C++ 98 스타일 : 멤버 함수 begin 
    // C++ 11 스타일 : 일반 함수 begin/end 사용  
    auto p1 = std::begin(v); 
    auto p2 = std::end(v); 
    std::cout << *p1 << " " << *(p2-1) << std::endl;
}
```

## 참고
codenuri 강석민 강사 강의 내용기반으로 정리한 내용입니다.  
[코드누리](https://github.com/codenuri)

<br>

    This is personal diary for study documents.
    Please comment if I'm wrong or missing something else 😄. 

[Top](#){: .btn .btn--primary }{: .align-right}