---
published: true
title:  "[Design Pattern] 데코레이터 패턴(Decorator) in C++"
excerpt: "다양한 디자인 패턴에 대해 알아보기"

categories:
  - DesignPattern
tags:
  - [데코레이터, DesignPattern, Decorator, Structural, Pattern]

toc: true
toc_sticky: true
author: chachayelmo
sitemap:
  changefreq : daily
  priority : 1.0
comment: true
date: 2023-01-12
last_modified_at: 2023-01-12
---

## 1. Decorator

- 동일한 클래스의 다른 객체 동작에 영향을 주지 않고 동적으로 개별 객체에 동작을 추가할 수 있도록 하는 디자인 패턴
- 완전히 새로운 객체를 정의하지 않고도 객체의 동작을 확장할 수 있기 때문에 Sub class를 만드는 것보다 효율적임

### 1.1. 의도

- Decorator는 객체들을 새로운 행동들을 포함한 특수 wrapper 객체들에 넣어서 행동들을 해당 객체들에 연결하는 구조적 디자인 패턴

### 1.2. 문제

- 알림 라이브러리를 만들고 있다고 가정
- 알림 lib의 목적은 다른 프로그램들이 사용자들에게 중요한 이벤트에 대해 알리는 것
    
![image](https://user-images.githubusercontent.com/23397039/211272681-39697d9c-3aab-49cb-81af-d761f4868164.png){: .align-center}

- 초기 앱 릴리즈 이후 어느 시점에서 라이브러리 사용자들이 이메일 알림보다 많은 것을 기대한다는 것을 알게 되어 아래와 같이 유형별로 자식 클래스를 구현

![image](https://user-images.githubusercontent.com/23397039/211272786-976e684a-517a-4540-8c40-d5983704bef4.png){: .align-center}

- 그런데 한 이용자가 여러 유형의 알림을 한 번에 사용하고 싶다고 하여 여러 알림 메소드를 합성한 특수한 자식 클래스를 만들었으나, 라이브러리 코드뿐만 아니라 클라이언트 코드도 엄청나게 양이 많아짐
    
![image](https://user-images.githubusercontent.com/23397039/211272844-7c0c19ba-cf0e-4253-a363-5e3512e3875e.png){: .align-center}  

### 1.3. 해결

- 객체의 동작을 변경해야 할 때 가장 먼저 고려되는 방법은 클래스의 확장이나 상속에는 몇 가지 주의사항이 있음
    - 상속은 static이라서 런타임 때 기존 객체의 행동을 변경할 수 없음
    - 자식 클래스는 하나의 부모 클래스만 가질 수 있음, 대부분 언어에서의 상속은 클래스가 동시에 여러 클래스의 행동을 상속하지 못하도록 함
- 이를 극복하기 위해 상속 대신 집합(Aggregation) or 포함(Composition)을 사용
    - Aggregation 관계에서는 한 객체가 다른 객체에 대한 참조를 갖고 일부 작업을 위임하는 반면 상속을 사용하면 객체 자체가 부모 클래스에서 행동을 상속한 후 해당 작업을 수행할 수 있음
    - 새로운 접근 방식은 헬퍼 객체를 다른 객체로 쉽게 대체하여 런타임 때 행동을 변경할 수 있음
    - 집합과 합성은 Decorator를 포함한 많은 디자인 패턴의 핵심 원칙

![image](https://user-images.githubusercontent.com/23397039/211273188-9763759d-1236-4435-aa9f-a83b54ace8d4.png){: .align-center}  

- Wrapper(래퍼) 는 패턴의 주요 아이디어를 명확하게 표현하는 Decorator 패턴의 별명임
    - 래퍼는 일부 타겟 객체와 연결할 수 있는 객체
    - 래퍼는 타겟 객체와 같은 메소드들의 집합이 포함되며, 자신이 받는 모든 요청을 타겟 객체에 위임
- Wrapper는 wrapping된 객체와 같은 인터페이스를 구현하므로 클라이언트 관점에서는 이러한 객체들은 같음

![image](https://user-images.githubusercontent.com/23397039/211273273-766cee39-2109-4a34-a0dd-3534d1de900e.png){: .align-center}  

- 앱들은 알림 Decorator들의 복잡한 스택들을 설정할 수 있음
- 스택의 마지막 Decorator는 실제로 클라이언트와 동작
- 클라이언트는 객체가 다른 객체들과 같은 인터페이스를 따르는 한 객체를 모든 사용자 지정 Decorator로 장식할 수 있음
- 예시
    - 사람이 옷을 입는 것은 Decorator 패턴을 사용하는 예
    - 나는 추울 때 스웨터를 입고, 스웨터를 입어도 춥다면 재킷을 입고, 비가 오면 비옷을 입음
    - 이 모든 옷은 기초 행동을 확장하지만, 나의 일부가 아니므로 필요하지 않을 때마다 옷을 쉽게 벗을 수 있음

## 2. 구조

![image](https://user-images.githubusercontent.com/23397039/211273317-2bc21a93-dc5e-4b90-aeab-ffafb3028cd4.png){: .align-center}  

- Component
    - 래퍼들과 래핑된 객체들 모두에 대한 공통 인터페이스 선언
- Concrete Component
    - 래핑되는 개체들의 클래스이며 기본 행동들을 정의
- Base Decorator
    - 클래스에 래핑된 객체들을 참조하기 위한 필드를 가짐
- Concrete Decorators
    - 컴포넌트들에 동적으로 추가될 수 있는 추가 행동들을 정의
- Client
    - 데코레이터들이 컴포넌트 인터페이스를 통해 객체들과 동작하는 한 여러 계층의 데코레이터들로 래핑이 가능

## 3. 사용

1. 객체들을 사용하는 코드를 훼손하지 않으면서 런타임에 추가 행동들을 객체에 할당할 수 있어야 할 때 사용
    - 비즈니스 로직을 계층으로 구성하고, 각 계층에 데코레이터를 생성하여 런타임에 이 로직의 다양한 조합들로 객체들을 구성할 수 있도록 함
2. 상속을 사용하여 객체의 행동을 확장하는 것이 이상하거나 불가능할 때 사용

## 4. Pros and Cons

### 4.1. Pros

- 새로운 자식 클래스를 만들지 않고 객체의 행동을 확장할 수 있음
- 런타임에 객체들에게 책임들을 추가/제거 할 수 있음
- 객체를 여러 데코레이터로 래핑하여 여러 행동들을 Composition 할 수 잇음
- Single Responsibility Principle, 다양한 행동들의 여러 변형들을 구현하는 monolithic 클래스를 여러 개의 작은 클래스로 나눌 수 있음

### 4.2. Cons

- 래퍼들의 스택에서 특정 래퍼를 제거하기가 어려움
- 데코레이터의 행동이 스택 내의 순서에 의존하지 않는 방식으로 구현하기 어려움

## 5. 코드로 알아보기

```cpp
#include <iostream>
#include <string>

// 컴포넌트 인터페이스
class Component {
public:
  virtual ~Component() {}
  virtual std::string Operation() const = 0;
};

// Concrete 클래스
class ConcreteComponent : public Component {
public:
  std::string Operation() const override {
    return "ConcreteComponent";
  }
};

// 다른 component들과 같은 인터페이스를 따름
// ConcreteDecorator에 대한 wrapping 인터페이스를 정의
class Decorator : public Component {
protected:
  Component* component_;

public:
  Decorator(Component* component) : component_(component) {
  }
  std::string Operation() const override {
    return this->component_->Operation();
  }
};

// 래핑된 객체를 호출하고 결과를 변경
class ConcreteDecoratorA : public Decorator {
 public:
  ConcreteDecoratorA(Component* component) : Decorator(component) {
  }
  std::string Operation() const override {
    return "ConcreteDecoratorA(" + Decorator::Operation() + ")";
  }
};

class ConcreteDecoratorB : public Decorator {
 public:
  ConcreteDecoratorB(Component* component) : Decorator(component) {
  }

  std::string Operation() const override {
    return "ConcreteDecoratorB(" + Decorator::Operation() + ")";
  }
};

// 클라이언트
void ClientCode(Component* component) {
  // ...
  std::cout << "RESULT: " << component->Operation();
  // ...
}

int main() {
  // 클라이언트에서 ConcreteComponent를 직접적으로 사용도 가능
  Component* simple = new ConcreteComponent;
  std::cout << "Client: I've got a simple component:\n";
  ClientCode(simple);
  std::cout << "\n\n";

  // 데코레이터로 래핑된 component도 사용 가능
  Component* decorator1 = new ConcreteDecoratorA(simple);
  Component* decorator2 = new ConcreteDecoratorB(decorator1);
  std::cout << "Client: Now I've got a decorated component:\n";
  ClientCode(decorator2);
  std::cout << "\n";

  delete simple;
  delete decorator1;
  delete decorator2;

  return 0;
}
```

## 참고
[wikipedia](https://en.wikipedia.org/wiki/Decorator_pattern)  
[refactoring.guru](https://refactoring.guru/design-patterns/decorator)  

<br>

    This is personal diary for study documents.
    Please comment if I'm wrong or missing something else 😄. 

[Top](#){: .btn .btn--primary }{: .align-right}