---
published: true
title:  "[Programming] 프로그래밍이란?"
excerpt: "프로그래밍에 대한 기초에 대해 알아보기"

categories:
  - OS
tags:
  - [programming, os]

toc: true
toc_sticky: true
 
date: 2022-09-14
last_modified_at: 2022-09-27
---


## 1. 프로그래밍 언어란?

인간과 컴퓨터간의 의사소통, 컴퓨터는 기계어만 이해 가능

- 기계어란? 0, 1 로 된 이진수 형태로 CPU가 이해할 수 있는 코드
- CPU가 수행할 수 있는 명령
    - instruction set(명령어 집합) = 더하기, 빼기, 데이터 저장, 불러오기, if, return 등 OP code
- 저급언어
    - 기계어, 어셈블리어 (ex. ADD A B)
- 고급언어
    - 하드웨어의 기술적 요소를 몰라도 작성 및 수정 가능 (ex., y = x + z)
    - C, JAVA, PYTHON 등
- 컴파일러
    - 고급언어로 작성된 코드를 한번에 기계어로 변환하는 프로그램
    - C, JAVA
- 인터프리터
    - 고급언어로 작성된 코드를 한줄씩 기계어로 변환하는 프로그램
    - Python, PHP, Ruby 등


## 2. 운영체제

- 종류: OS, Windows, Mac, UNIX
    ![os.png](../../assets/images/os.png){: .align-center}
- UNIX:
    - UNIX OS, LINUX OS
    - 역할:
        - 시스템 자원 관리
        - 사용자와 컴퓨터간의 communication 지원
        - 응용프로그램 제어
    - 시분할 시스템: 다중 사용자를 지원하고 컴퓨터 응답 시간을 최소화
    - 멀티태스킹: 단일 CPU에서 여러 응용프로그램을 병렬 실행을 가능하게 함, 사용자가 느끼기에 여러 프로그램이 동시에 실행되는 것 처럼 보이는 것
    ![time_system.png](../../assets/images/time_system.png){: .align-center}
    - UNIX: 현대 운영체제의 기본 기술을 모두 포함한 최초의 운영체제
    - 멀티태스킹, 시분할 시스템, 멀티 프로그래밍
    

## 3. Shell
    
사용자가 운영체제 기능과 서비스를 조작할 수 있도록 인터페이스 제공하는 프로그램 (CLI, GUI)

- Interface 제공
    - API: 함수로 제공 ex.,) open()
    - Library : c Lib
    - System Call
        - 시스템 콜 또는 시스템 호출 인터페이스
        - 운영체제가 운영체제의 각 기능을 사용할 수 있도록 시스템 콜이라는 명령 또는 함수 제공
        - API 내부에는 시스템콜을 호출하는 형태로 만들어지는 경우가 대부분


## 4. 내가 운영체제를 만든다면?
    
1. 운영체제를 개발 (Kernel)
2. System call 개발
3. C API 개발
4. Shell 프로그램 개발
5. 응용 프로그램 개발


## 5. System call
    
POSIX API: Portable Operating System Interface
- 서로 다른 UNIX OS의 **공통 API** 를 정리하여 이식성 높은 유닉스 응용프로그램을 개발하기 위한 목적
- 윈도우 API

![system_call.png](../../assets/images/system_call.png){: .align-center}


## 참고
[펀코딩](https://www.fun-coding.org/PL&OOP1-1.html)

<br>

    This is personal diary for study documents.
    Please comment if I'm wrong or missing something else 😄. 

[Top](#){: .btn .btn--primary }{: .align-right}