---
published: true
title:  "[Programming] C++ advance"
excerpt: "C++에 대해 알아보기, advance"

categories:
  - Cpp
tags:
  - [C++, Cpp, Advance]

toc: true
toc_sticky: true
 
date: 2022-12-22
last_modified_at: 2022-12-22
---

## 1. advance
- iterator 헤더에 정의
- iterator에 n개의 요소가 주어질 때 특정 index를 찾음

```cpp
#include <iostream>
#include <iterator>
#include <vector>
 
int main() 
{
  std::vector<int> v{ 3, 1, 4 };

  auto vi = v.begin();
  std::advance(vi, 2);
  std::cout << *vi << ' '; // 4

  vi = v.end();
  std::advance(vi, -2);
  std::cout << *vi << '\n'; // 1
}
```

## 참고
[cppreference_advance](https://en.cppreference.com/w/cpp/iterator/advance)  

<br>

    This is personal diary for study documents.
    Please comment if I'm wrong or missing something else 😄. 

[Top](#){: .btn .btn--primary }{: .align-right}