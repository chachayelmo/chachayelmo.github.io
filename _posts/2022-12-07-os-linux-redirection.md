---
published: true
title:  "[OS] Linux 리다이렉션(redirection)"
excerpt: "프로그래밍에 대해 알아보기, redirection"

categories:
  - OS
tags:
  - [리눅스, 리다이렉션, Programming, Linux, Redirection]

toc: true
toc_sticky: true
author: chachayelmo
sitemap:
  changefreq : daily
  priority : 1.0
comment: true
date: 2022-12-08
last_modified_at: 2022-12-08
---

## 1. Redirection
- 리눅스 쉘은 항상 아래와 같이 표준 입출력을 "파일 형태"로 열고 있음
  - stdin  (키보드)
  - stdout (모니터)
  - stderr (모니터)

- 프로그램은 연산 결과를 출력 장치 (파일, 모니터 등) 으로 내보내는 데, 출력되는 데이터를 임의로 다른 장치로 보내는 것을 redirection이라고 함

## 2. 사용법
- 표준입출력은 파일 형태로 열려 있음 ( 리눅스에서 열려 있는 파일은 File Descriptor(Fd)를 할당 받음)
  - stdin   = 0
  - stdout  = 1
  - stderr  = 2

- 위와 같이 숫자에 해당하는 Fd를 할당받기 때문에 redirection을 할 수 있음

```
1. > file  // stdout 을 파일로 redirection, 파일이 없으면 새로 만들고 있으면 덮어씌움
2. >> file // stdout 을 파일로 redirection, 파일이 없으면 새로 만들고 있으면 파일의 끝에 덧붙임
3. 2>&1    // stderr를 stdout으로 redirection, stderr 도 stdout으로 보낼 수 있음
4. < file  // 파일로부터 stdin을 받도록 redirection
```

## 참고
[Peter의 우아한 프로그래밍](https://gracefulprograming.tistory.com/100)

<br>

    This is personal diary for study documents.
    Please comment if I'm wrong or missing something else 😄. 

[Top](#){: .btn .btn--primary }{: .align-right}