---
published: true
title:  "[C++] STL 스레드(thread)"
excerpt: "thread 에 대해 알아보기"

categories:
  - Stl
tags:
  - [C++, Cpp, Stl, Thread]

toc: true
toc_sticky: true
author: chachayelmo
sitemap:
  changefreq : daily
  priority : 1.0
comment: true
date: 2023-02-07
last_modified_at: 2023-02-07
---

## 1. thread

- <thread> 헤더
- std::this_thread namespace
- 스레드 관련 4개의 전역 함수를 제공
    - this_thread::get_id() : 스레드 ID 반환
    - this_thread::sleep_for(duration) : duration 동안 스레드 재움
    - this_thread::sleep_until(time_point) : time_point 까지 스레드 재움
    - this_thread::yield() : 다른 스레드 스케줄링

```cpp
#include <iostream>
#include <thread>
#include <chrono>

using namespace std;

int main()
{
    thread::id id = this_thread::get_id();

    cout << id << endl;

    this_thread::sleep_for(3s);
    this_thread::sleep_until(chrono::system_clock::now() + 3s);
    this_thread::yield();
}
```

## 2. std::thread class

- 스레드를 나타내는 클래스
- thread 객체 생성 시 새로운 스레드가 생성
- 스레드는 반드시 join() / detach() 가 되어야 함
- 주요 멤버 함수
    - get_id() : 스레드 ID 반환, thread::id type
    - join() : 스레드가 종료될 때 까지 대기
    - detach() : 스레드가 독립적으로 실행될 수 있도록 허용

```cpp
#include <iostream>
#include <thread>
#include <chrono>

using namespace std;

void foo()
{
    cout << "thread start" << endl;
    this_thread::sleep_for(2s);
    cout << "thread end" << endl;    
}

int main()
{
    thread t(&foo);
    t.join(); // 스레드 종료 대기
    // t.detach();
}
```

## 3. STL의 thread 함수

- 일반 함수, 멤버 함수, 함수 객체, 람다 표현식 등 호출 가능한 모든 요소를 사용할 수 있음
- 인자 갯수에도 제한이 없음

```cpp
#include <iostream>
#include <thread>

using namespace std;

void f1() {}
void f2(int a) {}

struct Worker
{
    void Main() {}
};

struct Functor
{
    void operator()() {}
};

int main()
{
    thread t1(&f1);
    thread t2(&f2, 5);

    Worker w;
    thread t3(&Worker::Main, &w);

    Functor f;
    thread t4(f);

    thread t5( []() {cout << "thread 5 << endl" << endl;} );

    t1.join();
    t2.join();
    t3.join();
    t4.join();
    t5.join();
}
```

## 4. 스레드에 인자 전달

- 스레드 클래스의 생성자 인자로 전달
- std::bind 사용
- 참조로 전달할 때는 ref()를 사용

```cpp
#include <iostream>
#include <thread>
#include <functional>

using namespace std;

void f1(int a, int b) {}

int main()
{
    thread t1(&f1, 1, 2);
    thread t2( bind(&f1, 1, 2) );

    t1.join();
    t2.join();

    int n = 10;
    thread t3(&f1, 1, ref(n));
    t3.join();
    cout << n << endl; // 30
}
```

## 5. 스레드 동기화

- mutex, timed_mutex, recursive_mutex, recursive_timed_mutex, shared_mutex, shared_timed_mutex
- conditional_variable
- call_once
- std::lock_guard
    - RAII 를 사용한 동기화 객체 관리
    - 생성자에서 lock을 하고 소멸자에서 unlock 수행

```cpp
#include <iostream>
#include <thread>
#include <string>
#include <mutex>
using namespace std;

int global = 0;

mutex m;

void f1() {
    lock_guard<mutex> lg(m); // 생성자에서 m.lock
                             // 함수 내 지역변수이기 때문에 함수가 끝나고 lock_guard 객체가
                             // 파괴되서 소멸자를 호출하게 되서 unlock이 불림
                             // 따라서, 대기 중인 스레드는 이전에 호출된 스레드가 unlock되서 다음 스레드 사용 가능
    // m.lock();
    // exceptino이 발생되면 deadlock이 발생, 따라서 lock_guard 를 사용
    global = 100;
    global += 1;
    // m.unlock();
}

int main()
{
    thread t1(&f1);
    thread t2(&f1);

    t1.join();
    t2.join();
}
```

## 6. 스레드 간 데이터 공유

- 전역 변수, 공유 메모리 등을 사용
- promise, future 객체 사용
- <future> 헤더

```cpp
#include <iostream>
#include <thread>
#include <future>
using namespace std;

int global = 0;

mutex m;

void f1( promise<int>& p ) {
    this_thread::sleep_for(3s);
    p.set_value(10);
}

int main()
{
    promise<int> p;
    // 미래에 결과값을 담음
    future<int> ft = p.get_future();

    thread t1(&f1, ref(p));
    cout << "wait value " << endl;
    cout << "Value : " << ft.get() << endl; // set_value가 될 때까지 대기
    
    t1.join();
}
```

## 참고
codenuri 강석민 강사 강의 내용기반으로 정리한 내용입니다.  
[코드누리](https://github.com/codenuri)  

<br>

    This is personal diary for study documents.
    Please comment if I'm wrong or missing something else 😄. 

[Top](#){: .btn .btn--primary }{: .align-right}