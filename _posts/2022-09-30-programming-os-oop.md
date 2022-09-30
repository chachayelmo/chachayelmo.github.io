---
published: true
title:  "[Programming] 객체지향이란? OOP"
excerpt: "C++에 대해 알아보기, this"

categories:
  - os
tags:
  - [os, oop]

toc: true
toc_sticky: true
 
date: 2022-09-30
last_modified_at: 2022-09-30
---

## 1. 절차지향 프로그래밍
- 절차지향언어는 개체를 순차적으로 처리하여 프로그램 전체가 유기적으로 연결되어야 함
- ( 파스칼, 코볼, 포트란, C언어 등)

## 2. 객체지향 프로그래밍

- 객체 단위로 데이터와 함수를 하나로 묶어서 쓰는 언어

### 2.1. 상속성
  - 상위클래스에 근거하여 새로운 클래스와 행위를 정의

### 2.2. 다형성
  - 상속을 통해 기능을 확장하거나 변경하는 것
  - 코드의 재사용
  - 코드길이 감소가 되어 유지보수가 용이

### 2.3. 오버라이딩
  - 상위클래스에서 만들어진 메소드를 재창조하여 사용

### 2.4. 오버로딩
  - 메소드 이름은 같지만 매개변수의 갯수나 데이터 타입이 다르게 쓰이는 경우

### 2.5. 추상화
  - 큰 틀의 클래스를 구현하고 공통적인 요소나 필수적인 요소가 들어가는 것

### 2.6. 캡슐화
  - 특정한 목적을 위해 필요한 변수나 메소드를 하나로 묶는 것

### 2.7. 정보은닉
  - private : 해당 클래스만 접근 가능
  - protected: 해당 클래스 or 해당 클래스를 상속받은 클래스에서 접근 가능
  - public: 어떤 클래스라도 접근 가능

### 2.8. 다른 특징들
- Static method: self(this)를 갖고 있지 않고 데이터영역 메모리에 올라감, 객체없이 사용 가능하다는 의미
- Instance method: 호출된 객체에서 접근 가능(클래스 멤버 함수)

| 인터페이스 | 추상클래스 |
| --- | --- |
| 클래스가 아님 | 클래스 |
| 클래스와 관련이 없음 | 클래스와 관련이 있음 (Base 클래스로 사용) |
| 한 개의 클래스에 여러개 사용 가능 | 한 개의 클래스에 여러 개 사용 불가능 |
| 구현 객체의 같은 동작을 보장하기 위한 목적 | 상속 받아서 기능을 확장시키는 데 목적 |
- Composition(포함)
    - 다른 클래스의 일부 기능을 그대로 이용하고 싶으나, 전체 기능 상속은 피하고 싶을 때 사용
    - 상속 관계가 복잡할 경우, 코드 이해가 어려운 경우가 많음
    - composition = 명시적 선언, 상속 = 암시적 선언

## 3. S.O.L.I.D

### 3.1. SRP(Single Responsibility Principle)
  - 단일 책임 원칙: 클래스는 단 한 개의 책임을 가져야 함
  - 예시
      - class StudentScoreAndCourceManager() → 나쁜 예
      - class ScoreManager, CourseManager→ 변경

### 3.2. OCP(Open Closed Principle)
  - 개방 - 폐쇄 원칙: 확장에는 열려있고, 변경에는 닫혀있어야 함
  - method override
  - 행동 구현은 부모클래스의 자식 클래스에서 재정의해서 사용
  - 부모클래스는 수정할 필요 없고(변경에 닫힘), 자식클래스에서 재정의 (확장에 대한 개방)

### 3.3. LSP(Liskov Substitusion Principle)
  - 리스코프 치환 원칙: 자식 클래스는 언제나 자신의 부모클래스와 교체할 수 있어야 함
  - 갤럭시 is a kind of 스마트폰
      - 스마트폰 → 통화, 인터넷, 앱 다운 기능
      - 갤럭시 → 통화, 인터넷, 앱 다운 기능

### 3.4. ISP(Interface Segregation Principle)
  - 인터페이스 분리 원칙: 클래스에서 상관없는 메소드는 분리해야 함

### 3.5. DIP(Dependency Inversion Principle)
  - 의존성 역전 법칙: 부모클래스는 자식클래스의 구현에 의존하면 안됨
  - 예시
      
      ```python
      # 나쁜 예
      class BubbleSort
      class SortManager:
        def __init__(self):
          self.sort_method = BubbleSort() # 의존적
      
      # 좋은 예
      class SortManager:
        def __init__(self, sort_method): # (DI)의존성을 주입시킨다고 하는 것
          self.set_sort_method(sort_method)
        def set_sort_method(self, sort_method):
          self.sort_method = sort_method
        def begin_sort(self):
          self.sort_method.sort() # 하부 클래스가 바뀌더라도, 동일한 코드 활용 가능토록 인터페이스화
      ```
        
- 추가적으로 리팩토링을 위한 중복코드 제거, 최적화는 제일 나중에 할 것

## 참고
[잔재미코딩](https://www.fun-coding.org/PL&OOP1-2.html)

<br>

    This is personal diary for study documents.
    Please comment if I'm wrong or missing something else 😄. 

[Top](#){: .btn .btn--primary }{: .align-right}