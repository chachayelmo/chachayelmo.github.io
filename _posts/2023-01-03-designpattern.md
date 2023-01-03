---
published: true
title:  "[Design Pattern] 디자인 패턴"
excerpt: "다양한 디자인패턴에 대해 알아보기"

categories:
  - Algorithm
tags:
  - [DesignPattern, GoF]

toc: true
toc_sticky: true
 
date: 2023-01-03
last_modified_at: 2023-01-03
---

## 1. 디자인 패턴
- 디자인 패턴은 모듈의 세분화된 역할이나 모듈들 간의 인터페이스 구현 방식을 설계 할 때 참조할 수 있는 방법
- 설계 문제, 해결 방법과 언제 적용해야하는 지 알 수 있음
- 한 패턴에 변형을 가하거나 요구사항을 반영하여 다른 패턴으로 변형할 수 있음
- GoF(Gang of Four) 라고 불리는 사람들이 처음으로 디자인 패턴을 구체화하였음

## 2. 디자인 패턴의 분류
- 목적에 따라 3가지로 분류
  - 생성, Creational pattern
  - 구조, Structural pattern
  - 행동, Behavioral pattern
- 생성 패턴은 객체의 생성 과정에 관여, 구조 패턴은 객체의 합성에 관여, 행동 패턴은 객체가 상호작용하는 방법이나 관심사를 분리하는 방법
- 범위에 따라 분류도 가능
  - 패턴을 클래스에 적용하는지 객체에 적용하는 지 구분이 가능

## 3. 생성 패턴
- 특징
  - 객체의 생성과 관련된 패턴
  - 객체의 인스턴스 과정을 추상화하는 방법
  - 캡슐화를 통해 객체가 생성되거나 변경되어도 프로그램 구조에 영향을 받지 않음

- 클래스 : 객체를 생성하는 일부를 서브클래스가 담당
  - Factory Method
- 객체 : 객체 생성을 다른 객체에 위임
  - Abstract Factory
  - Builder
  - Prototype
  - Singleton

## 4. 구조 패턴
- 특징
  - 클래스나 객체들을 조합해 더 큰 구조로 만들 수 있게 해줌

- 클래스 : 상속을 통해 클래스나 인터페이스를 합성
  - Adapter (class)
- 객체 : 객체를 합성하는 방법
  - Adapter (object)
  - Bridge
  - Composite
  - Decorator
  - Facade
  - Flyweight
  - Proxy

## 5. 행동 패턴
- 특징
  - 클래스나 객체들이 서로 상호작용하는 방법, 교류하는 방법
  - 하나의 객체로 수행할 수 없는 작업을 여러 객체로 분배하면서 결합도를 최소화 할 수 있도록 도와줌

- 클래스 : 상속을 통해 알고리즘과 제어 흐름을 기술
  - Interpreter
  - Template Method
- 객체 : 하나의 작업을 수행하기 위해 객체 집합이 어떻게 협력하는지 기술
  - Chain of Responsibility
  - Command
  - Iterator
  - Mediator
  - Memento
  - Observer
  - State
  - Strategy
  - Visitor

## 참고
[4z7l](https://4z7l.github.io/2020/12/25/design_pattern_GoF.html)
[GoF](https://springframework.guru/gang-of-four-design-patterns/)

<br>

    This is personal diary for study documents.
    Please comment if I'm wrong or missing something else 😄. 

[Top](#){: .btn .btn--primary }{: .align-right}