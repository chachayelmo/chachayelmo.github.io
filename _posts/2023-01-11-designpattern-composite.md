---
published: true
title:  "[Design Pattern] Structural - Composite in C++"
excerpt: "다양한 디자인 패턴에 대해 알아보기"

categories:
  - Algorithm
tags:
  - [DesignPattern, Composite, Structural, Pattern]

toc: true
toc_sticky: true
 
date: 2023-01-11
last_modified_at: 2023-01-11
---

## 1. Composite

- Object Tree라고도 불림
- 동일한 유형 객체의 단일 인스턴스와 동일한 방법으로 처리되는 객체 그룹
- 부분-전체 계층을 트리구조로 구성
- 이 패턴으로 구현하면 클라이언트가 개별 객체와 구성을 균일하게 처리할 수 있음

### 1.1. 의도

- Composite 패턴은 객체들을 트리 구조들로 구성한 후, 이러한 구조들과 개별 객체들처럼 작업할 수 있도록 하는 구조 패턴

### 1.2. 문제

- 앱의 핵심 모델이 트리로 표현될 수 있을 때만 사용해야 함!
- 예를 들어 제품과 상자라는 두 가지 유형의 객체들이 있다고 했을 때,
- 상자에는 여러 개의 제품과 여러 개의 작은 상자들이 포함될 수 있고, 작은 상자에도 이러한 제품들과 상자들 등을 더 담을 수 있음
- 주문 시스템을 만들기로 가정했을 때 주문의 총가격을 계산하려면?

