---
published: true
title:  "[Programming] STL algorithm"
excerpt: "algorithm 에 대해 알아보기"

categories:
  - Cpp
tags:
  - [C++, Cpp, STL, algorithm]

toc: true
toc_sticky: true
 
date: 2023-01-30
last_modified_at: 2023-02-01
---

## 1. Container

- Sequence Container
    - 요소가 삽입된 순서대로 놓여있는 컨테이너
    - list, vector, deque, forward_list, array
- Associative Container
    - 삽입된 순서와 상관없이 규칙을 가지고 요소를 보관
    - tree : set, multi-set, map, multi-map
    - hash : unordered_set, unordered_multiset, unordered_map, unordered_multimap
- Container Adapter
    - stack, queue, priority_queue

## 2. 특징

- 값을 보관
- member type이 있음
- 반복자를 가지고 있음
- 제거와 리턴을 동시에 하지 않음
    - back, front, top 함수는 요소를 리턴만 함
    - pop_xxx 함수들은 제거만 하고, 리턴하지 않음
    - 임시 객체를 막을 수 있고, 예외 안정성을 보장
- STL 자체에는 예외가 거의 없음

```cpp
#include <iostream>
#include <list>
#include <vector>
using namespace std;

struct Point
{
    int x;
    int y;
};

int main()
{
    list<Point> s;
    Point pt;
    s.push_back(pt);

    list<Point>::value_type n; // Point
    list<Point>::iterator p; // 반복자 타입

    // auto p1 = s.begin();
    auto p1 = begin(s);

    list<int> s1;
    s1.push_back(10);
    s1.push_back(20);
    s1.push_back(30);
    // cout << s1.pop_back() << endl; // error
    cout << s1.back() << endl;
    s1.pop_back();
    cout << s1.back() << endl;
}
```

## 3. Sequence Container

- 각 요소가 삽입된 순서대로 선형적으로 놓인 컨테이너

