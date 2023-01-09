---
published: true
title:  "[Design Pattern] Creational - Abstract Factory pattern in C++"
excerpt: "다양한 디자인패턴에 대해 알아보기"

categories:
  - Algorithm
tags:
  - [DesignPattern, AbstractFactory, Creational, Pattern]

toc: true
toc_sticky: true
 
date: 2023-01-04
last_modified_at: 2023-01-04
---

### 1. Abstract Factory

- 구체적인 Concrete Class를 지정하지 않고 공통된 테마를 가진 개별 팩토리를 캡슐화하는 방법

### 1.1. 의도

- 추상 팩토리는 관련 객체들의 Concrete 클래스들을 지정하지 않고도 관련 객체들의 모음을 생성할 수 있도록 하는 생성 패턴

### 1.2. 문제

- 가구 판매장을 위한 프로그램을 만들고 있다고 가정
- 아래와 같은 제품들을 클래스로 구성
- 해당 제품군에서 여러가지 변형이 가능, 예를 들어 의자 + 소파 + 커피 테이블 에서 종류별로 아르데코, 빅토리안, 현대식으로 제공이 가능해야 됨
- 또한 카탈로그를 자주 변경해야되기 때문에 새로운 제품 및 종류를 추가할 때마다 기존 코드를 변경하는 것을 피해야 됨

![image](https://user-images.githubusercontent.com/23397039/210967569-6af956f1-eda5-43da-b458-a644e53dfb6f.png){: .align-center}

### 1.3. 해결책

- 각 제품군을 개별적인 인터페이스로 선언 및 모든 변형을 인터페이스에 따르도록 해야 됨

![image](https://user-images.githubusercontent.com/23397039/210967628-7decbbf8-5680-4c03-9c7a-af8df04da202.png){: .align-center}

- 그 다음 추상 팩토리 패턴을 선언하여 제품군 내의 모든 개별 제품들의 생성 메소드들이 목록화되어 있는 인터페이스를 만듬

![image](https://user-images.githubusercontent.com/23397039/210967696-00279c43-5939-4d7b-b399-efabfd857e8a.png){: .align-center}

- 제품의 변형은 추상 팩토리 인터페이스를 기반으로 별도의 팩토리 클래스를 만들어서 특정 종류의 제품을 return 하는 클래스를 가짐
- 클라이언트 코드는 자신에 해당하는 추상 인터페이스를 통해 팩토리들과 제품들과 함께 동작

## 2. 구조

![image](https://user-images.githubusercontent.com/23397039/210967757-0591952f-7eea-41e3-9e25-30ebc1a73a65.png){: .align-center}

- Abstract Product : Product를 위한 Interface 선언
- Concrete Product : Abstract Product 를 다양한 구현들
- Abstract Factory : 각각 Abstract Product들을 생성하기 위한 여러 메소드들의 집합을 선언
- Concrete Factory : Abstract Factory의 생성 메소드를 구현

## 3. 사용

- 관련된 제품군을 다양한 묶음으로 동작해야 하지만 해당 제품들이 Concrete 클래스 들에 의존하고 싶지 않을 때 사용, 이는 확장성에 좋음
    - 추상팩토리는 제품군의 각 클래스에서부터 객체들을 생성할 수 있는 인터페이스를 제공
    - 인터페이스를 통해 개체를 생성하면 이미 생성된 제품과 일치하지 않는 잘못된 제품 생성에 대해 걱정할 필요가 없음

## 4. Pros and Cons

### 4.1. Pros
- 팩토리로부터 만든 제품이 서로 호환되는지 확인 가능
- 제품과 클라이언트 코드 간의 tight couplong을 피할 수 있음
- Single Responsibility Principle
- Open/Closed Principle
### 4.2. Cons
- 패턴과 함께 새로운 인터페이스와 클래스가 도입되기 때문에 코드가 복잡해질 수 있음

## 5. 코드로 알아보기

```cpp
#include <iostream>
#include <string>

// 추상 프로덕트
class AbstractProductA {
public:
  virtual ~AbstractProductA(){};
  virtual std::string UsefulFunctionA() const = 0;
};

// Concrete 프로덕트
class ConcreteProductA1 : public AbstractProductA {
public:
  std::string UsefulFunctionA() const override {
    return "The result of the product A1.";
  }
};

class ConcreteProductA2 : public AbstractProductA {
  std::string UsefulFunctionA() const override {
    return "The result of the product A2.";
  }
};

class AbstractProductB {
public:
  virtual ~AbstractProductB(){};
  virtual std::string UsefulFunctionB() const = 0;

  // 프로덕트A 제품들하고 호환 가능하게 하는 함수
  virtual std::string AnotherUsefulFunctionB(const AbstractProductA &collaborator) const = 0;
};

class ConcreteProductB1 : public AbstractProductB {
 public:
  std::string UsefulFunctionB() const override {
    return "The result of the product B1.";
  }

  // 프로덕트A 제품들하고 호환 가능하게 하는 함수 구현부
  std::string AnotherUsefulFunctionB(const AbstractProductA &collaborator) const override {
    const std::string result = collaborator.UsefulFunctionA();
    return "The result of the B1 collaborating with ( " + result + " )";
  }
};

class ConcreteProductB2 : public AbstractProductB {
 public:
  std::string UsefulFunctionB() const override {
    return "The result of the product B2.";
  }

  std::string AnotherUsefulFunctionB(const AbstractProductA &collaborator) const override {
    const std::string result = collaborator.UsefulFunctionA();
    return "The result of the B2 collaborating with ( " + result + " )";
  }
};

// 추상팩토리 인터페이스
class AbstractFactory {
 public:
  virtual AbstractProductA *CreateProductA() const = 0;
  virtual AbstractProductB *CreateProductB() const = 0;
};

// Concrete 팩토리, 추상팩토리 인터페이스에 대한 구현
class ConcreteFactory1 : public AbstractFactory {
 public:
  AbstractProductA *CreateProductA() const override {
    return new ConcreteProductA1();
  }
  AbstractProductB *CreateProductB() const override {
    return new ConcreteProductB1();
  }
};

// Concrete 팩토리, 추상팩토리 인터페이스에 대한 구현
class ConcreteFactory2 : public AbstractFactory {
 public:
  AbstractProductA *CreateProductA() const override {
    return new ConcreteProductA2();
  }
  AbstractProductB *CreateProductB() const override {
    return new ConcreteProductB2();
  }
};

// 클라이언트 코드
void ClientCode(const AbstractFactory &factory) {
  const AbstractProductA *product_a = factory.CreateProductA();
  const AbstractProductB *product_b = factory.CreateProductB();
  std::cout << product_b->UsefulFunctionB() << "\n";
  std::cout << product_b->AnotherUsefulFunctionB(*product_a) << "\n";
  delete product_a;
  delete product_b;
}

int main() {
  std::cout << "Client: Testing client code with the first factory type:\n";
  ConcreteFactory1 *f1 = new ConcreteFactory1();
  ClientCode(*f1);
  delete f1;
  std::cout << std::endl;
  std::cout << "Client: Testing the same client code with the second factory type:\n";
  ConcreteFactory2 *f2 = new ConcreteFactory2();
  ClientCode(*f2);
  delete f2;
  return 0;
}
```

## 참고
[wikipedia](https://en.wikipedia.org/wiki/Abstract_factory_pattern)  
[refactoring.guru](https://refactoring.guru/design-patterns/abstract-factory)

<br>

    This is personal diary for study documents.
    Please comment if I'm wrong or missing something else 😄. 

[Top](#){: .btn .btn--primary }{: .align-right}