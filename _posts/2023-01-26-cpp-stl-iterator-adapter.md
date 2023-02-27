---
published: true
title:  "[C++] STL 반복자 어댑터(iterator adapter)"
excerpt: "iterator adapter에 대해 알아보기"

categories:
  - Cpp
tags:
  - [반복자어댑터, C++, Cpp, Iterator, Adapter]

toc: true
toc_sticky: true
author: chachayelmo
sitemap:
  changefreq : daily
  priority : 1.0
comment: true
date: 2023-01-26
last_modified_at: 2023-01-26
---

## 1. reverse_iterator

- 역방향으로 동작하는 반복자
- reverse_iterator를 사용하면 모든 알고리즘을 역방향으로도 동작하게 할 수 있음

```cpp
#include <iostream>
#include <list>
#include <algorithm>
using namespace std;

int main()
{
    list<int> s = {1,2,3,4,5,6,7,8,9,10};
    auto p1 = begin(s);
    auto p2 = end(s);

    reverse_iterator< list<int>::iterator > p3(p2);
    reverse_iterator< list<int>::iterator > p4(p1);
    
    cout << *p3 << endl; // 10
    ++p3; // --p2
    cout << *p3 << endl; // 9
    ++p3;
    cout << *p3 << endl; // 8
    
    // auto ret = find(p1, p2, 3);
    // auto ret = find(p2, p1, 3); // 에러, ++ 연산을 하기 때문에
    auto ret = find(p3, p4, 3);
    cout << *ret << endl;
}
```

### 1.1. reverse_iterator 만들어보기

```cpp
#include <iostream>
#include <list>
#include <algorithm>
using namespace std;

template<typename ITER> class ereverse_iterator
{
    ITER iter; // 내부적으로 iterator를 가짐
public:
    ereverse_iterator(ITER i) { iter = i; --iter; } // 항상 이전 값을 가지기 때문에 -- 연산
    ereverse_iterator& operator++() // ++ 동작 수행
    {
        --iter;
        return *this;
    }
    typename ITER::value_type operator*() // * 동작, int type임
    {
        return *iter;
    }
};

int main()
{
    list<int> s = {1,2,3,4,5,6,7,8,9,10};
    auto p1 = end(s);

    ereverse_iterator< list<int>::iterator > p2(p1);
    
    ++p2; // --p2
    cout << *p2 << endl; // 9
}
```

### 1.2. 만드는 방법

- 기존 반복자를 가지고 생성
    - reverse_iterator<> 클래스 사용
    - make_reverse_iterator<> 함수 사용
- 컨테이너에서 직접 꺼내기
    - rbegin(), rend() 멤버 함수
    - rbegin(), rend() 일반 함수

```cpp
#include <iostream>
#include <list>
#include <algorithm>
using namespace std;

int main()
{
    list<int> s = {1,2,3,4,5,6,7,8,9,10};
    auto p1 = begin(s);
    auto p2 = end(s);

    // 기존 반복자를 가지고 생성
    reverse_iterator< list<int>::iterator > r1(p2);
    auto r2 = make_reverse_iterator(p2);    
    // 컨테이너에서 직접 꺼내기
    auto r3 = s.rbegin();
    auto r4 = rbegin(s);

    cout << *r1 << endl; // 10
    cout << *r2 << endl; // 10
    cout << *r3 << endl; // 10
    cout << *r4 << endl; // 10
}
```

```cpp
#include <iostream>
#include <list>
#include <algorithm>
using namespace std;

int main()
{
    list<int> s = {1,2,3,4,5,6,7,8,9,10};

    // 컨테이너에서 꺼내는 iterator 4가지 종류
    list<int>::iterator p1 = begin(s);
    list<int>::reverse_iterator p2 = rbegin(s);
    list<int>::const_iterator p3 = cbegin(s);
    // *p3 = 10; // error
    list<int>::const_reverse_iterator p4 = crbegin(s);
}
```

## 2. move iterator

- move_iterator<> 클래스 템플릿 사용
- make_move_iterator<> 함수 사용
- C++11 부터 지원

```cpp
#include <iostream>
#include <string>
#include <vector>
using namespace std;

class People
{
    std::string name;
public:
    People(string s) : name(s) {}
    People(const People& p ) : name(p.name)
    {
        cout << "copy" << endl;
    }
    People(People&& p ) noexcept : name(std::move(p.name))
    {
        cout << "move" << endl;
    }
    People& operator=(const People& p )
    {
        name = p.name;
        cout << "copy=" << endl;
        return *this;
    }
    People& operator=(People&& p )
    {
        name = std::move(p.name);
        cout << "move=" << endl;
        return *this;
    }
};

int main()
{
    vector<People> v;
    v.push_back(People("A"));
    v.push_back(People("B"));
    v.push_back(People("C"));
    v.push_back(People("D"));

    cout << "-----------------" << endl;

    // vector<People> v2(begin(v), end(v));
    vector<People> v2(make_move_iterator(begin(v)),
                        make_move_iterator(end(v)));

    auto p1 = begin(v);
    People peo1 = *p1; // 복사 생성자

    // 방법 1.
    move_iterator< vector<People>::iterator> p2(p1);
    People peo2 = *p2; // 이동 생성자, std::move(*p1) 과 같은 역할

    // 방법 2.
    auto p3 = make_move_iterator(begin(v));
}
```

## 참고
codenuri 강석민 강사 강의 내용기반으로 정리한 내용입니다.  
[코드누리](https://github.com/codenuri)  

<br>

    This is personal diary for study documents.
    Please comment if I'm wrong or missing something else 😄. 

[Top](#){: .btn .btn--primary }{: .align-right}