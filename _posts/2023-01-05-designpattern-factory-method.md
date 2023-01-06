---
published: true
title:  "[Design Pattern] Creational - Factory Method pattern in C++"
excerpt: "다양한 디자인패턴에 대해 알아보기"

categories:
  - Algorithm
tags:
  - [DesignPattern, FactoryMethod, Creational, Pattern]

toc: true
toc_sticky: true
 
date: 2023-01-05
last_modified_at: 2023-01-06
---

## 1. Factory method pattern

- 새로운 클래스가 추가되어도 기존 코드의 변경없이 확장하기 위한 디자인 패턴
- 객체를 생성할 때 어떤 클래스의 인스턴스를 만들지를 서브클래스에게 위임

### 1.1. 의도

- 부모 클래스에서 객체들을 생성할 수 있는 인터페이스를 제공하지만, 자식 클래스들이 생성될 객체들의 유형을 변경할 수 있도록 하는 생성 패턴

### 1.2. 문제

- 물균 관리 앱을 개발하고 있다고 가정하면, 앱의 첫 번째 버전은 트럭운송만 처리할 수 있어 대부분 코드가 트럭(Truck) 클래스에 있을 때
- 선박(Ship)과 관련된 클래스를 추가하려면 전체 코드 베이스를 변경해야 함
- 차후 또 다른 교통수단을 추가하려면 아마도 전체 코드 베이스를 다시 변경해야 됨

### 1.3. 해결책

- 팩토리 메소드 패턴은 객체 생성 호출들을 팩토리 메소드에서 호출하도록 대체
- 객체들은 new를 통해 생성되지만 팩토리 메소스 내에서 호출됨

