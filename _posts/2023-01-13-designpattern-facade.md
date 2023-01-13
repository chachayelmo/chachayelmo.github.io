---
published: true
title:  "[Design Pattern] Structural - Facade in C++"
excerpt: "다양한 디자인 패턴에 대해 알아보기"

categories:
  - Algorithm
tags:
  - [DesignPattern, Facade, Structural, Pattern]

toc: true
toc_sticky: true
 
date: 2023-01-13
last_modified_at: 2023-01-13
---

## 1. Facade(파사드)

- 건축의 파사드와 유사하게 복잡한 기본 또는 구조 코드를 마스킹하는 전면 인터페이스 역할을 하는 객체
- 단일 또는 단순화된 API 뒤에 있는 복잡한 구성 요소와의 상호 작용을 마스킹하여 소프트웨어 라이브러리의 가독성과 유용성을 개선
- 일반적인 기능에 대한 컨텍스트별 인터페이스 제공
- loosely-coupled 코드를 선호하는 monolithic 코드와 tightly-coupled 시스템의 리팩토링을 위한 시작점 역할
- 개발자는 시스템이 매무 복잡하거나 상호 의존적인 클래스가 많아 이해하기 어려운 경우 Facade 디자인 패턴을 자주 사용
- 이 패턴은 더 큰 시스템의 복잡성을 숨기고 클라이언트에 더 간단한 인터페이스를 제공

### 1.1. 의도

- 라이브러리, 프레임워크에 대한 또는 복잡한 클래스 집합에 대한 단순화된 인터페이스를 제공하는 구조적 디자인 패턴

### 1.2. 문제

- 정교한 라이브러리나 프레임워크에 속하는 광범위한 객체들의 집합으로 코드를 작동하게 만들어야 된다고 할 때
- 이러한 객체들을 모두 초기화하고, 종속성 관계를 추적하고, 올바른 순서로 메소드들을 실행하는 등의 작업을 수행해야 함
- 따라서 클래스들의 비즈니스 로직이 타사 클래스들의 구현과 밀접하게 결합하여 코드를 이애하고 유지보수하기가 어려워짐

### 1.3. 해결책

- Facade는 복잡한 하위 시스템에 대한 간단한 인터페이스를 제공함
- 하위 시스템을 직접 작업하는 것과 비교하면 Facade는 클라이언트들이 정말 중요하게 생각하는 기능들만 포함하여 제공

## 2. 구조

![image](https://user-images.githubusercontent.com/23397039/211472846-18ff4916-2e0c-4dd3-9e87-4316fd1c6a74.png){: .align-center}

- Facade
    - 하위 시스템 기능들의 특정 부분을 편리하게 접근 가능
    - 클라이언트의 요청을 어디로 보내야 하는 지와 모듈들이 어떻게 작동해야 하는 지를 알고 있음
- Additional Facade : 또 다른 클래스를 생성하여 Facade와 관련 없는 기능들로 오염시켜 복잡한 구조로 만드는 것을 방지
- Subsystem classes : 다양한 객체들로 구성, Facade의 존재를 인식하지 못함
- Client : 하위 시스템 객체들을 직접 호출하는 대신 Facade를 사용

## 3. 사용

- 복잡한 하위 시스템에 대한 제한적이지만 간단한 인터페이스가 필요할 때 사용
- 하위 시스템을 계층들로 구성하려는 경우 사용
    - 각 하위 시스템의 진입점을 퍼사드 패턴으로 정의
    - 여러 하위 시스템이 퍼사드 패턴들을 통해서만 통신하도록 함으로써 하위 시스템 간의 결합도를 줄일 수 있음

## 4. Pros and Cons

### 4.1. Pros

- 복잡한 하위 시스템에서 코드를 별도로 분리할 수 있음

### 4.2. Cons

- 앱의 모든 클래스에 결합된 전지전능한 객체가 될 가능성이 있음

## 5. 코드로 알아보기

```cpp
#include <iostream>
#include <string>

 // 서브시스템은 클라이언트에서 Facade 또는 직접 호출 가능 
class Subsystem1 {
public:
  std::string Operation1() const {
    return "Subsystem1: Ready!\n";
  }
  // ...
  std::string OperationN() const {
    return "Subsystem1: Go!\n";
  }
};

class Subsystem2 {
public:
  std::string Operation1() const {
    return "Subsystem2: Get ready!\n";
  }
  // ...
  std::string OperationZ() const {
    return "Subsystem2: Fire!\n";
  }
};

// 퍼사드 클래스는 복잡한 서브 시스템에 대한 간단한 인터페이스 제공
// 클라이언트의 요청을 적절한 서브 시스템 객체에 위임
// lifecycle 관리 담당
class Facade {
protected:
  Subsystem1 *subsystem1_;
  Subsystem2 *subsystem2_;

public:
  // 메모리 소유권을 퍼사드 클래스로 위임한 케이스
  Facade(
      Subsystem1 *subsystem1 = nullptr,
      Subsystem2 *subsystem2 = nullptr) {
    this->subsystem1_ = subsystem1 ?: new Subsystem1;
    this->subsystem2_ = subsystem2 ?: new Subsystem2;
  }
  ~Facade() {
    delete subsystem1_;
    delete subsystem2_;
  }

  // 간결하게 짜인 퍼사드 메소드
  std::string Operation() {
    std::string result = "Facade initializes subsystems:\n";
    result += this->subsystem1_->Operation1();
    result += this->subsystem2_->Operation1();
    result += "Facade orders subsystems to perform the action:\n";
    result += this->subsystem1_->OperationN();
    result += this->subsystem2_->OperationZ();
    return result;
  }
};

// 클라이언트는 퍼사드가 제공하는 인터페이스를 통해 복잡한 하위 시스템과 함께 동작
void ClientCode(Facade *facade) {
  // ...
  std::cout << facade->Operation();
  // ...
}

int main() {
  Subsystem1 *subsystem1 = new Subsystem1;
  Subsystem2 *subsystem2 = new Subsystem2;
  Facade *facade = new Facade(subsystem1,
                              subsystem2);
  ClientCode(facade);

  delete facade;

  return 0;
}
```

## 참고
[wikipedia](https://en.wikipedia.org/wiki/Facade_pattern)  
[refactoring.guru](https://refactoring.guru/design-patterns/facade)  

<br>

    This is personal diary for study documents.
    Please comment if I'm wrong or missing something else 😄. 

[Top](#){: .btn .btn--primary }{: .align-right}