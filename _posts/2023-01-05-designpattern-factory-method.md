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
last_modified_at: 2023-01-05
---

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