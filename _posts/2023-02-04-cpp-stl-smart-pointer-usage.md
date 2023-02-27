---
published: true
title:  "[C++] STL 스마트포인터(Smart pointer) 사용법"
excerpt: "Smart pointer 에 대해 알아보기"

categories:
  - Cpp
tags:
  - [스마트포인터, C++, Cpp, STL, Smart, Pointer]

toc: true
toc_sticky: true
author: chachayelmo
sitemap:
  changefreq : daily
  priority : 1.0
comment: true
date: 2023-02-04
last_modified_at: 2023-02-04
---

## 1. 스마트 포인터 개념

- 자동화된 자원관리 등의 기능을 제공하는 포인터
- Raw Pointer
    - 생성자, 소멸자 등을 가질 수 없음
- Smart Pointer
    - 생성자, 소멸자 등을 가짐
    - 생성/복사/대입/소멸의 과정에 추가적인 기능을 수행

### 1.1. 스마트 포인터의 원리

- 내부적으로 객체의 주소를 보관하기 위해 포인터 멤버를 가짐
- → 와 * 연산자를 재정의 되어 있음

```cpp
#include <iostream>
#include <memory>
using namespace std;

class Car {
    int color;
public:
    ~Car() { cout << "~Car()" << endl; }
    void Go() { cout << "go()" << endl; }
};

int main()
{
    // Car* p = new Car;
    shared_ptr<Car> p( new Car );

    p->Go();    // p.operator->()
    (*p).Go();  // p.operator*()

    // delete p;
}
```

## 2. Shared_ptr

- copy 초기화는 안되고 direct 초기화만 가능

### 2.1. shared_ptr의 메모리 구조

