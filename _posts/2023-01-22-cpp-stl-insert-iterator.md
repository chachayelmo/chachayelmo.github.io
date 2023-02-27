---
published: true
title:  "[C++] STL 삽입 반복자(insert iterator)"
excerpt: "insert iterator에 대해 알아보기"

categories:
  - Stl
tags:
  - [삽입반복자, C++, Cpp, Stl, Iterator, Insert]

toc: true
toc_sticky: true
author: chachayelmo
sitemap:
  changefreq : daily
  priority : 1.0
comment: true
date: 2023-01-22
last_modified_at: 2023-01-22
---

## 1. Insert iterator

### 1.1. 컨테이너 요소를 추가하는 방법

- 멤버 함수 ( push_back / emplace_back / insert 등 ) 사용
- 삽입 반복자 (insert iterator)

### 1.2. 삽입 반복자

```cpp
#include <iterator>
```
- 컨테이너에 요소를 추가할 때 사용하는 반복자
- 3가지 종류 - 전방 삽입 / 후방 삽입 / 임의 삽입 제공

### 1.3. 모양

```cpp
// back_insert_iterator<컨테이너 클래스 이름> p(컨테이너객체);
back_insert_iterator< list<int> > p(s);
```

- copy와 같은 STL 알고리즘을 사용해서 컨테이너에 요소를 추가할 수 있음

### 1.4. 코드로 알아보기

```cpp
#include <iostream>
#include <list>
#include <iterator>
using namespace std;

int main()
{
    list<int> s = { 1,2,3,4,5 };

    s.push_back(10);

    back_insert_iterator< list<int> > p(s);
    *p = 20; // s.push_back(20);

    int x[5] = { 100,200,300,400,500};
    copy(x, x+5, p);

    for (auto& n : s)
        cout << n << endl;
}
```

## 2. back_inserter 함수

- back_inserter_iterator를 생성해 주는 helper 함수
- 대부분 back+inserter_iterator를 직접 사용하기 보다는 back_inserter()를 사용
- 클래스 템플릿은 반드시 템플릿 인자를 지정해야 하지만 함수 템플릿은 템플릿 인자를 생략할 수 있다 - C++17 이전까지

```cpp
#include <iostream>
#include <list>
#include <iterator>
using namespace std;

/* 표준에 이미 정의
template<typename T>
back_insert_iterator<T> back_inserter(T& c){
    return back_insert_iterator<T>(c);
}
*/

int main()
{
    list<int> s = { 1,2,3,4,5 };

    int x[5] = { 100,200,300,400,500};

    // copy(x, x+5, s.begin()); // 덮어씀

    // back_insert_iterator< list<int> > p(s); // 너무 길다
    copy(x, x+5, back_inserter(s));

    for (auto& n : s)
        cout << n << endl;
}
```

## 3. Insert iterator 종류

### 3.1. 종류

1. back_insert_iterator (후방 삽입 반복자)
    1. 생성 함수 : back_inserter
    2. 내부 구현 : push_back
2. front_insert_iterator (전방 삽입 반복자)
    1. 생성 함수 : front_inserter
    2. 내부 구현 : push_front
3. insert_iterator (임의 삽입 반복자)
    1. 생성 함수 : inserter
    2. 내부 구현 : insert

### 3.2. 주의 사항

- vector는 앞에 삽입할 수 없음 (push_front 멤버함수가 없다)
- 임의 삽입의 경우 생성자 인자가 2개임

```cpp
insert_iterator< list<int> > p(s, s.begin());
```

### 3.3. 전방 삽입과 임의 삽입

- front_insert_iterator를 사용해서 컨테이너에 추가하면 요소의 순서가 반대로 삽입
- insert_iterator를 사용해서 컨테이너의 앞에 삽입하면 요소의 순서대로 삽입

```cpp
#include <iostream>
#include <list>
#include <iterator>
using namespace std;

int main()
{
    list<int> s = { 0,0,0,0,0 };

    int x[5] = { 1,2,3,4,5};

    // front_insert_iterator< list<int> > p(s);
    // insert_iterator< list<int> > p(s, ++s.begin());

    // copy(x, x+5, p);

    // copy(x, x+5, back_inserter(s));      // 0 0 0 0  1 2 3 4 5
    // copy(x, x+5, front_inserter(s));     // 5 4 3 2 1 0 0 0 0 0
    copy(x, x+5, inserter(s, ++s.begin())); // 0 1 2 3 4 5 0 0 0 0

    for (auto& n : s)
        cout << n << endl;
}
```

### 3.4. back_insert_iterator 만들어보기

- 모든 반복자는 *로 요소 접근이 가능하고 ++ 연산자로 이동 가능해야 함
- 연산 : ++, *, =

```cpp
#include <iostream>
#include <list>
#include <iterator>
using namespace std;

template<typename C>
class eback_insert_iterator
{
    C* container;
public:
    // Member Type
    using iterator_category = output_iterator_tag;
    using value_type = void;
    using pointer = void;
    using reference = void;
    using difference_type = void;

    using container_type = C;

    explicit eback_insert_iterator(C& c)
        : container(std::addressof(c)) {}

    eback_insert_iterator& operator*() { return *this; }
    eback_insert_iterator& operator++() { return *this; }
    eback_insert_iterator& operator++(int) { return *this; }
    eback_insert_iterator& operator=(const typename C::value_type& a)
    {
        container->push_back(a);
        return *this;
    }
    eback_insert_iterator& operator=(const typename C::value_type&& a)
    {
        container->push_back(move(a));
        return *this;
    }

};

int main()
{
    list<int> s = { 1, 2 };

    eback_insert_iterator< list<int> > p(s);

    *p = 30; // ( p.operator*() ).operator=(30)

    int x[2] = { 3, 4 };
    copy(x, x + 2, p);

    for (auto& n : s)
        cout << n << endl;
}
```

## 참고
codenuri 강석민 강사 강의 내용기반으로 정리한 내용입니다.  
[코드누리](https://github.com/codenuri)  

<br>

    This is personal diary for study documents.
    Please comment if I'm wrong or missing something else 😄. 

[Top](#){: .btn .btn--primary }{: .align-right}