---
published: true
title:  "[Programming] system call 함수"
excerpt: "프로그래밍에 대해 알아보기, system_call"

categories:
  - OS
tags:
  - [Programming, Linux, SystemCall]

toc: true
toc_sticky: true
 
date: 2022-12-09
last_modified_at: 2022-12-09
---

## 1. fork
- 프로세스를 생성하고자 할 때 사용하는 system 함수
- fork 함수를 호출하는 프로세스는 부모 프로세스가 되고, 새롭게 생성되는 프로세스는 자식 프로세스가 됨
- 자식 프로세스는 부모 프로세스의 메모리를 그대로 복하여 가지게 됨

```cpp
#include <stdio.h>
#include <unistd.h>
#include <sys/types.h>
 
int main() {
     
  pid_t pid;
    
  int x;
  x = 0;
    
  pid = fork();
    
  if(pid > 0) {  // 부모 프로세스
    x = 1;
    printf("부모 PID : %ld,  x : %d , pid : %d\n",(long)getpid(), x, pid);
  }
  else if(pid == 0){  // 자식 프로세스
    x = 2;
    printf("자식 PID : %ld,  x : %d\n",(long)getpid(), x);
  }
  else {  // fork 실패
    printf("fork Fail! \n");
    return -1;
  }
    
  return 0;
}
```

![image](https://user-images.githubusercontent.com/23397039/206148626-cca2d581-d08a-4268-8525-ed9e11360707.png)

## 2. exec
- system command에 해당하는 명령어를 수행하는 함수
- 종류

```cpp
#include <unistd.h>
// path를 실행하며, arg0 ~ argn을 인자로 전달
int execl(const char *path, const char *arg0, ... , const char *argn, (char *)0); 

// path를 실행하며, argv 를 인자로 전달
int execv(const char *path, char *const argv[]);                                 

// path를 실행하며, arg0 ~ argn을 인자로 전달 및 envp에 새로운 환경 변수 설정 가능
int execle(const char *path, const char *arg0, ... , const char *argn, (char *)0, char *const envp[]);

// file을 실행하며 arg0 ~ argn을 인자로 전달, 파일은 이 함수를 호출한 프로세스의 환경 변수 PATH에 정의된 경로를 찾음
int execve(const char *file, const char *arg0, ... , const char *argn, (char *)0);

// file을 실행하며 argv를 인자로 전달
int execvp(const char *file, char *const argv[]);
```

## 3. select
- 다중 입출력으로 프로그램에서 여러 개의 파일을 작업하고자 할 때 사용하는 함수
- block/async 입출력 모델 중 하나

- 예를 들어 A, B, C, D 4개의 파일을 다루는 작업을 하고 싶다고 가정할 때
  1. A 파일 작업
  2. A 파일 작업 마침
  3. B 파일 작업
  4. B 파일 작업하다가 block 상태에 빠짐

- 예를 들어 read() 함수를 호출했는데 읽을 파일이 없다면 프로그램을 읽을 내용이 생길 때까지 block 됨
- 솔루션으로 파일을 순서대로 작업할 게 아니라 작업할 준비가 된 파일에 대해서만 작업을 하면 됨, 그것에 대한 시초가 select 시스템콜

```cpp
#include <sys/select.h>
 
int select(int n,
    fd_set* readfds,
    fd_set* writefds,
    fd_set* exceptfds,
    struct timeval* timeout);
    
FD_CLR(int fd, fd_set* set);
FD_ISSET(int fd, fd_set* set);
FD_SET(int fd, fd_set* set);
FD_ZERO(fd_set* set);
```

- select는 Fd가 입출력을 수행할 준비가 되거나 timeout 변수에 정해진 시간이 경과할 때까지만 block이 됨
  - readfds   : 읽기가 가능한 지 감시
  - writhefds : 쓰기가 가능한 지 감시
  - exceptfds : 예외가 발생했거나 대역을 넘어선 소켓이 존재하는 지 감시

```cpp
#include <stdio.h>
#include <sys/time.h>
#include <sys/types.h>
#include <unistd.h>
 
#define TIMEOUT 5
#define BUF_LEN 1024
 
int main() {
  struct timeval tv;
  fd_set readfds;
  int ret;
  
  // 표준 입력에서 입력을 기다리기 위한 준비
  FD_ZERO(&readfds);
  FD_SET(STDIN_FILENO, &readfds);
  
  // select가 5초 동안 기다리도록 timeval 구조체를 설정
  tv.tv_sec = TIMEOUT;
  tv.tv_usec = 0;
  
  // select() 시스템콜을 이용해 입력을 기다림.
  ret = select(STDIN_FILENO + 1, &readfds, NULL, NULL, &tv);
  
  if (ret == -1) {
    perror("select");
    return 1;
  }
  else if (!ret){
    printf("%d seconds elapsed.\n", TIMEOUT);
    return 0;
  }
  
  // select() 시스템콜이 정수를 반환하면 block없이 즉시 읽기가 가능
  if (FD_ISSET(STDIN_FILENO, &readfds)) {
    char buf[BUF_LEN + 1];
    int len;
    
    // '블록(block)'없이 읽기가 가능
    len = read(STDIN_FILENO, buf, BUF_LEN);
    if (len == -1) return 1;
    if (len) {
      buf[len] = '\0';
      printf("read: %s\n", buf);
    }
    
    return 0;
  }
}
```

## 참고
[개발여행기](https://codetravel.tistory.com/23)
[wikipedia_fork](https://en.wikipedia.org/wiki/Fork_(system_call))
[bbolmin](https://bbolmin.tistory.com/35)
[팡투루야](https://pangtrue.tistory.com/31)

<br>

    This is personal diary for study documents.
    Please comment if I'm wrong or missing something else 😄. 

[Top](#){: .btn .btn--primary }{: .align-right}