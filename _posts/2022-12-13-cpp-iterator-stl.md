---
published: true
title:  "[C++] STL"
excerpt: "C++에 대해 알아보기, STL"

categories:
  - Cpp
tags:
  - [C++, Cpp, Stl]

toc: true
toc_sticky: true
author: chachayelmo
sitemap:
  changefreq : daily
  priority : 1.0
comment: true
date: 2022-12-13
last_modified_at: 2022-12-13
---

## 1. STL
- ADL (Argument Dependant Lookup)

```cpp
#include <vector> 
#include <algorithm> 
  
namespace Video 
{ 
  void foo() {} 
  struct Card {}; 
  void init(Card c) {} 
} 
  
int main() 
{ 
  Video::foo();    //Video 반드시 필요 
  Video::Card c;   //Video 반드시 필요 
  init(c);         //Video 없어도 가능 (ADL)

  std::vector<int> v(10, 3); 
  find(v.begin(), v.end(), 3); // ok 
}
```

## 2. STL 핵심
- 반복자 카테고리 5가지:
  1. 입력 반복자    : 입력, ++
  2. 출력 반복자    : 출력, ++
  3. 전진형 반복자  : 입력, ++, 멀티패스, 싱글리스트 반복자
  4. 양방형 반복자  : 입력, ++, 멀티패스, 더블리스트 반복자
  5. 임의접근 반복자: 입력, ++, --, +, -, [], 멀티패스, 연속된 메모리와 유사 (vector)
- 알고리즘은 컨테이너를 알지 못함
- 알고리즘과 동일 이름의 멤버함수가 있다면 반드시 멤버 함수를 사용해야 됨
  - 예) std::list 는 remove 알고리즘이 아닌 remove 멤버함수를 사용

```cpp
#include <vector> 
#include <list> 
#include <algorithm> 
#include <iostream> 

int main() 
{
  // vector
  std::vector<int> v = { 1,2,3,1,2,3 }; 

  // remove는 컨테이너의 크기를 줄이지 않음
  // 단지, 앞으로 당겨놓는 index를 당김
  // 따라서 메모리 해지를 할 수 없음 
  auto p = remove(v.begin(), v.end(), 3); 

  for (auto&& n : v) std::cout << n << ", "; 

  // vector의 멤버함수 erase 로만 메모리를 제어할 수 있음 
  v.erase(p, v.end()); 
  std::cout << std::endl; 
  for (auto&& n : v) std::cout << n << ", "; 

  // list
  std::list<int> v1 = { 1,2,3,1,2,3 }; 

  // list의 경우 절대 아래처럼 하면 안됨 
  // auto p1 = remove(v1.begin(), v1.end(), 3); 
  // for (auto&& n : v1) std::cout << n << ", "; 
  // v1.erase(p1, v.end()); 
    
  // list는 멤버 함수 remove를 사용
  // 3을 만나면 즉시 제거하고 node만 조작 
  v1.remove(3); 
    
  std::cout << std::endl; 
  for (auto&& n : v) std::cout << n << ", "; 
}
```

## 3. 단위전략 디자인, Policy Base Design

```cpp
#include <set> 
#include <functional> 
#include <algorithm> 
#include <iostream> 
using namespace std; 
  
// 단위 전략 디자인 Policy Base Design 
/* 
template<typename T,  
         typename Pred = less<T>,       // 요소 비교 
         typename Ax = allocator<T>>    // 메모리 할당기         
class set 
{ 
  Pred pred; // 요소 비교를 위한 정책 클래스 
public: 
  pair<...> insert(const T& newElem) 
  { 
    //if (rootElem < newElem) 
    if (pred(rootElem, newElem)) 
        add 오른쪽; 
    //elif(newElem < rootElem) 
    elif(pred(newElem, rootElem)) 
        add 왼쪽; 
    else 
        이미존재한다. return 실패 
            
  } 
}; 
*/ 
  
// set에 정책 클래스 조건은 "이항 함수객체"
struct Greater 
{ 
  bool operator()(int a, int b) const { return a > b; } 
}; 
  
int main() 
{ 
  set<int, Greater> s;        // RB Tree 
  s.insert(10); 
  s.insert(20); 
  s.insert(15); 

  for (auto&& n : s) cout << n << ", "; 
}
```

## 4. Equality ve Equivalency

```cpp
#include <set> 
#include <functional> 
#include <algorithm> 
#include <iostream> 

struct Greater 
{
  bool operator()(int a, int b) const { return (a / 10) > (b / 10); } 
}; 
  
int main() 
{
  std::set<int, Greater> s;        // RB Tree 
  s.insert(15); 
  s.insert(20); 
  s.insert(35); 
  s.insert(45);

  // 선형 검색 - Tree의 특징을 활용 못해서 느림 
  // "==" 로 "동등"을 판단 , Equality 
  auto p1 = std::find(s.begin(), s.end(), 32); 

  if (p1 == s.end()) 
      std::cout << "검색 실패" << std::endl; 

  // 이진 검색 - Tree의 특징을 활용해서 빠름 
  // "조건자 함수객체"로 동등을 판단 , Equivalency 
  auto p2 = s.find(32); 

  if (p2 == s.end()) 
      std::cout << "검색 실패" << std::endl; 
  else 
      std::cout << "검색 성공: " << *p2 << std::endl; // 35 
}
```

## 참고
codenuri 강석민 강사 강의 내용기반으로 정리한 내용입니다.  
[코드누리](https://github.com/codenuri)  

<br>

    This is personal diary for study documents.
    Please comment if I'm wrong or missing something else 😄. 

[Top](#){: .btn .btn--primary }{: .align-right}