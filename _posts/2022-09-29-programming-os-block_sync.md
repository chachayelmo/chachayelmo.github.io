---
published: true
title:  "[OS] 블록(Block), 논블록(Non-Block), 싱크(Sync) 와 어싱크(Async) 란?"
excerpt: "프로그래밍에 대해 알아보기 5번째, 블록, 논블록, 싱크, 어싱크"

categories:
  - OS
tags:
  - [운영체제, 블록, 논블록, 싱크, 어싱크, Block, NonBlock, Sync, Async]

toc: true
toc_sticky: true
author: chachayelmo
sitemap:
  changefreq : daily
  priority : 1.0
date: 2022-09-29
last_modified_at: 2022-10-03
---

## 1. Sync

- 동기 방식은 서버에서 요청을 보냈을 때 응답이 돌아와야 다음 동작을 수행할 수 있음, A작업이 모두 진행 될 때까지 B작업은 대기
- 호출하는 함수가 호출되는 함수의 작업 완료 후 리턴을 기다리거나, 또는 호출되는 함수로부터 바로 리턴받더라도 작업 완료 여부를 호출하는 함수 스스로 계속 확인

## 2. Async

- 요청을 보냈을 때 응답상태와 상관없이 다음 동작을 수행, A작업과 B작업이 동시에 실행, 즉 Sync, Async는 호출되는 함수의 작업 완료 여부를 누가 신경쓰냐임
- 호출되는 함수에게 callback을 전달해서 호출되는 함수의 작업이 완료되면 호출되는 함수가 전달받은 callback을 실행하고, 호출하는 함수는 작업 완료 여부를 신경쓰지 않음

## 3. Blocking I/O

- 호출된 함수가 자신의 작업을 모두 끝낼때까지 제어권을 가지고 있어 호출한 함수가 대기하도록 함

## 4. NonBlocking I/O

- 호출된 함수가 바로 return해서 호출한 함수에게 제어권을 주어 다른일을 할 수 있게 함
- 즉 Blocking, NonBlocking은 호출되는 함수를 바로 리턴하냐 마냐임

![non_blocking](https://user-images.githubusercontent.com/23397039/193536310-ddd72ac4-3533-45c4-b09c-eaf47eaf9777.png){: .align-center}


## 5. NonBlocking - Sync

- 호출되는 함수는 바로 리턴하고, 호출하는 함수는 작업 완료 여부를 신경 씀
  - [future 함수](https://en.cppreference.com/w/cpp/thread/future)
- 신경 쓰는 방법:
    - 기다리거나 (Blocking)
    - 물어보거나 (NonBlocking)

![non_block_sync](https://user-images.githubusercontent.com/23397039/193536305-1ed40497-114d-4bc2-874a-8c2a7afa8126.png){: .align-center}


## 6. Blocking - Async

- 호출되는 함수가 바로 리턴하지 않고, 호출되는 함수가 바로 리턴하지 않고, 호출하는 함수는 작업 완료 여부를 신경쓰지 않음
- Node.js 와 MySQL 조합, Node.js 쪽에서 callback 지옥을 헤치면서 aynsc로 전진해와도, 결국 DB 작업 호출 시에는 MySQL에서 제공하는 드라이버를 호출하게 되는데 이 드라이버가
Blocking 방식

![blocking_async](https://user-images.githubusercontent.com/23397039/193536147-8f18a4f1-f836-41fa-aafa-9e57f3749b07.png){: .align-center}

## 7. 다른 방식의 설명
- java = sync, block
- java script = async, non-block
- 결과값 전달
- 제어권 반환 (Block, Non-block)
  - Block, non-block 은 제어권
- Sync, async 는 시간(timing)

- 제어권, 반환
  - 제어할 수 없는 대상의 처리 방법

```
fucntion 호출자(){

functionA();
functionB();
functionC();

}

function functionA() {
  return something;
}
```

### 7.1. Block
1. 제어권이 넘어감
  - 호출자() -> function()
2. functionA()에서 return이 될 때까지 호출자()는 제어권이 없음 (block)

### 7.2. Non Block
1. functionA()가 return이 되면 호출자()는 제어권을 받아옴
2. 제어권이 넘어감 호출자() -> functionA()
3. 그리고 functionA() -> 호출자() 로 바로 제어권이 넘어옴 (non-block)
4. functionA()는 따로 스레드로 돌아서 함수실행
5. functionA()의 결과값이 어떻게 가지?

### 7.3. Synchronous (동기)
- 어떤 대상들의 시간이 일치하는가?
  1. 함수a의 "끝"과 함수b의 "시작"을 맞춤
  ```
    —— 함수 a ——> “끝, 시작”—— 함수 b ——>
  ```
  2. 시간을 일치시킴
  ```
    —— a(제어권 반환) ——> “끝”
    —— b(결과값 전달) ——> “끝”
  ```

### 7.4. Asynchronous (비동기)
- 시간을 맞추지 않음
  ```
  ———— 함수 a ————>
        —— 함수 b ——>

  ——- 제어권 반환 ——>
  —————- 결과값 전달 ———>
  ```

- 순서대로 실행되지 않음
- 스레드나 프로세스가 여러개가 동작하고 있다고 생각하면 됨
- 비동기로 주어진 일을 다 수행하고 callback function을 호출
- 비동기로 실행되는 애들은 callback 함수를 task queue에 넣어서 이를 실행해주는 event loop로 실행

### 7.5. Non block & Sync
- 리눅스 시스템콜(I/O)에 해당
![process_thread.png](../../assets/images/linux_system_call.png){: .align-center}

- 동작
  1. kernel function은 끝나지 않았지만 결과값(완료되지 않음)과 제어권을 반환
  2. 대신 주기적으로 끝났는지 물어봄
  3. 그 사이에 다른 일을 수행
  4. 결과 값(완료)되면 끝

- **Note)** Java의 future라는 method (비동기/논블로킹)

### 7.6. 요약
- Block vs Non-Block
  - 제어권, 제어할 수 없는 대상을 어떻게 처리하는가?
- Synchronous vs Asynchronous
  - 시간, 대상들의 시간을 일치시키는가?

## 참고
[유투브 테크톡](https://www.youtube.com/watch?v=IdpkfygWIMk)

<br>

    This is personal diary for study documents.
    Please comment if I'm wrong or missing something else 😄. 

[Top](#){: .btn .btn--primary }{: .align-right}