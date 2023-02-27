---
published: true
title:  "[C++] STL 알고리즘(algorithm)"
excerpt: "algorithm 에 대해 알아보기"

categories:
  - Cpp
tags:
  - [알고리즘, C++, Cpp, STL, algorithm]

toc: true
toc_sticky: true
author: chachayelmo
sitemap:
  changefreq : daily
  priority : 1.0
comment: true
date: 2023-01-27
last_modified_at: 2023-01-27
---

## 1. Algorithm

### 1.1. 멤버 함수가 아닌 일반 함수로 되어 있음

- STL 알고리즘은 특정한 컨테이너가 아닌 다양한 컨테이너에 대해서 사용할 수 있음

```cpp
#include <algorithm>
// 특정한 함수는 다른 헤더에 들어있음
#include <numeric>
#include <memory>
```

### 1.2. 알고리즘은 컨테이너를 알지 못함

- remove 알고리즘은 컨테이너의 크기를 줄이지 않음
- 인자로 전달된 반복자가 어떤 컨테이너인지 알수 없음
- 컨테이너의 크기를 줄이려면 컨테이너의 erase 멤버함수를 사용해야 됨

![image](https://user-images.githubusercontent.com/23397039/215021061-65c240c1-9c2c-4106-bda9-78758191a762.png){: .align-center}

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

int main()
{
    vector<int> v = {1,2,3,1,2,3,1,2,3,1};
    auto p = remove(begin(v), end(v), 3);
    for (auto& i : v)
        cout << i << " "; // 1,2,1,2,1,2,1 2,3,1
    cout << endl;

    v.erase(p, end(v));
    
    for (auto& i : v)
        cout << i << " "; // 1,2,1,2,1,2,1    
}
```

### 1.3. 알고리즘보다 멤버 함수가 좋은 경우가 있음

- list에서 remove를 수행할 때는 요소를 당기는 것보다는 요소 자체를 제거하는 것이 효율적
- list에서는 remove 알고리즘보다는 remove 멤버 함수를 사용하는 것이 좋음

```cpp
#include <iostream>
#include <vector>
#include <list>
#include <algorithm>
using namespace std;

int main()
{
    // vector<int> v = {1,2,3,1,2,3,1,2,3,1};
    list<int> v = {1,2,3,1,2,3,1,2,3,1};
    // 리스트의 경우 알고리즘 remove를 사용해서 앞으로 당기는 것보다 자체를 제거하는 것이 좋음
    v.remove(3);
    for (auto& i : v)
        cout << i << " ";
}
```

## 2. 함수 객체

- 최초에는 () 연산자를 재 정의해서 함수처럼 사용 가능한 객체
- 요즘에는 () 연산자를 사용해서 함수처럼 호출 가능한 모든 객체를 뜻함

```cpp
#include <iostream>
using namespace std;

// () 를 사용해서 호출하는 것들
// 1. 함수
// 2. 함수 포인터
// 3. ()를 재정의한 클래스
// 4. 람다표현식

struct Plus
{
    int operator()(int a, int b) const
    {
        return a+b;
    }
};

int main()
{
    Plus p;
    int n = p(1,2); // p.operator()(1,2)
    cout << n << endl;

    // a + b  -> a.operator+(b)
    // a - b  -> a.operator-(b)
    // a()    -> a.operator()()
    // a(1,2) -> a.operator()(1,2)
}
```

- 장점
    - 알고리즘에 전달 시 일반 함수는 인라인 치환이 안되지만 함수 객체는 인라인 치환 가능
    - 상태를 가질 수 있음
- STL이 제공하는 함수 객체

```cpp
#include <functional>
// 산술연산
plus<>
minus<>
multiplies<>
divides<>
modulus<>
negate<>
// 비교연산
equal_to<>
not_equal_to<>
greater<>
less<>
greater_equal<>
less_equal<>
// 논리연산
logical_and<>
logical_or<>
logical_not<>
```

```cpp
#include <iostream>
#include <functional>
using namespace std;

int main()
{
    plus<int> p;
    cout << p(1,2) << endl;

    multiplies<int> m;
    cout << m(3,4) << endl;
}
```

## 3. 알고리즘과 함수

- STL 알고리즘은 함수를 인자로 가지는 경우가 많음
    - 알고리즘의 활용도를 높여줌
    - for_each, transform
    - 단항 함수 : 인자가 1개인 함수
    - 이항 함수 : 인자가 2개인 함수
- 일반 함수 뿐 아니라, 함수 객체, 람다 표현식을 사용할 수 있음

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
#include <string>
#include <fstream>
using namespace std;

// for_each는 인자를 1개밖에 못받기 때문에 ostream 등을 추가할 수 없음
void foo(int& a){
    cout << a << " ";
}

// 함수객체를 활용
struct Show
{
    string sep;
    ostream& os;
    Show(ostream& o = cout, string s = "\n") : os(o), sep(s) {}
    void operator()(int a) const {
        os << a << sep;
    }
};

int main()
{
    vector<int> v = {1,2,3,4,5,6,7,8,9,10};

    // 1. 일반 함수 전달
    for_each(begin(v), end(v), foo);
    cout << endl;
    // 2. 함수 객체 전달
    Show s(cout, "\t");
    for_each(begin(v), end(v), s);
    // 3. 람다 표현식
    for_each(begin(v), end(v), [](int a) {cout << a << endl;} );
}
```

## 4. std::transform

- 인자로 전달된 컨테이너의 모든 요소에 연산을 적용 후 다른 컨테이너에 복사하는 알고리즘
- 단항 함수인 경우

```cpp
#include <iostream>
#include <list>
#include <algorithm>
using namespace std;

// 단항 함수인 경우
int foo(int a) { return a + 10; }

int main()
{
    list<int> s1 = {1,2,3,4,5};
    list<int> s2 = {0,0,0,0,0};

    // transform( begin(s1), end(s1), begin(s2), foo);
    transform( begin(s1), end(s1), back_inserter(s2), foo);

    for(auto& i : s2)
        cout << i << " "; // 11 12 13 14 15
                          // back_inserter 0 0 0 0 0 11 12 13 14 15
}
```

- 이항 함수인 경우

```cpp
#include <iostream>
#include <list>
#include <algorithm>
#include <functional>
using namespace std;

// 이항 함수인 경우
int foo(int a, int b) { return a + b; }

int main()
{
    list<int> s1 = {1,2,3,4,5};
    list<int> s2 = {1,2,3,4,5};
    list<int> s3 = {0,0,0,0,0};

    // 일반 함수
    // transform( begin(s1), end(s1), begin(s2), begin(s3), foo);
    // 함수 객체
    // transform( begin(s1), end(s1), begin(s2), begin(s3), multiplies<int>());
    // 람다 표현식
    transform( begin(s1), end(s1), begin(s2), begin(s3),
                            [](int a, int b){ return a + b;} );

    for(auto& i : s3)
        cout << i << " "; // 2 4 6 8 10
                          // multiplies 1 4 9 16 25
}
```

## 5. 여러가지 알고리즘

- 조건자 (Predicator)
    - bool 타입을 리턴하는 함수 객체

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

bool foo( int a) { return a % 3 == 0; }

int main()
{
    vector<int> v = {6,9,3,1,2};

    auto p = find( begin(v), end(v), 3);
    // 주어간 구간에서 첫번째 나오는 3의 배수를 찾고 싶을 때
    auto p1 = find_if( begin(v), end(v), foo);
    auto p2 = find_if( begin(v), end(v), 
                            [](int a) { return a % 3 == 0;});
    cout << *p << endl;     // 3
    cout << *p1 << endl;    // 6
    cout << *p2 << endl;    // 6
}
```

- 알고리즘의 4가지 형태
    - 기본 버전
    - 조건자 버전
        - _if
    - 복사 버전
        - _copy
    - 조건자 복사 버전
        - _copy_if

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

template<typename T>
void show(T& v) {
    for( auto& i : v)
        cout << i << " ";
    cout << endl;
}

bool foo( int a) { return a % 3 == 0; }

int main()
{
    vector<int> v1 = {1,2,3,4,5,6,7,8,9,10};
    vector<int> v2 = {0,0,0,0,0,0,0,0,0,0};

    // 기존 버전
    // remove( begin(v1), end(v1), 3);

    // 조건자 버전
    // remove_if( begin(v1), end(v1), [](int a){ return a % 3 == 0;});

    // 복사 버전, 리턴 값은 v2의 반복자
    // auto p = remove_copy( begin(v1), end(v1), begin(v2), 3);
    // v1 = 1 2 3 4 5 6 7 8 9 10
    // v2 = 1 2 4 5 6 7 8 9 10 0

    auto p = remove_copy_if( begin(v1), end(v1), begin(v2), 
                                      [](int a){ return a % 3 == 0;});
    // v1 = 1 2 3 4 5 6 7 8 9 10
    // v2 = 1 2 4 5 7 8 10 0 0 0
    show(v1); 
    show(v2); 
}
```

- remove_copy
    - copy 후에 remove 하는 것 보다 동시에 하는 것이 빠름
- sort_copy (x)
    - copy 후에 sort해야되기 때문에 성능 향상이 없어서 제공하지 않음

## 6. std::bind

- M항 함수의 인자를 고정한 새로운 함수를 생성
- C++11 부터 지원

```cpp
#include <functional>
```

- placeholder(_1, _2, … ) 는 std::placeholders namespace에 있음
- 일반 함수 뿐 아니라 함수 객체, 멤버 함수, 람다 표현식 등에도 사용 가능

```cpp
#include <iostream>
#include <functional>
#include <algorithm>
using namespace std::placeholders;
using namespace std;

void foo( int a, int b, int c, int d)
{
    printf("%d, %d, %d, %d\n", a,b,c,d);
}

int main()
{
    // 4항 함수
    foo(1,2,3,4);
    std::bind(&foo, 1,2,3,4)();

    // 4항 함수를 2항으로
    std::bind(&foo, 10, _2, _1, 20)(1,2); // 10, 2, 1, 20

    // 4항 함수를 3항으로
    std::bind(&foo, 10, _2, _1, _3)(30,1,2); // 10, 1, 30, 2

    modulus<int> m;
    cout << m(10, 3) << endl;
    bind(m, 10, 3)();   // 10 % 3
    bind(m, _1, 3)(7); //  7 % 3

    // 3의 배수가 아닌 것 찾기
    int x[3] = {2,6,9};
    auto p = find_if(x, x+3, bind(m, _1, 3));
    cout << *p << endl; // 1
}
```

## 참고
codenuri 강석민 강사 강의 내용기반으로 정리한 내용입니다.  
[코드누리](https://github.com/codenuri)  

<br>

    This is personal diary for study documents.
    Please comment if I'm wrong or missing something else 😄. 

[Top](#){: .btn .btn--primary }{: .align-right}