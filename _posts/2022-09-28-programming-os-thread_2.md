---
published: true
title:  "[Programming] Thread - 스레드 동기화"
excerpt: "프로그래밍에 대해 알아보기 5번째, 스레드 동기화"

categories:
  - OS
tags:
  - [Programming, Thread, Sync]

toc: true
toc_sticky: true
 
date: 2022-09-28
last_modified_at: 2022-09-28
---

## 1. Thread Synchronization

- 동기화
  - 작업들 사이에 실행 시기를 맞추는 것
  - 여러 스레드가 동일한 자원(데이터) 접근 시 동기화 이슈 발생
  - 동일 자원을 여러 스레드가 동시 수정 시 각 스레드 결과에 영향을 줌

## 2. 동기화 이슈 해결방안
- Mutual exclusion (상호 배제)
  - 스레드는 프로세스 모든 데이터를 접근할 수 있으므로 여러 스레드가 변경하는 공유 변수에 대해 Exclusive access 필요
  - 어느 한 스레드가 공유 변수를 갱신하는 동안 다른 스레드가 동시에 접근하지 못하게 막음
  - 임계 자원: critical resource
  - 임계 영역: critical section

```python
# Python
lock.acquire()
for i in range(1000):
  g_count += 1
lock.release()
```
- Semaphore
  - Mutex 와 세마포어
  - ciritical section 에 대한 접근을 막기 위해 locking 메커니즘 필요
    - Mutex (binary semaphore): 임계 구역에 하나의 스레드만 들어갈 수 있음
    - Semaphore: 임계구역에 여러 스레드가 들어갈 수 있음, counter를 두어서 동시에 리소스에 접근할 수 있는 스레드 수를 제어

### 2.1. Semaphore
- S: 세마포어 값 (초기 값만큼 여러 프로세스가 동시 임계 영역 접근 가능)
- P: 검사 (임계영역에 들어갈 때)
    - S 값이 1 이상이면, 임계 영역 진입 후, S값 1 차감(S값이 0이면 대기)
- V: 증가 (임계영역에서 나올 때)
    - S 값을 1 더하고, 임계 영역을 나옴

  ```cpp
  P(S) : wait(S) {
    while (S <= 0) { // 대기
      S--; // 다른 프로세스 접근 제한
    }
  }

  V(S) : signal(S) {
    S++; // 다른 프로세스 접근 허용
  }
  ```

- 바쁜 대기
  - wait() 는 S가 0이라면 임계영역에 들어가기 위해 반복문 수행
  
  ```cpp
  P(S) : wait(S) {
    while (S <= 0) { // 바쁜 대기
      S--; // 다른 프로세스 접근 제한
    }
  }
  ```

- 대기 큐
    - S가 음수일 경우, 바쁜 대기 대신, 대기큐에 넣음
    
    ```cpp
    wait(S) {
      S->count--;
      if (S->count < 0) {
        add this process to S->queue;
        wakeup(P)
      }
    }
    ```
    
- wakeup() 함수를 통해 대기큐에 있는 프로세스 재실행

  ```cpp
  signal(S) {
    S->count--;
    if (S->count < 0) {
      remove a process P from S->queue;
      wakeup(P)
    }
  }
  ```

#### 2.1.1 주요 함수
- sem_open()
    - 세마포어 생성
- sem_wait()
    - 임계영역 접근 전, 세마포어를 잠그고(lock), 세마포어가 잠겨있다면 풀릴 때까지 대기(wait)
- sem_post()
    - 공유자원에 대한 접근이 끝났을 떄, 세마포어 잠금을 해제

## 3. 스레드 교착상태와 기아상태
- Thread deadlock & starvation
  - 프로그래밍은 근본적으로 중단없이 코드 실행
  - 중단은 대부분 loop로 표현
  - loop는 CPU에 부하를 걸리게 함

### 3.1. 교착상태(Deadlock)

- 무한 대기 상태, 두개 이상의 작업이 서로 상대방의 작업이 끝나기만들 기다리고 있을 때
- 발생조건
    - 상호배제(Mutual exclusion)
        - 프로세스들이 필요로 하는 자원에 대해 배타적인 통제권을 요구
    - 점유대기(Hold and wait)
        - 프로세스가 할당된 자원을 가진 상태에서 다른 자원을 기다림
    - 비선점(No preemption)
        - 프로세스가 어떤 자원의 사용을 끝낼 때까지 그 자원을 뺏을 수 없음
    - 순환대기(Circular wait)
        - 각 프로세스는 순환적으로 다음 프로세스가 요구하는 자원을 가짐
    
![deadlock.png](../../assets/images/deadlock.png){: .align-center}

## 참고
[펀코딩](https://www.fun-coding.org/thread.html)

<br>

    This is personal diary for study documents.
    Please comment if I'm wrong or missing something else 😄. 

[Top](#){: .btn .btn--primary }{: .align-right}