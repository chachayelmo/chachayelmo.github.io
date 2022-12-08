---
published: true
title:  "[Programming] 파일 디스크립터"
excerpt: "C++에 대해 알아보기, File descriptor"

categories:
  - OS
tags:
  - [Programming, Linux, FileDescriptor]

toc: true
toc_sticky: true
 
date: 2022-12-08
last_modified_at: 2022-12-08
---

## 1. 파일 디스크립터
- File Descriptor(Fd) 는 리눅스/유닉스 계열의 시스템에서 프로세스가 파일을 다룰 때 사용하는 개념
- 프로세스에서 특정 파일에 접근할 때 사용
- 일반적으로 0이 아닌 정수값을 가짐

## 2. 유닉스 시스템
- 유닉스 시스템에서 모든 것을 파일이라고 하며, 정규파일부터 디렉토리, 소켓, 파이프, 블록 디바이스 등 모든 객체들을 파일로 관리
- 유닉스 시스템에서 프로세스가 위의 파일들에 접근할 때 Fd를 이용
- 즉, 프로세스가 실행 중에 파일을 Open 하면
  - 커널은 해당 프로세스의 Fd 숫자 중 사용하지 않는 가장 작은 값을 할당
  - 그 다음 프로세스가 열려있는 파일에 System call을 이용해서 접근할 때, Fd 값을 이용해서 특정 파일을 지칭
- 프로그램이 프로세스로 메모리에서 실행될 때, Fd는 redirection(stdin, stdout, stderr) 값을 할당

## 3. Fd 확인하는 방법
1. 실행중인 프로세스 PID 확인

```
$ ps -ef | grep sshd
```

2. PID로 해당 프로세스의 Fd 정보 확인

```
$ sudo ls -trn /proc/{PID}/fd
```

## 참고
[두발로걷는개의 발자국](https://twofootdog.tistory.com/51)

<br>

    This is personal diary for study documents.
    Please comment if I'm wrong or missing something else 😄. 

[Top](#){: .btn .btn--primary }{: .align-right}