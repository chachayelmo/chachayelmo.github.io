---
published: true
title:  "[Design Pattern] 브릿지 패턴(Bridge) in C++"
excerpt: "다양한 디자인패턴에 대해 알아보기"

categories:
  - Algorithm
tags:
  - [브릿지, DesignPattern, Bridge, Structural, Pattern]

toc: true
toc_sticky: true
author: chachayelmo
sitemap:
  changefreq : daily
  priority : 1.0
comment: true
date: 2023-01-10
last_modified_at: 2023-01-10
---

## 1. Bridge

- 비즈니스 로직이나 대규모 클래스를 독립적으로 개발할 수 있는 별도의 클래스 계층으로 나누는 것
- 구현에서 추상화를 분리하여 각자 독립적으로 변형이 가능하고 확장이 가능하도록 함

### 1.1. 의도

- 브릿지는 대규모 클래스 또는 밀접하게 관련된 클래스들의 집합을 두 개의 개별 계층구조(추상화 및 구현)로 나눈 후 각각 독립적으로 개발할 수 있도록 하는 구조적 디자인 패턴

### 1.2. 문제

- 원과 직사각형이 한 쌍의 자식 클래스들이 있는 모양 클래스가 있다고 가정
- 이 클래스 계층 구조를 확장하여 색상을 도입하기 위해 빨간색, 파란색 모양의 자식클래스들을 만들 계획이면, 이미 2개의 자식클래스가 있으므로 아래와 같이 4개의 클래스 조합을 만들어야 함

