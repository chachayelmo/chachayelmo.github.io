---
published: true
title:  "[C++] STL 스레드 비동기(thread asnyc)"
excerpt: "asnyc 에 대해 알아보기"

categories:
  - Cpp
tags:
  - [C++, Cpp, STL, Async]

toc: true
toc_sticky: true
author: chachayelmo
sitemap:
  changefreq : daily
  priority : 1.0
comment: true
date: 2023-02-08
last_modified_at: 2023-02-08
---

## 1. async

- 스레드를 생성하는 2가지 방법
    - thread 클래스 - low level api
    - async 함수 - high level api
- <future> 헤더
- 함수를 다른 스레드로 실행하거나 지연된 실행을 할 때 사용
- launch policy

```cpp
#include <iostream>
#include <thread>
#include <future>
using namespace std;

int f1() {
    this_thread::sleep_for(3s);
    return 10;
}

int main()
{
    // thread t1(&f1);
    // t1.join();

    future<int> ft = async( launch::async, &f1);

    cout << "main routine" << endl;

    cout << ft.get() << endl; // 스레드 종료를 대기, join의 역할
    // ft.get을 하지 않는다면?
    // future 객체의 소멸자에서 내부적으로 get()을 호출
}
```

## 2. launch policy

- launch::async : 비동기로 실행, 스레드 생성에 실패할 경우 예외 발생
- launch::deferred : 지연된 실행
- launch::async | 
launch::deferred : 스레드 생성이 가능하면 스레드로 그렇지 않으면 지연된 실행

```cpp
#include <iostream>
#include <thread>
#include <future>
using namespace std;

int f1() {
    cout << "f1 : " << this_thread::get_id() << endl;
    this_thread::sleep_for(3s);
    return 10;
}

int main()
{
    cout << "main : " << this_thread::get_id() << endl; 

    // 1. 
    // future<int> ft = async( launch::async, &f1);
    // 2. f1을 지연된 실행(get을 호출할 때)
    // future<int> ft = async( launch::deferred, &f1); // 새로운 스레드를 만드는 건 아님
    // 3.
    future<int> ft = async( launch::deferred | launch::async , &f1);

    this_thread::sleep_for(1s);

    cout << "main routine" << endl;

    cout << ft.get() << endl;
}
```

## 3. async 임시객체

```cpp
#include <iostream>
#include <thread>
#include <future>
using namespace std;

int f1() {
    this_thread::sleep_for(3s);
    cout << "f1 end" << endl;
    return 10;
}

int main()
{
    async( launch::async , &f1);  // 리턴값으로 future<int> 객체
                                  // 리턴용 임시객체

    // future<int> ft = async( launch::async , &f1);
    cout << "main routine" << endl;
    // ft.get();
}
```

## 참고
codenuri 강석민 강사 강의 내용기반으로 정리한 내용입니다.  
[코드누리](https://github.com/codenuri)  

<br>

    This is personal diary for study documents.
    Please comment if I'm wrong or missing something else 😄. 

[Top](#){: .btn .btn--primary }{: .align-right}