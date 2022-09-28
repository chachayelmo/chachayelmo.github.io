---
published: true
title:  "[Programming] IPC 인터프로세스커뮤니케이션"
excerpt: "프로그래밍에 대해 알아보기 3번째, IPC 방법들"

categories:
  - OS
tags:
  - [programming, IPC]

toc: true
toc_sticky: true
 
date: 2022-09-27
last_modified_at: 2022-09-28
---

# IPC 방법

## 1. Pipe
- Pipe
  - 기존 파이프는 단방향 통신
  - fork()로 자식 프로세스 만들었을 때, 부모 자식간의 통신
  
  ![pipe.png](../../assets/images/pipe.png){: .align-center}
  
  - 예시
  ```cpp
  int main() {
    char buf[255];
    int fd[2], pid, nbytes;
    if ( pipe(fd) < 0 ) // pipe(fd) 로 파이프 생성
      exit(1);
    pid = fork(); // 이 함수 실행 다음 코드부터 부모/자식 프로세스로 나누어짐
    
    if (pid > 0) {  // 부모프로세스에는 pid가 0이 들어감
      write(fd[1], msg, MSGSIZE);
    }
    else {
      nbyte = read(fd[0], buf, MSGSIZE); // fd[0] 로 읽음
      printf("%d %s \n", nbytes, buf);
      exit(0);
    }
    return
  }
  ```

## 2. Message Queue

- 메시지 큐는 기본적으로 FIFO 정책으로 데이터 전송

- 메시지 큐 코드 예제
  - A process
    ```cpp
    msqid = msgget(key, msgflag); // key = 1234, msgflag는 옵션
    msgsnd(msgid, &sbuf, buf_length, IPC_NOWAIT);
    ```
  - B process
    ```cpp
    msgid = msgget(key, msgflag); // key는 동일하게 1234로 해야 해당 큐의 msgid를 얻을 수 있음
    msgrcv(msqid, &rbuf, MSGSZ, 1, 0);
    ```
### 2.1 Pipe vs Message Queue

- Message Queue
    - 부모/자식이 아니라, 어느 Process 간에라도 데이터 송수신 가능
    - 먼저 넣은 데이터가 먼저 읽힘(FIFO)
- Pipe
    - 부모/자식 프로세스간 통신
    - 단방향 or 양방향
    
- 둘 다 Kernel space의 메모리 사용
- 메모리 공간도 kernal/user 로 구분 됨
![message_queue.png](../../assets/images/message_queue.png){: .align-center}

## 3. Shared Memory

- 직접적으로 Kernel space에 메모리 공간을 만들고, 해당 공간을 변수처럼 쓰는 방식
- Message Queue 처럼 FIFO가 아니라, 해당 메모리 주소를 마치 변수처럼 접근하는 방식
- Shared Memory key를 가지고, 여러 프로세스가 접근 가능
![shared_memory.png](../../assets/images/shared_memory.png){: .align-center}

- 예시
    - 공유 메모리 생성
    
    ```cpp
    sshmid = shmget((key_t)1234, SIZE, IPC_CREAT|0666));
    shmaddr = shmat(shmid, (void *)0, 0);
    ```
    
    - 공유 메모리에 쓰기
    
    ```cpp
    strcpy((char *)shmaddr, "Linux programming");
    ```
    
    - 공유 메모리에서 읽기
    
    ```cpp
    printf("%s\n", (char *)shmaddr);
    ```

## 4. Signal

- UNIX에서 30년 이상 사용된 전통적인 기법
- 커널 또는 프로세스에서 다른 프로세스에 어떤 이벤트가 발생되었는지를 알려주는 기법
- 주요 시그널
  - SIGKILL   : 프로세스 죽이기
  - SIGALARM  : 알람 발생
  - SIGSTP    : 프로세스 멈추기
  - SIGCONT   : 멈춘 프로세스 실행
  - SIGINT    : 프로세스에 인터럽트를 보내서 프로세스를 죽이기
  - SIGSEG    : 프로세스가 다른 메모리영역을 침범

- 동작
    1. 프로그램에서 특정 시그널의 기본 동작 대신 다른 동작을 하도록 구현 가능
    2. 각 프로세스에서 시그널 처리에 대해 다음과 같은 동작 설정 가능
        1. Signal ignore
        2. Signal block (block을 푸는 순간, 해당 프로세스에서 Signal 처리)
        3. 프로그램 안에 등록된 Signal handler로 재정의한 특정 동작 수행
        4. 등록된 Signal handler가 없다면, 커널에서 기본 동작 수행
    3. 예시
        1. Signal handler 등록 및 handler 구현
            
            ```cpp
            static void signal_handler (int signo) {
              printf("Catch SIGINT!\n");
              exit (EXIT_SUCCESS);
            }
            
            int main() {
              if (signal(SIGINT, signal_handler) == SIG_ERR) {
                printf("Can't catch SIGINT!\n");
                exit(EXIT_FAILURE);
              }
              for (;;)
                pause();
            
              return 0;
            }
            ```
            
        2. Signal handler 무시
        
            ```cpp
            int main()
            {
              if (signal(SIGINT, SIG_IGN) == SIG_ERR) {
                printf("Can't catch SIGINT!\n");
                exit(EXIT_FAILURE);
              }
            
              for (;;)
                pause();
            
              return 0;
            }
            ```
        
    
### 4.1. Signal 과 Process
- PCB에서 해당 프로세스가 block 또는 처리해야 하는 Signal 관련 정보 관리
    ![signal.png](../../assets/images/signal.png){: .align-center}

## 5. Socket

- 소켓은 네트워크 통신을 위한 기술
- 기본적으로 클라이언트와 서버 등 두개의 다른 컴퓨터간의 네트워크 기반 통신을 위한 기술
![socket.png](../../assets/images/socket.png){: .align-center}

### 5.1.1 소켓과 IPC

- 소켓을 하나의 컴퓨터 안에서 두 개의 프로세스 간의 통신 기법으로 사용 가능
- 리눅스 커널 내부 네트워크 stack 예시
![computer_env.png](../../assets/images/computer_env.png){: .align-center}

## 참고
[펀코딩](https://www.fun-coding.org/ipc.html)

<br>

    This is personal diary for study documents.
    Please comment if I'm wrong or missing something else 😄. 

[Top](#){: .btn .btn--primary }{: .align-right}