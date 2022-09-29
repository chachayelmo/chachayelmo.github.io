---
published: true
title:  "[Programming] C++ thread"
excerpt: "C++에 대해 알아보기, thread 함수"

categories:
  - cpp
tags:
  - [C++, cpp, thread]

toc: true
toc_sticky: true
 
date: 2022-09-30
last_modified_at: 2022-09-30
---

## 1. std::thread
- c++ 표준 스레드 라이브러리
- <thread> 헤더 파일에 포함
- OS API의 스레드 사용보다 사용하기 쉬움

## 2. 스레드 방법
### 2.1. 스레드 종료까지 대기
- **join()** 함수를 사용하여 스레드가 종료할 때까지 대기
- join을 사용하면 blocking

### 2.2. 스레드 식별자
- **get_id()**를 통해 해당 스레드의 식별자를 얻을 수 있음

```cpp
//cpp
std::thread Thread1;
Thread1.get_id()

std::this_thread::get_id()
```

### 2.3. 스레드 object와 커널 스레드 분리
- **detach** 함수를 사용하면 thread object와 thread의 연결고리를 떼어냄
- detach 이후에는 thread object는 thread를 제어할 수 없음

### 2.4. 일시 중지와 양보
- **sleep_for, sleep_until**을 사용하면 스레드를 일시중지 시킬 수 있음
  - sleep_for : 지정한 시간동안
  - sleep_until : 지정시간까지
- **yield** 를 사용하여 자신(스레드)의 활동을 포기하고 다른 스레드에게 양보

```cpp
//cpp
std::this_thread::yield();
```

## 3. 공유 자원 사용하기
- 멀티스레드 프로그래밍에서는 공유 자원 관리가 가장 큰 문제
  - 너무 빡빡하게 관리하면 성능 하락
  - 너무 느슨하게 관리하면 시한 폭탄 동작
- **mutex**를 사용하여 공유 리소스를 관리하는 것이 일반적임
  - mutex는 Windows에 구현에서는 critical section (임계 영역)을 사용

### 3.1. 공유자원 사용할 수 있는지 조사
- 스레드 A가 mutex의 lock을 호출 했을 때, 이미 스레드 B에서 lock을 호출했다면 A는 B가 lock을 풀어줄 때까지 대기한다
- 스레드A는 다른 일을 하고 싶어도 할 수가 없음 ( 이럴 때 **try_lock()**을 사용 )
- 다른 스레드가 먼저 락을 걸었다면 대기하지 않고 즉시 false를 반환
- 반대로 true를 반환한 경우 공유 리소스의 소유권을 가짐

### 3.2. 자동으로 lock 풀기
- 실수 lock 호출 후 unlock을 안했으면 데드락 상황에 빠짐 or 코드 실행 중 예외가 발생하여 lock을 풀지 못하는 경우
- 이런 문제를 풀기 위해 **lock_guard** 라는 유틸리티 클래스를 사용
- scope를 벗어날 때 자동으로 unlock 을 호출

```cpp
// cpp, 클래스 생성 후 특정한 시기에 스레드를 만들어서 동작시킴
#include <mutex>
#include <thread>
#include <iostream>

int main(){
  std::mutex mtx_lock;

  std::thread Threads1([&]()
  {
    for (int i = 0 ; i < 5; ++i)
    {
      std::lock_guard<std::mutex> guard(mtx_lock);
      std::cout << "Thread1 Num : " << i << " " << std::this_thread::get_id() <<std::endl;
      // mtx_lock.unlock();
    }
  });

  Threads1.join();
}
```

### 3.3. 반복하여 lock 걸기

```cpp
// cpp
std::recursive_mutext m_mtx; // 사용
```

### 3.4. 딱 한번만 실행
- 멀티스레드 환경에서 프로그램 실행 중에서 단 한번만의 코드 실행이 필요할 때

```cpp
//cpp
std::once_flag p_flag;
void Test()
{
	// something
}

int main()
{
	for ( ... )
	{
		std::call_once(p_flag, Test);
	}
}

```

## 참고
[모두의 코드](https://modoocode.com/269)

<br>

    This is personal diary for study documents.
    Please comment if I'm wrong or missing something else 😄. 

[Top](#){: .btn .btn--primary }{: .align-right}