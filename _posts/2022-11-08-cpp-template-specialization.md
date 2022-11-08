---
published: true
title:  "[Programming] C++ Template Specialization"
excerpt: "C++에 대해 알아보기, 템플릿 특수화"

categories:
  - Cpp
tags:
  - [C++, Cpp, Template, Specialization]

toc: true
toc_sticky: true
 
date: 2022-11-08
last_modified_at: 2022-11-08
---

## 1. 템플릿 특수화
- 템플릿을 특수화시켜서 특정 타압에서 동작하게 만들 수 있음

## 2. 코드로 알아보기
- [web_compiler](https://godbolt.org/) 에서 확인

```cpp
#include <iostream> 
using namespace std; 
  
// primary template 
template<typename T> class Stack 
{ 
public: 
  void push(T a) { cout << "T" << endl; } 
};

// 부분 특수화(partial specialization) 
template<typename T> class Stack<T*> 
{ 
public: 
  void push(T* a) { cout << "T*" << endl; } 
};

// 특수화(specialization) 
template<> class Stack<char*> 
{ 
public: 
  void push(char* a) { cout << "char*" << endl; } 
}; 
  
int main() 
{ 
  Stack<int>   s1;  s1.push(0); // T
  Stack<int*>  s2;  s2.push(0); // T*
  Stack<char*> s3;  s3.push(0); // char*
}
```

```cpp
#include <iostream> 
using namespace std; 

template<typename T, typename U> struct Test 
{ 
  void print() { cout << "T, U" << endl; } 
};

// T*, U* 
template<typename T, typename U> struct Test<T*, U*> 
{ 
  void print() { cout << "T*, U*" << endl; } 
}; 

// T, T 
// primary template 이 템플릿 인자가 2개라도 부분 특수화 할때 인자의 갯수는 바뀔수 있음
// 하지만 Test<T,T> 부분은 반드시 2개로 표기 해야 됨
template<typename T> struct Test<T, T> 
{ 
  void print() { cout << "T, T" << endl; } 
}; 

// short, T 
template<typename T> struct Test<short, T> 
{ 
  void print() { cout << "short, T" << endl; } 
}; 

// short, int 
template<> struct Test<short, int> 
{ 
  void print() { cout << "short, int" << endl; } 
}; 

// T,Test<U,V> 
template<typename T, typename U, typename V>  
struct Test<T, Test<U, V>> 
{ 
  void print() { cout << "T, Test<U, V>" << endl; } 
}; 
  
int main() 
{ 
  Test<int, double>   t1; t1.print(); // T, U 
  Test<int*, double*> t2; t2.print(); // T*, U* 
  Test<int, int>      t3; t3.print(); // T, T 
  Test<short, char>   t4; t4.print(); // short, T 
  Test<short, int>    t5; t5.print(); // short, int 
  Test<double, Test<char, int>>  t6; t6.print(); // T, Test<U,V> 
}
```

## 참고
codenuri 강석민 강사 강의 내용기반으로 정리한 내용입니다.  
[코드누리](https://github.com/codenuri)

<br>

    This is personal diary for study documents.
    Please comment if I'm wrong or missing something else 😄. 

[Top](#){: .btn .btn--primary }{: .align-right}