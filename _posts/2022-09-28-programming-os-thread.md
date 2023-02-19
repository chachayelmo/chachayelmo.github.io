---
published: true
title:  "[OS] 스레드(Thread) 란?"
excerpt: "프로그래밍에 대해 알아보기 4번째, 스레드"

categories:
  - OS
tags:
  - [스레드, 운영체제, Programming, Thread]

toc: true
toc_sticky: true
author: chachayelmo
sitemap:
  changefreq : daily
  priority : 1.0
date: 2022-09-28
last_modified_at: 2022-09-28
---

## 1. Thread
- Light weight process 로 불림
- Process vs Thread
    1. 프로세스
        - 프로세스 간에는 각 프로세스의 데이터 접근 불가능
    2. 스레드
        - 하나의 프로세스에 여러 개의 스레드 생성 가능
        - 스레드들은 동시에 실행 가능
        - 프로세스 안에 있으므로, 프로세스 데이터 접근 가능
      ![process_thread.png](../../assets/images/process_vs_thread.png){: .align-center}
        
    
### 1.1. MultiThread
    
- software 병행 작업 처리를 위해 멀티스레드 사용
![multithread.png](../../assets/images/multithread.png){: .align-center}

### 1.2. Multi processing & Thread
- Multi tasking = 단일 CPU
- Multi processing = 다수의 CPU
- 최근 CPU는 멀티 코어를 가지므로, 스레드를 여러 개 만들어 멀티 코어로 활용도를 높임

![multiprocessing_thread.png](../../assets/images/multiprocessing_thread.png){: .align-center}

## 2. Thread 장점

- 사용자에 대한 응답성 향상
![pros_thread.png](../../assets/images/pros_thread.png){: .align-center}

- 자원 공유 효율
  - IPC 기법과 같이 프로세스간 자원 공유를 위해 번거로운 작업이 필요없음, 프로세스의 데이터 접근 가능
![process_shared_thread.png](../../assets/images/process_shared_thread.png){: width="50%" height="50%"}{: .align-center}

## 3. Thread 단점
- 스레드중 한 개만 문제가 있어도, 전체 프로세스가 영향을 받음
- 스레드를 많이 생성하면 context switching이 많이 일어나 성능 저하
  - 예: 리눅스 OS에서는 스레드를 프로세스와 같이 다룬다.
  - 스레드를 많이 생성하면 모든 스레드를 스케줄링해야 하므로, context switching 이 빈번할 수 밖에 없음
![cons_thread.png](../../assets/images/cons_thread.png){: .align-center}

## 4. Thread vs Process

| 프로세스 | 스레드 |
| --- | --- |
| 독립적 | 프로세스의 subset |
| 각각 독립적인 자원을 가짐 | 스레드는 주소영역을 공유 |
| 자신만의 주소영역을 가짐 | 주소영역을 공유 |
| 프로세스간 IPC 통신 | IPC 통신 안함 |

- PThread (Posix Thread) : 스레드 관련 표준 API

## 참고
[펀코딩](https://www.fun-coding.org/thread.html)

<br>

    This is personal diary for study documents.
    Please comment if I'm wrong or missing something else 😄. 

[Top](#){: .btn .btn--primary }{: .align-right}