![image](https://user-images.githubusercontent.com/23397039/215420451-34feff8c-a8be-4c86-bf69-e4b77b0e14c1.png){: .align-center}

![image](https://user-images.githubusercontent.com/23397039/215420525-480982ee-503d-45e4-af10-e1b88ed7c1f1.png){: .align-center}

```cpp
#include <iostream>
#include <list>
#include <vector>
#include <deque>
#include <forward_list>
#include <array>

using namespace std;

int main()
{
    // list<int> s;
    deque<int> s;
    // vector<int> s;

    s.push_front(10); // vector는 불가능
    s.push_back(20);

    for (auto& i : s)
        cout << i << endl;

    cout << s[0] << endl; // deque만 가능
}
```

### 3.1. Vector

```cpp
template <typename T,
					typename Allocator = allocator<t> >

// vector 생성
vector<int> v(10, 0); // 10개를 0으로
vector<int> v{10, 0); // 10, 0 2개 인자
```

- vector 기본 사용법

```cpp
#include <iostream>
#include <list>
#include <vector>
#include <deque>
#include <forward_list>
#include <array>

using namespace std;

int main()
{
    vector<int> v;
    vector<int> v2(10);
    vector<int> v3(10, 3);
    vector<int> v4(v3);

    v.push_back(10);
    v.push_back(20);
    v.insert( begin(v) + 1, 30); // 10 20 30

    for (auto& i : v)
        cout << i << endl;

    int n = v.front();
    int n1 = v[0];

    int x[5] = { 1,2,3,4,5 };
    v.assign(x, x+5); // 1 2 3 4 5
    for (auto& i : v)
        cout << i << endl;

    v[100] = 10; // 예외 없이 runtime error
    v.at(100) = 10; // 예외 발생, 그러나 성능저하로 많이 사용되지는 않음
}
```

- size, capacity

```cpp
#include <iostream>
#include <list>
#include <vector>
#include <deque>
#include <forward_list>
#include <array>

using namespace std;

int main()
{
    vector<int> v(10, 0);
    v.resize(7);

    cout << v.size() << endl; // 7
    cout << v.capacity() << endl; // 10
    
    v.push_back(10);
    cout << v.size() << endl; // 8
    cout << v.capacity() << endl; // 10

    v.pop_back();
    cout << v.size() << endl; // 7
    cout << v.capacity() << endl; // 10

    v.shrink_to_fit();
    cout << v.size() << endl; // 7
    cout << v.capacity() << endl; // 7

    v.push_back(20);
    cout << v.size() << endl; // 8
    cout << v.capacity() << endl; // 14
}
```

- vector 포인터값

```cpp
#include <iostream>
#include <list>
#include <vector>
#include <deque>
#include <forward_list>
#include <array>

using namespace std;

void print(int * arr, int sz){
    for (int i = 0; i < sz; i++ )
        cout << arr[i] << endl;
}

int main()
{
    vector<int> v = {1,2,3,4,5,6,7,8,9,10};

    // print(v, v.size()); // error
    print(&v[0], v.size());
    print(v.data(), v.size());
}
```

### 3.2. Array

- 배열은 stack에 memory를 잡고
- vector는 heap에 memory를 잡고 stack에 포인터를 가짐

![image](https://user-images.githubusercontent.com/23397039/215420734-27e09417-8791-4c74-a7ad-ec59ea07b005.png){: .align-center}

- std::array는 배열과 동일하게 stack에 버퍼를 만드는 컨테이너
- 실제 배열을 사용하므로 전방삽입, 후방삽입, 임의 삽입이 모두 불가능

```cpp
#include <iostream>
#include <vector>

using namespace std;

/*
template<typename T, int N> struct array
{
    T buf[N];
    typedef T* iterator;

    iterator begin() { return buf; }
    iterator end() { return buf + N; }

    int size() const { return N; }
};
*/

#include <array>

int main()
{
    int x[5]; // stack에 5개
    vector<int> v(5); // heap에 만들고 stack에 포인터를 가짐

    array<int, 5> arr = {1,2,3,4,5};

    // arr.push_back(10); 

    cout << arr.size() << endl;
}
```

### 3.3. 사용자 정의 타입과 컨테이너

- 사용자 정의 타입을 컨테이너에 넣을 때 디폴트 생성자가 없는 경우, 복사 생성을 위한 객체를 전달
- sort의 조건자 버전을 사용
- 사용자 정의 타입에 < 와 == 연산자를 제공

```cpp
#include <iostream>
#include <vector>
#include <algorithm>

using namespace std;

class Point
{
public:
    int x,y;
public:
    // 디폴트 생성자가 없는 경우
    Point(int a, int b) : x(a), y(b) {}

    // < 와 == 를 제공
    bool operator<(const Point& p) const {
        return x < p.x;
    }
    bool operator==(const Point& p) const {
        return x == p.x;
    }

};

#include <array>

int main()
{
    vector<Point> v1;
    // vector<Point> v2(10); // error
    vector<Point> v2(10, Point(0,0));

    // v2.resize(20); // error
    v2.resize(20, Point(0, 0));

    // sort(begin(v2), end(v2)); // error
    // sort(begin(v2), end(v2),
    //         [](const Point& p1, const Point& p2) { return p1.x > p2.x; } );
    sort(begin(v2), end(v2)); // <, == 를 class에서 제공
}
```

- std::rel_ops namespace
    - >, !=, <=, >= 연산자의 일반화 버전을 제공함
    - 

```cpp
#include <iostream>
#include <vector>
#include <algorithm>

using namespace std;
using namespace std::rel_ops;

/*
namespace std
{
    namespace rel_ops
    {
        template<typename T>
        bool operator != ( const T& lhs, const T& rhs) {
            return ! (lhs == rhs);
        }
    }
}
*/
class Point
{
public:
    int x,y;
public:
    Point(int a, int b) : x(a), y(b) {}

    // < 와 == 를 제공
    bool operator<(const Point& p) const {
        return x < p.x;
    }
    bool operator==(const Point& p) const {
        return x == p.x;
    }

};

int main()
{
    Point p1(1,1);
    Point p2(2,2);

    p1 == p2;
    p1 < p2;

    // using namespace std::rel_ops; 필요
    p1 > p2;
    p1 != p2;
}
```

- 컨테이너에 객체를 넣는 방법

```cpp
#include <iostream>
#include <vector>

using namespace std;

class Point
{
    int x,y;
public:
    Point(int a, int b) : x(a), y(b) { cout << "Point()" << endl; }

    ~Point() { cout << "~Point()" << endl; }
    Point(const Point&) { cout << "Point(const Point&)" << endl; }
    Point(Point&&) { cout << "Point&&" << endl; }
};

int main()
{
    vector<Point> v;

    // 1.
    // Point p1(10, 20);
    // v.push_back(p1);

    // 2. 임시객체
    // v.push_back(Point(10, 20));

    // 3. 객체 자체를 vector가 만들게 함
    v.emplace_back( 10,20 );

    // 4. move를 사용
    Point p1(10, 20);
    // v.push_back(p1);
    v.push_back(std::move(p1));

    cout << "---------" << endl;
}
```

## 4. associative container

### 4.1. Set

- 트리로 구성되어 있으며, C++의 경우 set이 RB tree로 구현되어 있음

```cpp
#include <iostream>
#include <set>

using namespace std;

int main()
{
    set<int> s; // RB tree

    s.insert(30);
    s.insert(40);
    s.insert(20);
    s.insert(10);
    s.insert(45);
    s.insert(25);

    auto p = begin(s); // 왼쪽 마지막 자식을 가리킴
    while (p != end(s)) {
        cout << *p << endl;
        ++p;
    }
}
```

- set의 기본모양

```cpp
template<
	class Key,
	class Compare = std::less<Key>
  class Allocator = std::allocator<Key>
> class set;
```

- 요소를 추가하는 방법
    - push_xxx는 사용 불가, insert 또는 emplace 로만 사용이 가능
    - 반복자를 통해서 값을 변경할 수 없음
- 요소를 검색하는 방법
    - tree 라서 find 알고리즘을 사용하기 보단 멤버함수 find를 사용하는 것이 좋음

```cpp
#include <iostream>
#include <set>

using namespace std;

int main()
{
    // set<int> s;
    set<int, greater<int>> s; // RB tree

    // s.push_front(10); // error
    s.insert(30);
    s.insert(40); // < 연산으로 비교, > 로 하려면 정책을 바꾸면 됨
    s.insert(20);
    s.insert(10);
    s.insert(45);
    s.insert(25);

    auto p = begin(s); // 왼쪽 마지막 자식을 가리킴
    // *p = 10; // error, 상수임
    while (p != end(s)) {
        cout << *p << endl;
        ++p;
    }

    // find는 선형검색이기 때문에 트리에서는 안좋음
    // 멤버함수 find가 있으며 이진 검색을 함
    auto p2 = s.find(10);
    cout << *p2 << endl;
}
```

- set은 중복을 허용하지 않음
    - multiset은 중복을 허용
- set의 return 타입
    - set : pair<iterator, bool>
    - multiset : iterator

```cpp
#include <iostream>
#include <set>

using namespace std;

int main()
{
    // set<int> s;
    multiset<int> s;

    s.insert(30);
    s.insert(40);
    s.insert(20);
    s.insert(10);
    s.insert(45);
    s.insert(25);

    // 이미 들어가 있는 경우
    // pair<set<int>::iterator, bool> ret = s.insert(20);
    // set은 pair를 반환
    // multiset : iterator만 반환
    auto ret = s.insert(20);
    // if ( ret.second == false )
    //     cout << "fail" << endl;

    for (auto& n : s)   cout << n << endl;
}
```

- set에 사용자 정의 타입을 넣으려면
    - 사용자 정의 타입 안에 < 연산을 제공하거나
    - < 연산을 수행하는 함수 객체를 만들어 set의 2번째 템플릿 인자로 전달

## 참고
codenuri 강석민 강사 강의 내용기반으로 정리한 내용입니다.  
[코드누리](https://github.com/codenuri)  

<br>

    This is personal diary for study documents.
    Please comment if I'm wrong or missing something else 😄. 

[Top](#){: .btn .btn--primary }{: .align-right}