![image](https://user-images.githubusercontent.com/23397039/211250838-7bf743d8-af38-4e99-8a42-7e815a761810.png){: .align-center}

- 새로운 모양이나 색상이 추가할 때마다 기하급수적으로 늘게 됨
- 따라서 코드는 점점 복잡해짐

### 1.3. 해결책

- 모양과 색상의 2가지 독립적인 부분에서 모양 클래스들을 확장하려고 하기 때문에 발생
- 클래스 상속과 관련된 General한 문제임
- 브릿지 패턴은 상속에서 객체 composition으로 바꿔서 이를 해결
- 이는 독립적인 부분들 중 하나를 별도의 클래스 계층구조로 추출하여 각 클래스들이 한 클래스 내에서 모든 상태와 행동들을 갖는 대신 새계층 구조의 객체를 참조하도록 함
    
![image](https://user-images.githubusercontent.com/23397039/211250868-838a332c-f1be-4aec-8a56-60b326383d70.png){: .align-center}

- 위의 예시로 보면 모양과 색상 관련해서 각각의 클래스로 출출하여 모양 클래스가 색상 객체들 중 하나를 가리키는 필드로 받게 함
- 이제 모양클래스는 연결된 색상 관련 객체에 모든 색상 관련 작업을 위임할 수 있음
- 이 참조는 모양과 색상 클래스들 사이에서 브릿지 역할을 함

### 1.4. 추상화와 구현

- 추상화 (인터페이스)는 일부 개체에 대한 상위 레이어로서 자체적으로 작업을 수행해서는 안되며,
- 작업들은 구현 레이어(플랫폼)에 위임해야 됨
- 이는 프로그래밍 언어의 인터페이스나 추상 클래스를 의미하는 것이 아님
- 예시
    - 앱에서 추상화는 그래픽 사용자 인터페이스이며 구현은 그래픽 사용자 인터페이스 레이어가 사용자와 상호작용하여 결과로 호출하는 배경(API)임
    - 이러한 앱은 두 가지 독립적인 방향으로 확장이 가능
        - 다른 여러가지의 그래픽 사용자 인터페이스를 가짐(예를 들어 일반 고객, 관리자용 인터페이스 등)
        - 여러 다른 API를 지원(Mac, Windows, Linux 등)
    - 크로스 플랫폼 앱을 구성하는 방법 중 하나
        - 추상화 : 앱의 그래픽 사용자 인터페이스
        - 구현 : 운영 체제의 API
        
![image](https://user-images.githubusercontent.com/23397039/211250942-a0325902-7a86-44e7-814c-2fb172aab555.png){: .align-center}

  - 추상화 객체는 앱의 드러나는 모습을 제어하고 연결된 구현 객체에 실제 작업들을 위임
  - 서로 다른 구현들은 공통 인터페이스를 따르는 한 상호 호환이 가능
  - 이에 따라 같은 그래픽 사용자 인터페이스는 리눅스와 윈도우에 동시에 동작이 가능

## 2. 구조

![image](https://user-images.githubusercontent.com/23397039/211251003-737f8f1f-9a21-4a1f-ba5c-c606245ffcf2.png){: .align-center}

- Abstraction
  - 상위 수준의 제어를 제공하며 Implementation 객체에서 실제 작업을 수행
- Implementation
  - 모든 Concrete Implementation 객체들의 공통적인 인터페이스를 선언하며 추상화는 여기에 선언된 메소드들을 통해서만 소통이 가능
- Concrete Implementation
  - 플랫폼별 맞충형 코드가 포함
- Refined Abstraction
  - Abstraction에서 인터페이스를 확장하거나 변형이 가능
- Client
  - 추상화 객체를 구현 객체들 중 하나와 연결하는 역할을 해야 됨

## 3. 사용

1. 어떤 기능의 여러 변형을 가진 monolithic 클래스로 나누려고 할 때 사용(예를 들어 클래스가 다양한 데이터베이스 서버들과 동작하는 경우)
    - 문제
        - monolithic 클래스가 커질수록 동작 방식을 파악하기 어려워지고 변경하는데 오랜 시간이 걸림
    - 해결
        - 따라서 브릿지 패턴을 사용하여 monolithic 클래스를 여러 클래스 계층구조로 나눠서 각각의 클래스들은 독립적으로 변경할 수 있음
        - 코드의 유지관리를 단순화 및 기존 코드 손상을 최소화
2. 여러 독립 차원에서 클래스를 확장해야 할 때 사용
    - 각 차원에 대해 별도의 클래스 계층구조를 추출하는 것이 좋음
    - 원래 클래스는 모든 작업을 자체적으로 수행하는 대신 추출된 계층구조들에 속한 객체들에 관련 작업들을 위임
3. 런타임에 구현을 전환할 수 있어야 할 때에 사용
    - 추상화 내부의 구현 객체를 바꿀 수 있으며, 그렇게 하기 위해 필드에 새로운 값을 할당하기만 하면 됨

## 4. Pros and Cons

### 4.1. Pros

- 플랫폼 독립적인 앱을 만들 수 있음
- 클라이언트 코드는 추상화와 함께 동작하며 플랫폼의 세부 정보에 노출되지 않음
- Open/Closed Principle, 새로운 추상화와 구현을 서로 독립적으로 도입 가능
- Single Responsibility Principle, 추상화 레벨의 상위 수준의 로직과 구현레벨의 플랫폼 세부 정보에 집중이 가능

### 4.2. Cons

- 응집력 있는 클래스에 패턴을 적용하여 코드를 더 복잡하게 만들 수 있음

## 2. 코드로 알아보기

```cpp
#include <iostream>
#include <string>

// 구현의 인터페이스
class Implementation {
public:
  virtual ~Implementation() {}
  // 순수가상함수로 서브클래스에 구현
  virtual std::string OperationImplementation() const = 0;
};

// 서로 다른 플랫폼 A
class ConcreteImplementationA : public Implementation {
public:
  std::string OperationImplementation() const override {
    return "ConcreteImplementationA: Here's the result on the platform A.\n";
  }
};
// 플랫폼 B
class ConcreteImplementationB : public Implementation {
public:
  std::string OperationImplementation() const override {
    return "ConcreteImplementationB: Here's the result on the platform B.\n";
  }
};

// 제어를 위한 추상화 인터페이스
class Abstraction {
protected:
  // implementation을 필드로 가짐
  Implementation* implementation_;

public:
  // 생성자에서 Implementation을 가지고 객체 생성
  Abstraction(Implementation* implementation) : implementation_(implementation) {
  }

  virtual ~Abstraction() {}

  virtual std::string Operation() const {
    // 동작에서 implementation 의 함수를 사용
    return "Abstraction: Base operation with:\n" +
           this->implementation_->OperationImplementation();
  }
};

// Implementation을 그대로 사용하며 추상화 인터페이스를 확장하는 것도 가능 
class ExtendedAbstraction : public Abstraction {
public:
  ExtendedAbstraction(Implementation* implementation) : Abstraction(implementation) {
  }
  std::string Operation() const override {
    return "ExtendedAbstraction: Extended operation with:\n" +
           this->implementation_->OperationImplementation();
  }
};

// 클라이언트는 추상화 객체만으로 핸들링
void ClientCode(const Abstraction& abstraction) {
  // ...
  std::cout << abstraction.Operation();
  // ...
}

int main() {
  // 플랫폼 A
  Implementation* implementation = new ConcreteImplementationA;
  Abstraction* abstraction = new Abstraction(implementation);
  ClientCode(*abstraction);
  std::cout << std::endl;
  delete implementation;
  delete abstraction;

  // 플랫폼 B
  implementation = new ConcreteImplementationB;
  abstraction = new ExtendedAbstraction(implementation);
  ClientCode(*abstraction);

  delete implementation;
  delete abstraction;

  return 0;
}
```

## 참고
[wikipedia](https://en.wikipedia.org/wiki/Bridge_pattern)  
[refactoring.guru](https://refactoring.guru/design-patterns/bridge)  

<br>

    This is personal diary for study documents.
    Please comment if I'm wrong or missing something else 😄. 

[Top](#){: .btn .btn--primary }{: .align-right}