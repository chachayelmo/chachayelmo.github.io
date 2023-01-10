---
published: true
title:  "[Design Pattern] Creational - Prototype pattern in C++"
excerpt: "다양한 디자인패턴에 대해 알아보기"

categories:
  - Algorithm
tags:
  - [DesignPattern, Prototype, Creational, Pattern]

toc: true
toc_sticky: true
 
date: 2023-01-06
last_modified_at: 2023-01-06
---

## 1. Prototype pattern

- 실제 제품을 만들기에 앞서 테스트를 위한 샘플 제품을 만드는 것
- 객체를 생성하는 데 비용이 많이 들고, 비슷한 객체가 이미 있는 경우에 사용되는 패턴
- 즉, 원본 객체를 새로운 객체에 복사하여 필요에 따라 수정하는 메커니즘을 제공

![image](https://user-images.githubusercontent.com/23397039/210472834-4c78775b-a6b8-4f98-94ba-948ad5d7038d.png)

- 모든 프로토타입 클래스는 concrete class를 알 수 없어도 객체를 복사할 수 있는 공통 인터페이스를 사용함
- 프로토타입 object는 full copy를 통해 다른 객체의 private field에 접근할 수 있음

## 2. 코드로 알아보기

```cpp
#include <iostream>
#include <string>
#include <unordered_map>

using std::string;

enum Type {
  PROTOTYPE_1 = 0,
  PROTOTYPE_2
};

// Prototype 인터페이스
class Prototype {
 protected:
  string prototype_name_;
  float prototype_field_;

 public:
  Prototype() {}
  Prototype(string prototype_name)
      : prototype_name_(prototype_name) {
  }
  virtual ~Prototype() {}
  virtual Prototype *Clone() const = 0;
  virtual void Method(float prototype_field) {
    this->prototype_field_ = prototype_field;
    std::cout << "Call Method from " << prototype_name_ << " with field : " << prototype_field << std::endl;
  }
};

// Concrete class
class ConcretePrototype1 : public Prototype {
 private:
  float concrete_prototype_field1_;

 public:
  ConcretePrototype1(string prototype_name, float concrete_prototype_field)
      : Prototype(prototype_name), concrete_prototype_field1_(concrete_prototype_field) {
  }

  ConcretePrototype1(const ConcretePrototype1&){
    std::cout << "copy ConcretePrototype1" << "\n";
  }

  // Clone 메소드는 ConcretePrototype1를 복사
  Prototype *Clone() const override {
    return new ConcretePrototype1(*this);
  }
};

class ConcretePrototype2 : public Prototype {
 private:
  float concrete_prototype_field2_;

 public:
  ConcretePrototype2(string prototype_name, float concrete_prototype_field)
      : Prototype(prototype_name), concrete_prototype_field2_(concrete_prototype_field) {
  }

  ConcretePrototype2(const ConcretePrototype2&){
    std::cout << "copy ConcretePrototype2" << "\n";
  }

  // ConcretePrototype2를 복사
  Prototype *Clone() const override {
    return new ConcretePrototype2(*this);
  }
};

// Prototype Factory 클래스
class PrototypeFactory {
 private:
  std::unordered_map<Type, Prototype *, std::hash<int>> prototypes_;

 public:
  PrototypeFactory() {
    prototypes_[Type::PROTOTYPE_1] = new ConcretePrototype1("PROTOTYPE_1 ", 50.f);
    prototypes_[Type::PROTOTYPE_2] = new ConcretePrototype2("PROTOTYPE_2 ", 60.f);
  }

  ~PrototypeFactory() {
    delete prototypes_[Type::PROTOTYPE_1];
    delete prototypes_[Type::PROTOTYPE_2];
  }

  Prototype *CreatePrototype(Type type) {
    // new 보단 copy를 통한 object 사용
    return prototypes_[type]->Clone();
  }
};

// Client
void Client(PrototypeFactory &prototype_factory) {
  std::cout << "Let's create a Prototype 1\n";

  Prototype *prototype = prototype_factory.CreatePrototype(Type::PROTOTYPE_1);
  prototype->Method(90);
  delete prototype;

  std::cout << "Let's create a Prototype 2 \n";

  prototype = prototype_factory.CreatePrototype(Type::PROTOTYPE_2);
  prototype->Method(10);

  delete prototype;
}

int main() {
  PrototypeFactory *prototype_factory = new PrototypeFactory();
  Client(*prototype_factory);
  delete prototype_factory;

  return 0;
}
```

## 참고
[wikipedia](https://en.wikipedia.org/wiki/Prototype_pattern)  
[refactoring.guru](https://refactoring.guru/design-patterns/prototype/cpp/example)
[newtownboy](https://velog.io/@newtownboy/%EB%94%94%EC%9E%90%EC%9D%B8%ED%8C%A8%ED%84%B4-%ED%94%84%EB%A1%9C%ED%86%A0%ED%83%80%EC%9E%85%ED%8C%A8%ED%84%B4Prototype-Pattern)

<br>

    This is personal diary for study documents.
    Please comment if I'm wrong or missing something else 😄. 

[Top](#){: .btn .btn--primary }{: .align-right}