![image](https://user-images.githubusercontent.com/23397039/211258805-c712790e-9cd9-44c3-9a31-783ad962f2cf.png){: .align-center}

- 모든 상자를 열어본 후 내부 모든 제품을 보고 가격의 합계를 계산하는 것인데 현실 세계에서는 이러한 접근 방법이 가능하나,
- 프로그램에서 이 작업은 덧셈 루프를 실행하는 것만큼 간단하지 않음, 왜냐하면 덧셈 루프를 실행하기 위해 진행 중인 제품들 및 상자들의 클래스들, 상자의 중첩 및 기타 세부 사항들을 미리 알고 있어야 하기 때문

### 1.3. 해결

- Composite 패턴은 총가격을 계산하는 메소드를 선언하는 공통 인터페이스를 통해 제품들 및 상자들 클래스들과 작업을 제안
- 메소드는 제품의 경우 단순히 제품 가격을 반환하며, 상자의 경우 상자에 포함된 각 항목을 살펴보고 가격을 확인한 뒤 해당 상자의 총가격을 반환

![image](https://user-images.githubusercontent.com/23397039/211258843-fd3cd077-f2cd-4604-96cf-a410795b94a0.png){: .align-center}

- Composite 패턴은 객체 트리의 모든 컴포넌트에 대해 재귀적으로 행동을 실행

## 2. 구조

![image](https://user-images.githubusercontent.com/23397039/211258876-ba54d272-231d-49ca-af2b-b01507eeece5.png){: .align-center}

- Component : 인터페이스로 트리의 단순/복잡한 요소의 공통적인 작업을 설명
- Leaf : 트리의 기본 요소이며 더이상 하위 요소가 없음, 실제 작업들을 수행
- Composite
    - Component 인터페이스를 통해서만 Leaf 들과 함께 동작하며 하위 요소들의 클래스를 알지 못함
    - 요청을 받으면 작업을 Leaf에 위임하고 중간 결과들을 처리한 다음 최종 결과를 클라이언트에게 반환
- Client : Component 인터페이스를 통해 모든 요소들과 동작

## 3. 사용

- 트리와 같은 객체 구조를 구현해야 할 때 사용
    - 공통 인터페이스를 공유하는 2 가지 기본 요소인 Leaf와 Composite를 제공
- 클라이언트 코드가 Leaf와 Composite를 모두 균일하게 처리하고 싶을 때 사용
    - 패턴에 의해 정의된 모든 요소들은 공통 인터페이스를 공유하며, 이 인터페이스를 사용하는 클라이언트는 동작하는 객체들을 걱정할 필요가 없음

## 4. Pros and Cons

### 4.1. Pros

- 다형성과 재귀를 사용으로 인해 복잡한 트리구조를 편리하게 작업할 수 있음
- Open/Closed Principle, 기존 코드의 손상없이 트리와 함께 동작하는 새로운 요소를 도입할 수 있음

### 4.2. Cons

- 기능이 너무 다른 클래스에 대해 공통 인터페이스를 제공하기 어려울 수 있음
- 특정 시나리오에서는 컴포넌트 인터페이스를 지나치게 일반화하여 이해하기 어렵게 만들 수 있음

## 5. 코드로 알아보기

```cpp
#include <algorithm>
#include <iostream>
#include <list>
#include <string>

// 컴포넌트 인터페이스
class Component {
protected:
  Component *parent_;

public:
  virtual ~Component() {}
  // 선택적으로 base Component에서 함수를 기본 구현 제공 가능
  void SetParent(Component *parent) {
    this->parent_ = parent;
  }
  Component *GetParent() const {
    return this->parent_;
  }

  // 경우에 따라 child에서 해야할 작업을 정의하는 것이 좋을 수도 있음
  // 이 경우, Concrete component 클래스를 클라이언트 코드에 노출할 필요가 없어짐
  // 단점은 이러한 메소드가 leaf-level component에 대해 비어있음
  virtual void Add(Component *component) {}
  virtual void Remove(Component *component) {}

  virtual bool IsComposite() const {
    return false;
  }

  // 행동을 Concrete class에 위임
  virtual std::string Operation() const = 0;
};

// Composition의 마지막 객체
class Leaf : public Component {
public:
  std::string Operation() const override {
    return "Leaf";
  }
};

// 실제 작업을 자식에게 위임하고 자식들로부터 결과를 취합
class Composite : public Component {
protected:
  std::list<Component *> children_;

public:
  void Add(Component *component) override {
    this->children_.push_back(component);
    component->SetParent(this);
  }

  void Remove(Component *component) override {
    children_.remove(component);
    component->SetParent(nullptr);
  }
  bool IsComposite() const override {
    return true;
  }

  // 재귀로 자식들을 모두 수행하고 결과를 합산함
  std::string Operation() const override {
    std::string result;
    for (const Component *c : children_) {
      if (c == children_.back()) {
        result += c->Operation();
      } else {
        result += c->Operation() + "+";
      }
    }
    return "Branch(" + result + ")";
  }
};

// 클라이언트 코드
void ClientCode(Component *component) {
  // ...
  std::cout << "RESULT: " << component->Operation();
  // ...
}

void ClientCode2(Component *component1, Component *component2) {
  // ...
  if (component1->IsComposite()) {
    component1->Add(component2);
  }
  std::cout << "RESULT: " << component1->Operation();
}

int main() {
  Component *simple = new Leaf;
  std::cout << "Client: I've got a simple component:\n";
  ClientCode(simple);
  std::cout << "\n\n";

  Component *tree = new Composite;
  Component *branch1 = new Composite;

  Component *leaf_1 = new Leaf;
  Component *leaf_2 = new Leaf;
  Component *leaf_3 = new Leaf;
  branch1->Add(leaf_1);
  branch1->Add(leaf_2);
  Component *branch2 = new Composite;
  branch2->Add(leaf_3);
  tree->Add(branch1);
  tree->Add(branch2);
  std::cout << "Client: Now I've got a composite tree:\n";
  ClientCode(tree);
  std::cout << "\n\n";

  std::cout << "Client: I don't need to check the components classes even when managing the tree:\n";
  ClientCode2(tree, simple);
  std::cout << "\n";

  delete simple;
  delete tree;
  delete branch1;
  delete branch2;
  delete leaf_1;
  delete leaf_2;
  delete leaf_3;

  return 0;
}
```

## 참고
[wikipedia](https://en.wikipedia.org/wiki/Composite_pattern)  
[refactoring.guru](https://refactoring.guru/design-patterns/composite)  

<br>

    This is personal diary for study documents.
    Please comment if I'm wrong or missing something else 😄. 

[Top](#){: .btn .btn--primary }{: .align-right}