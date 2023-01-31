---
published: true
title:  "[Design Pattern] Policy Base Design in C++"
excerpt: "다양한 디자인 패턴에 대해 알아보기"

categories:
  - Algorithm
tags:
  - [DesignPattern, PolicyBase, Pattern]

toc: true
toc_sticky: true
 
date: 2023-01-31
last_modified_at: 2023-01-31
---

## 1. Policy Base Design

- 템플릿 인자로 Policy를 담은 클래스를 전달하는 기술
- 성능 저하 없이 Policy만 교체 가능
- Policy 클래스에는 지켜야 하는 규칙이 있게 됨
  - 규칙을 표현하는 코딩 방식은 없음
  - 동기화 정책 클래스에는 lock/unlock 이 필요
  - 템플릿 기반 라이브러리, 특히 STL에서 널리 사용되는 디자인 기법

## 2. 코드로 알아보기

```cpp
class nolock;

template<typename T,  
    typename ThreadModel = nolock> class List 
{ 
  ThreadModel tm; 
public: 
  void push_front(const T& a) 
  { 
    tm.lock(); 
    // ... 
    tm.unlock(); 
  } 
}; 
  
// 동기화 policy를 담은 클래스 
class nolock 
{ 
public: 
  inline void lock() {} 
  inline void unlock() {} 
}; 
  
class posix_lock 
{ 
public: 
  inline void lock() {} 
  inline void unlock() {} 
}; 
  
//------------------------------- 
// 2번째 인자로 정책을 보냄
List<int, nolock> s; // 전역변수.   멀티스레드에 안전하지 않다. 
  
int main() 
{ 
    s.push_front(10); 
}
```

- STL의 실제 vector

```cpp
template<
    class T,
    class Allocator = std::allocator<T> // Allocator 를 교체할 수 있음
> class vector;
```

```cpp
#include <iostream> 
#include <vector> 
using namespace std; 

/* 
template<typename T, 
            typename Ax = allocator<T>> class vector 
{ 
  T* buff; 
  int size; 
  Ax ax; //메모리 allocator
public: 
  void push_back(const T& a) 
  { 
    // 메모리 재할당
    buff = ax.allocate(10);  // 메모리 할당 
    ax.deallocate(buff, 10); // 메모리 해지 
  } 
}; 
*/ 
  
int main() 
{ 
    vector<int> v; 
}
```

- C++ 표준 메모리 allocator

```cpp
#include <iostream> 
#include <vector> 
using namespace std; 
  
struct Point 
{ 
  int x, y; 
  Point(int a = 0, int b = 0) { std::cout << __PRETTY_FUNCTION__ << std::endl; } 
    
  ~Point() { std::cout << __PRETTY_FUNCTION__ << std::endl; } 
}; 
int main() 
{ 
  // C++ 표준 메모리 할당기 
  std::allocator<Point> ax; 

  // Point 메모리만 2개 할당 
  Point* p = ax.allocate(2); 

  // 생성자 호출 
  ax.construct(p, 1, 2);        // new(p) Point(1,2) 
  ax.construct(p+1, 1, 2); 

  cout << "==============" << endl;

  // 소멸자 호출 
  ax.destroy(p); 
  ax.destroy(p+1); 

  // 메모리 해지 
  ax.deallocate(p, 2); 
}
```

## 참고
codenuri 강석민 강사 강의 내용기반으로 정리한 내용입니다.  
[코드누리](https://github.com/codenuri)  
[cppreference](https://en.cppreference.com/w/cpp/container/vector)  

<br>

    This is personal diary for study documents.
    Please comment if I'm wrong or missing something else 😄. 

[Top](#){: .btn .btn--primary }{: .align-right}