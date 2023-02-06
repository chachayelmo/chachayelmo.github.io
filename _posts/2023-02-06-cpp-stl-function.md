---
published: true
title:  "[Programming] STL function"
excerpt: "function 에 대해 알아보기"

categories:
  - Cpp
tags:
  - [C++, Cpp, STL, Bind, Function]

toc: true
toc_sticky: true
 
date: 2023-02-06
last_modified_at: 2023-02-06
---

## 1. bind

- 함수의 인자를 고정한 새로운 함수를 만들 때 사용
- <functional> 헤더
- namespace std::placeholders;
- reference_wrapper, ref()
    - bind의 인자를 고정할 때 값이 아닌 참조 방식으로 고정

```cpp
#include <iostream>
#include <functional>
using namespace std;
using namespace std::placeholders;

void f1(int a, int b, int c) { printf("f1 : %d %d %d\n", a,b,c); }
void f2(int& a) { a = 20; }

int main()
{
    bind(&f1, 1,2,3)();         // f1(1,2,3)
    bind(&f1, 1,2,_1)(10);      // f1(1,2,10)
    bind(&f1, 1, _2,_1)(10,20); // f1(1,20,10) 

    int n = 0;
    // bind는 값으로 넘김
    bind(&f2, n)(); // f2(n)
    cout << n << endl; // 0

    // reference_wrapper<int> r(n);
    // bind(&f2, r)();

    bind(&f2, ref(n))(); // cref : const 참조
    cout << n << endl; // 20
}
```

### 1.1. 멤버함수, 멤버 데이터 bind

- 모든 종류의 callable object에 사용 가능
- 일반함수/멤버함수/함수객체/람다표현식/멤버데이터

```cpp
#include <iostream>
#include <functional>
using namespace std;
using namespace std::placeholders;

class Test{
public:
    int data = 0;

    void f(int a, int b) {
        data = a;
        printf("f : %d %d\n", a, b);
    }
};

int main()
{
    Test t;

    // 객체를 고정하는 경우
    // 참조
    // bind(&Test::f, &t, 1, 2)(); // t.f(1,2);
    // bind(&Test::f, ref(t), 1, 2)(); // t.f(1,2);
    // cout << t.data << endl; // 1

    // 값
    // bind(&Test::f, t, 1, 2)(); // t.f(1,2);
    // cout << t.data << endl; // 0

    // ======================================

    // 객체를 인자로 전달하는 경우
    // bind(&Test::f, _1, 1, 2)(&t); // t.f(1,2);
    bind(&Test::f, _1, 1, 2)(t); // t.f(1,2);

    cout << t.data << endl; // 1

}
```

## 2. function

- 함수 포인터를 일반화한 개념
- callable object를 보관했다가 나중에 호출할 때 사용
- 함수 signature가 다른 경우 bind()와 함께 사용하면 됨

```cpp
#include <iostream>
#include <functional>
using namespace std;
using namespace std::placeholders;

void f1(int a) { printf("f1 : %d\n", a);}
void f2(int a, int b, int c) { printf("f2 : %d %d %d\n", a,b,c);}

int main()
{
    // void(*f)(int) = &f1;
    // void(*f)(int) = &f2; // error

    function<void(int)> f;
    f = &f1;
    f(10); // f1(10)

    f = bind(&f2, 1, 2, _1);
    f(10); // f2(1,2,10)
}
```

### 2.1. 멤버 함수와 function

- 멤버 함수를 호출하려면 반드시 객체가 있어야 함
- bind를 사용해서 객체를 고정하는 방식
- function으로 호출할 때 객체를 전달하는 방식

```
#include <iostream>
#include <functional>
using namespace std;
using namespace std::placeholders;

class Test{
public:
    int data = 0;

    void f(int a, int b) {
        data = a;
        printf("f : %d %d\n", a, b);
    }
};

int main()
{
    Test t;

    // 1. 일반 함수 모양의 function
    function<void(int)> f1;

    f1 = bind(&Test::f, &t, _1, 20);
    f1(10); // t.f(10, 20)

    // 2. 객체를 function의 인자로 받는 방법
    function<void(Test*, int)> f2; // 포인터로 받음
    f2 = bind(&Test::f, _1, _2, 20);
    f2(&t, 100); // t.f(100,20)

    function<void(Test, int)> f3; // 값으로 받음
    f3 = bind(&Test::f, _1, _2, 20);
    f3(t, 200); // t.f(200,20)

    function<void(Test&, int)> f4; // 참조로 받음
    f4 = bind(&Test::f, _1, _2, 20);
    f4(t, 300); // t.f(300,20)

    cout << t.data << endl; // 300
}
```

### 2.2. 멤버 data와 function

- public 멤버 data도 function에 담을 수 있음
- 멤버 data도 접근하려면 객체가 필요
- bind를 사용해서 객체를 고정하는 방식
- fucntion으로 호출할 때 객체를 전달하는 방식

```cpp
#include <iostream>
#include <functional>
using namespace std;
using namespace std::placeholders;

class Test{
public:
    int data = 10;
};

int main()
{
    Test t1, t2;

    // 1. 객체를 고정하는 방식
    function<int&()> f1;
    f1 = bind(&Test::data, &t1); // t1.data를 보관
    cout << f1() << endl; // 10, getter

    f1() = 20;
    cout << t1.data << endl; // 20

    // 2. 객체를 전달하는 방식
    function<int&(Test*)> f2;
    f2 = bind(&Test::data, _1);
    f2(&t1) = 20;
    f2(&t2) = 30;

    cout << t1.data << endl; // 20
    cout << t2.data << endl; // 30

    function<int&(Test&)> f4;
    f4 = bind(&Test::data, _1);
    f4(t1) = 20;
    f4(t2) = 30;

    cout << t1.data << endl; // 20
    cout << t2.data << endl; // 30
}
```

### 2.3. example

```cpp
#include <iostream>
#include <functional>
#include <map>
#include <string>
#include <list>
using namespace std;
using namespace std::placeholders;

class NotificationCenter
{
    using HANDLER = function<void(void*)>;

    map<string, vector<HANDLER>> notif_map;
public:
    void Register(string key, HANDLER h)
    {
        notif_map[key].push_back(h);
    }

    void Notify(string key, void* param)
    {
        vector<HANDLER>& v = notif_map[key];

        for (auto& f : v)
            f(param);
    }
};

void f1(void* p) { cout << "f1" << endl; }
void f2(void* p, int a, int b) { cout << "f2" << endl; }

int main()
{
    NotificationCenter nc;
    nc.Register("CONNECT_WIFI", &f1);
    nc.Register("CONNECT_WIFI", bind(&f2, _1, 0, 0) );

    nc.Notify("CONNECT_WIFI", (void*)0);

}
```

## 참고
codenuri 강석민 강사 강의 내용기반으로 정리한 내용입니다.  
[코드누리](https://github.com/codenuri)  

<br>

    This is personal diary for study documents.
    Please comment if I'm wrong or missing something else 😄. 

[Top](#){: .btn .btn--primary }{: .align-right}