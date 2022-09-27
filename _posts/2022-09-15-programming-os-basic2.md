---
published: true
title:  "[Programming] 프로그래밍이란? - 프로세스, 컨텍스트 스위칭"
excerpt: "프로그래밍에 대해 알아보기 2번째"

categories:
  - OS
tags:
  - [programming, process, IPC]

toc: true
toc_sticky: true
 
date: 2022-09-15
last_modified_at: 2022-09-27
---

## 1. Process 구성요소

- Code: text 코드
- Data: 변수/초기화된 데이터
- Stack: 임시 데이터(함수호출, 로컬변수 등)
- Heap: 동적으로 만들어지는 데이터 (malloc, new 등)

![process_conf.png](../../assets/images/process_conf.png)

- PC (Program Counter) + SP(Stack Pointer)
- PCB: Process Context Block
- 프로세스가 실행중인 상태를 캡쳐/구조화 시켜서 저장

## 2. Context Switching

- CPU에 실행할 프로세스를 교체하는 기술
- 실행 중지할 프로세스 정보를 해당 프로세스의 PCB에 업데이트해서 Main Memory에 저장
- 다음 실행할 프로세스 정보를 Main Memory에 있는 해당 PCB 정보(PC, SP)를 CPU의 Register에 넣고 실행

![context.png](../../assets/images/context.png)

- Dispatch : ready 상태의 프로세스를 running 상태로 바꾸는 것

## 3. Interrupt & Context Switching

- 프로세스 실행 중 인터럽트 발생
- 현 프로세스 실행 중단 (PCB 업데이트)
- 인터럽트 처리 함수 실행 (OS)
- 현 프로세스 재실행 (PCB 정보를 CPU에 Load)

![kernel_interrupt.png](../../assets/images/kernel_interrupt.png)

- 프로세스 스위칭은 실제 nano sec 단위로 일어남
    - 컨텍스트 스위칭 동작
        - 프로세스1 PCB 정보를 메인메모리에 저장
        - 프로세스2 PCB 정보를 메인메모리에서 로드함

### 3.1 Compiler
- 초기 컴퓨터 프로그램들은 어셈블리어로 작성
    - 서로 다른 CPU Architecture가 등장할 때마다 매번 똑같은 프로그램 작성
    - 어셈블리어로는 프로그램 작성 속도가 매우 떨어짐
- 컴파일러 등장
    - CPU 아키텍처에 따라 컴파일러 프로그램만 만들면 기존 코드 재작성할 필요가 없음
    그러나 어셈블리어로 작성한 코드보다는 속도가 떨어질 수 있음

## 4. IPC
### 4.1 Inter Process Communication
- 필요한 이유?
    - 성능을 높이기 위해 여러 Process 를 만들어서 동시 실행, Process 간 상태 확인 및 데이터 송수신 필요
- fork() 시스템 콜
    - Process 자신을 복사해서 새로운 프로세스로 만들 수 있음 (부모, 자식 프로세스)
    - Process를 fork()해서 여러 프로세스를 동시에 실행시킬 수 있음
        - CPU가 한 개일 때만 생각하지만, CPU안에 Core가 8개 가 되는 경우가 많고, 각 Process를 각 Core에 동시 실행 가능, 병렬 처리
- Example
    - 1 ~ 10000 까지 더하기
        - fork() 함수로 10개 process를 만들어서 1 ~ 1000, 1001 ~ 2000 .. 계산
        - 각각 더한 값을 모두 합하면, 더 빠르게 동작
        - 이때, 각 Process가 더한 값을 수집하기 위해 IPC가 필요

### 4.2 IPC 방법
    1. file
        - 간단히 다른 프로세스에 전달할 내용을 파일에 쓰고, 다른 프로세스가 파일을 읽음
        - 파일을 사용하면 실시간으로 직접 원하는 프로세스에 데이터 전달이 어려움, 왜냐면 계속 실시간으로 파일을 읽을 수 없기 때문에
    2. Message Queue
    3. Shared Memory
    4. Pipe
    5. Signal
    6. Semaphore
    7. Socket

    자세한 내용은 다음에 알아보도록 하시죠.    

### 4.3 리눅스 프로세스
    - 프로세스간 공간은 완전 분리
    - 커널 공간은 공유

![ipc_1.png](../../assets/images/ipc_1.png)
![ipc_2.png](../../assets/images/ipc_2.png)

## 참고
[펀코딩](https://www.fun-coding.org/contextswitching.html)

<br>

    This is personal diary for study documents.
    Please comment if I'm wrong or missing something else 😄. 

[Top](#){: .btn .btn--primary }{: .align-right}