![image](https://user-images.githubusercontent.com/23397039/216742217-8d378325-2f8b-49a8-b7aa-367cebf8aaa1.png){: .align-center}

- shared_ptr 생성 시, 참조 계수 등을 관리하는 control block이 생성됨

```cpp
#include <iostream>
#include <memory>
using namespace std;

class Car {
    int color;
    int speed;
public:
    Car(int c=0, int s=0) : color(c), speed(s) {}
    ~Car() { cout << "~Car()" << endl; }
    void Go() { cout << "go()" << endl; }
};

int main()
{
    int a = 0; // copy initialization
    int a1(0); // direct initialization

    // shared_ptr<Car> p = new Car; // error
    shared_ptr<Car> p1(new Car); // ok

    shared_ptr<Car> p2 = p1;
}
```

- 삭제자 변경
    - shared_ptr<> 생성자의 2번째 인자로 삭제자 전달
    - 함수, 함수 객체, 람다 표현식 등을 사용 가능
    - 할당자 변경은 3번째 인자로 가능
    
    ```cpp
    #include <iostream>
    #include <memory>
    using namespace std;
    
    class Car {
        int color;
        int speed;
    public:
        Car(int c=0, int s=0) : color(c), speed(s) {}
        ~Car() { cout << "~Car()" << endl; }
        void Go() { cout << "go()" << endl; }
    };
    
    void foo(Car* p)
    {
        cout << "Delete Car" << endl;
        delete p;
    }
    
    int main()
    {
        // shared_ptr<Car> p(new Car);
        // shared_ptr<Car> p(new Car, foo);
        shared_ptr<Car> p(new Car,
                          [](Car* p){ delete p; } // deleter
                          // allocator
                          );    
    }
    ```
    

### 2.2. shared_ptr 과 배열

- 삭제자를 변경해야 함
- [] 연산자가 제공되지 않음
- shared_ptr을 사용해서 배열을 관리하는 것은 권장되지 않음
- C++17 이후
    - shared_ptr<T[]>를 사용

```cpp
#include <iostream>
#include <memory>
using namespace std;

class Car {
    int color;
    int speed;
public:
    Car(int c=0, int s=0) : color(c), speed(s) {}
    ~Car() { cout << "~Car()" << endl; }
    void Go() { cout << "go()" << endl; }
};

int main()
{
    // shared_ptr<Car> p1(new Car[10]); // BUG - delete[], [] 연산 없음
    // shared_ptr<Car> p1(new Car[10], [](Car* p){ delete[] p; });
    shared_ptr<Car[]> p1(new Car[10]); // ok, C++17

    p1[0].Go();
}
```

### 2.3. 주요 멤버 함수

- → 연산과 . 연산
    - → 연산 : 객체의 멤버에 접근
    - . 연산 : shared_ptr 자체 멤버에 접근
- get : 대상체의 포인터 반환
- use_count : 참조계수 반환
- reset : 대상체 변경
- swap : 대상체 교환

```cpp
#include <iostream>
#include <memory>
using namespace std;

class Car {
    int color;
    int speed;
public:
    Car(int c=0, int s=0) : color(c), speed(s) {}
    ~Car() { cout << "~Car()" << endl; }
    void Go() { cout << "go()" << endl; }
};

int main()
{
    shared_ptr<Car> p1(new Car);

    p1->Go();

    Car* p = p1.get();
    cout << p << endl;

    shared_ptr<Car> p2 = p1;
    cout << p1.use_count() << endl; // 2

    // p1 = new Car; // error
    p1.reset(new Car); // ok
    p1.reset();

    p1.swap(p2);
}
```

### 2.4. make_shared

- 메모리 조각화 현상을 해결하기 위해 나옴
- 대상 객체와 제어 블럭을 동시에 메모리 할당하므로 효율적
- 예외 상황에 좀 더 안전
- 메모리 할당/해지 방식을 변경하려면 allocate_shared를 사용

```cpp
#include <iostream>
#include <memory>
using namespace std;

class Car {
    int color;
    int speed;
public:
    Car(int c=0, int s=0) : color(c), speed(s) {}
    ~Car() { cout << "~Car()" << endl; }
    void Go() { cout << "go()" << endl; }
};

void* operator new( size_t sz) {
    cout << "new sz : " << sz << endl;
    return malloc(sz);
}

int main()
{
    // 메모리 조각화 현상
    // shared_ptr<Car> p1(new Car); // new sz = 8  : Car object
                                 // new sz = 24 : control block

    shared_ptr<Car> p1 = make_shared<Car>(1, 2); // new sz = 24 : Car obj + control block
    // shared_ptr<Car> p2(make_shared<Car>()); // 가능

    // 예외 상황에서 안전
    // 아래 3가지 일이 어떤 순서로 일어나는 지 표준에 언급되지 않음
    // 따라서 1번 후 3번으로 넘어가면 memory leak이 발생할 수 있음
    // f( shared_ptr<Car>(new Car), foo() );  // 1.메모리 할당
                                              // 2. shared_ptr 만듬
                                              // 3. foo 함수 호출

    // make_shared를 사용
    // f( make_shared<Car>(), foo() );

    // 메모리 할당/해지 변경하는 방식
    // shared_ptr<Car> p1 = allocate_shared<Car>(MyAlloc<Car>());
}
```

### 2.5. 주의사항

- raw pointer를 사용해서 2개 이상의 shared_ptr를 생성하면 안됨

![image](https://user-images.githubusercontent.com/23397039/216742243-1e473a39-350e-4641-a823-185694fb31b8.png){: .align-center}

```cpp
#include <iostream>
#include <memory>
using namespace std;

class Car {
    int color;
    int speed;
public:
    Car(int c=0, int s=0) : color(c), speed(s) {}
    ~Car() { cout << "~Car()" << endl; }
    void Go() { cout << "go()" << endl; }
};

void* operator new( size_t sz) {
    cout << "new sz : " << sz << endl;
    return malloc(sz);
}

int main()
{
    Car* p = new Car;

    shared_ptr<Car> sp1(p); // 제어 블럭 생성
    shared_ptr<Car> sp2(p); // 제어 블럭 생성, free 가 2번나서 에러

    // 아래와 같이 사용해야 됨, 만들면서 초기화
    shared_ptr<Car> sp3(new Car); // RAII
    shared_ptr<Car> sp3 = make_shared<Car>();
}
```

### 2.6. enable_shared_from_this

- Worker의 객체 수명
    - 멀티스레드인 경우, 주 스레드의 sp1도 사용하지 않고, 새로운 스레드도 종료 되었을 때 파괴 되어야 함
    - Worker 객체가 자기 자신의 참조계수를 증가 해야 함

![image](https://user-images.githubusercontent.com/23397039/216742267-42e6e1e3-d719-4203-b08c-ff6cc5ef4333.png){: .align-center}

- enable_shared_from_this 는 this를 가지고 제어 블럭을 공유하는 shared_ptr을 만들 수 있게 함
- CRTP 기술
- shared_from_this() 함수를 사용
- shared_from_this()를 호출하기 전에 반드시 제어블럭이 생성되어 있어야 함

```cpp
#include <iostream>
#include <memory>
#include <thread>
using namespace std;

class Car {
    int color;
    int speed;
public:
    Car(int c=0, int s=0) : color(c), speed(s) {}
    ~Car() { cout << "~Car()" << endl; }
    void Go() { cout << "go()" << endl; }
};

class Worker : public enable_shared_from_this<Worker> // CRTP
{
    Car c;
    shared_ptr<Worker> holdMe;
public:
    void Run()
    {
        holdMe = shared_from_this(); // 사용, 참조계수가 올라감

        thread t(&Worker::Main, this);
        t.detach();
    }
    void Main()
    {
        c.Go(); // 멤버 data(Car) 를 사용
        cout << "finish thread" << endl;

        holdMe.reset();
    }
};

int main()
{
    {
        shared_ptr<Worker> sp = make_shared<Worker>();
        sp->Run(); // 스레드 수행
    } // Worker가 파괴되어 Car가 없는 문제를
      // 해결하기 위해 enablle_shared_from_this 사용
}
```

## 3. weak_ptr

### 3.1. 상호 참조 문제

![image](https://user-images.githubusercontent.com/23397039/216758288-f955e0ac-e017-4904-b547-a362c65218b9.png){: .align-center}

```cpp
#include <iostream>
#include <memory>
using namespace std;

struct People
{
    string name;
    // shared_ptr<People> bf;
    People* bf; // Raw Pointer : 참조 계수가 증가하지 않음
                // 단점 : 대상 객체가 파괴되었는 지 알 수 없음
                // 솔루션 : weak_ptr
    // weak_ptr<People> bf;
    People(string s) : name(s) {}
    ~People() { cout << "~People :" << name << endl; }
};

int main()
{
    shared_ptr<People> p1( new People("KIM"));
    {
        shared_ptr<People> p2( new People("LEE"));

        // p1->bf = p2;
        // p2->bf = p1;
        p1->bf = p2.get();
        p2->bf = p1.get();
    } // p2 자원 파괴

    if (p1->bf != 0)
        cout << p1->bf->name << endl;
}
```

### 3.2. weak_ptr

- use count가 증가하지 않는 스마트 포인터
- expired() 멤버 함수로 대상 객체의 유효성을 확인할 수 있음
- use_count가 0일 때 대상 객체는 파괴 되지만 제어 블록은 use_count, weak_count 모두 0일 때 파괴

```cpp
#include <iostream>
#include <memory>
using namespace std;

struct People
{
    string name;
    weak_ptr<People> bf;
    People(string s="") : name(s) {}
    ~People() { cout << "~People :" << name << endl; }
    void Go() { cout << "Go" << endl; }
};

int main()
{
    // shared_ptr<People> wp; // use_count 증가
    weak_ptr<People> wp; // 증가 x

    // {
        shared_ptr<People> sp(new People);
        wp = sp;
        cout << sp.use_count() << endl;
    // } // 대상 객체 파괴

    if (wp.expired())
        cout << "destroy" << endl;
    else {
        cout << "exist" << endl;

        // weak_ptr을 사용해서는 대상객체에 접근할 수 없음
        // wp->Go(); // error

        // weak_ptr 을 다시 shared_ptr로 만들어야 함
        shared_ptr<People> sp2 = wp.lock();

        if( sp2 )
            sp2->Go();
    }
        
}
```

```cpp
#include <iostream>
#include <memory>
using namespace std;

struct People
{
    string name;
    weak_ptr<People> bf;
    People(string s) : name(s) {}
    ~People() { cout << "~People :" << name << endl; }
};

int main()
{
    shared_ptr<People> p1( new People("KIM"));
    // {
    shared_ptr<People> p2( new People("LEE"));

    p1->bf = p2;
    p2->bf = p1;
    // } // p2 자원 파괴

    // if (p1->bf.expired()) {}

    shared_ptr<People> sp2 = p1->bf.lock();
    if (sp2)
        cout << sp2->name << endl;
    else
        cout << "destory" << endl;
}
```

## 4. unique_ptr

- Raw Pointer와 동일한 크기를 가짐
- 단, 삭제자 변경 시 크기가 커질 수 있음

```cpp
#include <iostream>
#include <memory>
using namespace std;

struct People
{
    string name;
    weak_ptr<People> bf;
    People(string s="") : name(s) {}
    ~People() { cout << "~People :" << name << endl; }
    void Go() { cout << "Go" << endl; }
};

int main()
{
    shared_ptr<People> sp1(new People);
    shared_ptr<People> sp2 = sp1; // ok. use count 증가

    unique_ptr<People> up1(new People); // 자원 독점
    // unique_ptr<People> up2 = up1; // error
    cout << sizeof(sp1) << endl; // 16
    cout << sizeof(up1) << endl; // 8
}
```

- 복사는 안되지만 이동은 가능

```cpp
#include <iostream>
#include <memory>
using namespace std;

struct People
{
    string name;
    weak_ptr<People> bf;
    People(string s="") : name(s) {}
    ~People() { cout << "~People :" << name << endl; }
    void Go() { cout << "Go" << endl; }
};

int main()
{
    unique_ptr<People> up1(new People); // 자원 독점
    // unique_ptr<People> up2 = up1; // error

    unique_ptr<People> up2 = move(up1); // ok
}
```

- 삭제자 변경 가능

```cpp
#include <iostream>
#include <memory>
using namespace std;

struct Deleter
{
    void operator()(int* p) const
    {
        delete p;
    }
};

int main()
{
    // 1. 함수 객체 사용
    // unique_ptr<int, Deleter> up1(new int);

    // 2. 함수 포인터 사용
    // unique_ptr<int, void(*)(int*)> up1(new int, foo);

    // 3. 람다 표현식 사용
    auto f = [](int* p) { delete p; cout << "lambda" << endl; };

    unique_ptr<int, decltype(f)> up(new int, f);

    // 4. 배열을 관리하고 싶을 때, C++11 부터 가능
    unique_ptr<int[]]> up(new int[10]);
}
```

### 4.1. shared_ptr 과 unique_ptr 의 호환성

```cpp
#include <iostream>
#include <memory>
using namespace std;

int main()
{
    shared_ptr<int> sp(new int);
    unique_ptr<int> up(new int);
    
    shared_ptr<int> sp1 = up;       // error
    shared_ptr<int> sp2 = move(up); // ok

    unique_ptr<int> up1 = sp;       // error
    unique_ptr<int> up2 = move(sp); // error
}
```

```cpp
#include <iostream>
#include <memory>
using namespace std;

class Shape {};
class Rect : public Shape {};
class Circle : public Shape {};

unique_ptr<Shape> CreateShape(int type)
{
    unique_ptr<Shape> p;
    switch(type)
    {
        case 1: p.reset(new Rect); break;
        case 2: p.reset(new Circle); break;
    }
    return p;
}

int main()
{
    // CreateShape는 unique_ptr로 만들어야
    // 받는 입장에서 unique/shared ptr 둘 다 사용 가능
    unique_ptr<Shape> p1 = CreateShape(1);
    shared_ptr<Shape> p2 = CreateShape(2);
}
```

## 참고
codenuri 강석민 강사 강의 내용기반으로 정리한 내용입니다.  
[코드누리](https://github.com/codenuri)  

<br>

    This is personal diary for study documents.
    Please comment if I'm wrong or missing something else 😄. 

[Top](#){: .btn .btn--primary }{: .align-right}