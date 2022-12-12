---
published: true
title:  "[Programming] C++ 반복자 카테고리"
excerpt: "C++에 대해 알아보기, iterator categorty"

categories:
  - Cpp
tags:
  - [C++, Cpp, Iterator]

toc: true
toc_sticky: true
 
date: 2022-12-12
last_modified_at: 2022-12-12
---

## 1. 반복자 카테고리
- 입력 : 컨테이너에서 꺼내 오는 것
  - *p = ?
- 출력 : 컨테이너에 넣는 것, 
  - ? = *p

- 덧셈을 할 수 있는 조건은 연속된 메모리를 가지는 컨테이너여야 가능
- 즉, linked-list는 연속된 메모리가 아니기 때문에 덧셈이 불가능

- 반복자 카테고리 5가지:
  1. 입력 반복자    : 입력, ++
  2. 출력 반복자    : 출력, ++
  3. 전진형 반복자  : 입력, ++, 멀티패스, 싱글리스트 반복자
  4. 양방형 반복자  : 입력, ++, 멀티패스, 더블리스트 반복자
  5. 임의접근 반복자: 입력, ++, --, +, -, [], 멀티패스, 연속된 메모리와 유사 (vector)

- 멀티패스란? 구간을 여러개의 반복자가 접근할 수 있는 것
  - STL 컨테이너는 모두 멀티패스
  - 멀티패스를 지원하지 않는 건? 입력 버퍼
  - sort는 멀티패스를 지원해야 하며, find는 멀티패스를 지원하지 않아도 됨

```cpp
#include <iostream> 
#include <vector> 
#include <list> 
#include <forward_list> 
#include <algorithm> 
using namespace std; 

int main() 
{ 
  std::vector<int> v = { 1,2,3 }; 
  auto p1 = v.begin(); 
  ++p1;            //  ok
  --p1;            //  ok
  p1 = p1 + 3;     //  ok

  std::list<int> s = { 1,2,3 }; 
  auto p2 = s.begin(); 
  ++p2;              // ok 
  --p2;              // ok 
  // p2 = p2 + 3;    //  error

  std::forward_list<int> fs = { 1,2,3 }; // 싱글리스트, uni-direction 
  auto p3 = fs.begin(); 
  ++p3;              // ok 
  //--p3;            // error 
  // p3 = p3 + 3;    //  error
} 
```

```cpp
#include <iostream> 
#include <vector> 
#include <list> 
#include <forward_list> 
#include <algorithm> 
using namespace std; 

int main() 
{ 
  std::list<int> s = {5, 7, 4, 2, 8, 6, 1, 9, 0, 3};

  auto p = std::find(s.begin(), s.end(), 5); 
              // 최소 요구조건은? 입력 반복자, 출력 x, 후진 x, 멀티패스 x 

  std::reverse(s.begin(), s.end());  
              // => 양방형 반복자, 후진이 필요

  // error
  std::sort(s.begin(), s.end()); // 퀵 소트
              // => 임의접근 반복자 , 반 자르기 위해 뺄셈이 필요 
  
}
```

```cpp
#include <iostream> 
#include <vector> 
#include <list> 
#include <forward_list> 
#include <algorithm> 
using namespace std; 

int main() 
{ 
  std::list<int> s = { 1,2,3,4,5,6,7,8,9,10 }; 

  // sort : 임의접근 반복자를 요구
  // list 반복자 : 양방향 반복자
  sort(s.begin(), s.end()); // error 

  s.sort(); // list는 STL 알고리즘에 보낼 수 없기 때문에 멤버 함수 sort가 존재 
            // quick이 아닌 다른 알고리즘 사용 

  // 그렇다면 vector는?
  vector<int> v = { 1,2,3,4,5 }; 
  //v.sort(); // 없음. 임의접근 반복자이므로 STL 알고리즘으로 커버 가능 

  std::forward_list<int> s2 = { 1,2,3 }; 
  // reverse : 양방향 반복자를 요구 
  // forward_list : 전진형 반복자
  std::reverse(s2.begin(), s2.end()); // error 
} 
 
// vector는 연속된 메모리 
// 00000000 
// list 는 떨어진 메모리 
// 0-0-0-0-0-0 
// deque는 연속된 메모리를 가리키다가 떨어진 메모리 후 다시 연속된 메모리 
// 따라서 deque는 덧셈을 지원 
// 00000-00000-00000

```

## 참고
codenuri 강석민 강사 강의 내용기반으로 정리한 내용입니다.  
[코드누리](https://github.com/codenuri)  

<br>

    This is personal diary for study documents.
    Please comment if I'm wrong or missing something else 😄. 

[Top](#){: .btn .btn--primary }{: .align-right}