![Untitled](https://user-images.githubusercontent.com/23397039/210953543-f046479e-8003-4ed0-9459-2f87b912b711.png)

- 약간의 제한으로 자식 클래스들은 다른 유형의 제품들을 해당 제품들이 공통 기초 클래스 또는 공통 인터페이스가 있는 경우에만 return이 가능

![Untitled](https://user-images.githubusercontent.com/23397039/210953653-3ac083be-e495-4c76-b4bf-2d8c727ea5ea.png)

## 2. 구조

![image](https://user-images.githubusercontent.com/23397039/210953958-9975d549-3a72-49ec-bcc1-895bf3d0b300.png)

- Creator : 새로운 Product 객체들을 반환하는 팩토리 메소드를 선언, 팩토리 메소드의 return 은 제품 인터페이스와 일치해야 함
- Product : 인터페이스를 선언하여 생성자와 자식 클래스들이 생성할 수 있는 객체의 공통된 부분
- ConcreteCreator : Creator의 팩토리 메소드를 오버라이드하여 각각의 제품을 return 하도록 함
- ConcreteProduct : 인터페이스의 구현부

## 3. 사용

1. 팩토리 메소드는 우리의 코드가 함께 동작해야 하는 객체들의 정확한 유형들과 의존관계들을 미리 모르는 경우에 사용
    - Product 생성 코드와 실제 사용하는 코드를 분리, 그러면 Product 생성자 코드를 나머지 코드와 독립적으로 확장하기가 쉬워짐
2. Library 또는 framework 의 사용자들에게 내부 컴포넌트들을 확장하는 방법을 제공하고 싶을 때 사용
    - 상속은 Library 나 framework의 default 행동을 확장하는 가장 쉬운 방법임
3. 기존 객체들을 매번 재구축하는 대신 이들을 재사용하여 system resource 를 절약하고 싶을 때 사용
    - 코드의 중복사용을 줄일 수 있음
    

## 4. 장단점

- 장점
    - Creator와 Product들이 tight coupling 되는 것을 방지할 수 있음
    - Single Responsibility Principle, Product 생성 코드를 한 곳으로 모을 수 있음
    - Open/Closed Principle, 기존 client 코드를 유지하면서 새로운 타입의 Product를 추가할 수 있음
- 단점
    - 패턴을 구현하기 위해 많은 서브클래스를 도입해야 하므로 코드가 더 복잡해질 수 있음

## 5. 코드로 알아보기

```cpp
#include <iostream>
#include <string>

// Product 인터페이스
class Product {
 public:
  virtual ~Product() {}
  virtual std::string Operation() const = 0;
};

class ConcreteProduct1 : public Product {
public:
  std::string Operation() const override {
    return "{Result of the ConcreteProduct1}";
  }
};
class ConcreteProduct2 : public Product {
public:
  std::string Operation() const override {
    return "{Result of the ConcreteProduct2}";
  }
};

// Creator
class Creator {
public:
  virtual ~Creator(){};
  // 순수가상함수로 FactoryMethod를 만듬, 즉 서브클래스에게 이 함수를 위임
  virtual Product* FactoryMethod() const = 0;

  std::string SomeOperation() const {

    // Creator에서 함수를 구현했지만 서브클래스의 FactoryMethod() 을 콜
    Product* product = this->FactoryMethod();

    // 결과 확인
    std::string result = "SomeOperation() : " + product->Operation();
    delete product;
    return result;
  }
};

// ConcreteCreator 구현
class ConcreteCreator1 : public Creator {
public:
  Product* FactoryMethod() const override {
    return new ConcreteProduct1();
  }
};

// ConcreteCreator 구현
class ConcreteCreator2 : public Creator {
public:
  Product* FactoryMethod() const override {
    return new ConcreteProduct2();
  }
};

// Client
void ClientCode(const Creator& creator) {
  // 클라이언트 입장에서는 어떤 creator인지 알 필요가 없음
  std::cout << "Client: I'm not aware of the creator's class, but it still works.\n"
            << creator.SomeOperation() << std::endl;
}

int main() {
  std::cout << "App: Launched with the ConcreteCreator1.\n";
  Creator* creator = new ConcreteCreator1();
  ClientCode(*creator);
  std::cout << std::endl;
  std::cout << "App: Launched with the ConcreteCreator2.\n";
  Creator* creator2 = new ConcreteCreator2();
  ClientCode(*creator2);

  delete creator;
  delete creator2;
  return 0;
}
```

## 1. Factory method pattern

- 새로운 클래스가 추가되어도 기존 코드의 변경없이 확장하기 위한 디자인 패턴
- 객체를 생성할 때 어떤 클래스의 인스턴스를 만들지를 서브클래스에게 위임

![image](https://user-images.githubusercontent.com/23397039/210472577-ce2965e8-d340-4a6a-9e23-9dd4f2dbd9b2.png)

## 2. 코드로 알아보기

```cpp
#include <iostream>
#include <string>

// Product 인터페이스
class Product {
 public:
  virtual ~Product() {}
  virtual std::string Operation() const = 0;
};

class ConcreteProduct1 : public Product {
public:
  std::string Operation() const override {
    return "{Result of the ConcreteProduct1}";
  }
};
class ConcreteProduct2 : public Product {
public:
  std::string Operation() const override {
    return "{Result of the ConcreteProduct2}";
  }
};

// Creator
class Creator {
public:
  virtual ~Creator(){};
  // 순수가상함수로 FactoryMethod를 만듬, 즉 서브클래스에게 이 함수를 위임
  virtual Product* FactoryMethod() const = 0;

  std::string SomeOperation() const {

    // Creator에서 함수를 구현했지만 서브클래스의 FactoryMethod() 을 콜
    Product* product = this->FactoryMethod();

    // 결과 확인
    std::string result = "SomeOperation() : " + product->Operation();
    delete product;
    return result;
  }
};

// ConcreteCreator 구현
class ConcreteCreator1 : public Creator {
public:
  Product* FactoryMethod() const override {
    return new ConcreteProduct1();
  }
};

// ConcreteCreator 구현
class ConcreteCreator2 : public Creator {
public:
  Product* FactoryMethod() const override {
    return new ConcreteProduct2();
  }
};

// Client
void ClientCode(const Creator& creator) {
  // 클라이언트 입장에서는 어떤 creator인지 알 필요가 없음
  std::cout << "Client: I'm not aware of the creator's class, but it still works.\n"
            << creator.SomeOperation() << std::endl;
}

int main() {
  std::cout << "App: Launched with the ConcreteCreator1.\n";
  Creator* creator = new ConcreteCreator1();
  ClientCode(*creator);
  std::cout << std::endl;
  std::cout << "App: Launched with the ConcreteCreator2.\n";
  Creator* creator2 = new ConcreteCreator2();
  ClientCode(*creator2);

  delete creator;
  delete creator2;
  return 0;
}
```

## 참고
[wikipedia](https://en.wikipedia.org/wiki/Factory_method_pattern)  
[refactoring.guru](https://refactoring.guru/design-patterns/factory-method/cpp/example)

<br>

    This is personal diary for study documents.
    Please comment if I'm wrong or missing something else 😄. 

[Top](#){: .btn .btn--primary }{: .align-right}