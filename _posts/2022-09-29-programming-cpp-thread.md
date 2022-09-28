---
published: true
title:  "[Programming] C++ thread"
excerpt: "C++에 대해 알아보기, thread 함수"

categories:
  - C++
tags:
  - [C++, thread]

toc: true
toc_sticky: true
 
date: 2022-09-29
last_modified_at: 2022-09-29
---

## 1. 필요한 용어들
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

## 2 Block
1. 제어권이 넘어감
  - 호출자() -> function()
2. functionA()에서 return이 될 때까지 호출자()는 제어권이 없음 (block)

### 2.1 Non Block
1. functionA()가 return이 되면 호출자()는 제어권을 받아옴
2. 제어권이 넘어감 호출자() -> functionA()
3. 그리고 functionA() -> 호출자() 로 바로 제어권이 넘어옴 (non-block)
4. functionA()는 따로 스레드로 돌아서 함수실행
5. functionA()의 결과값이 어떻게 가지?

### 2.2 Synchronous (동기)
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

### 2.3 Asynchronous (비동기)
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

## 3. Non block & Sync
- 리눅스 시스템콜(I/O)에 해당
![process_thread.png](../../assets/images/linux_system_call.png){: .align-center}

- 동작
  1. kernel function은 끝나지 않았지만 결과값(완료되지 않음)과 제어권을 반환
  2. 대신 주기적으로 끝났는지 물어봄
  3. 그 사이에 다른 일을 수행
  4. 결과 값(완료)되면 끝

- **Note)** Java의 future라는 method (비동기/논블로킹)

## 4. 요약
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