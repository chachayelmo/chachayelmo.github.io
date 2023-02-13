---
published: true
title:  "[Design Pattern] Chain of Responsibility in C++"
excerpt: "다양한 디자인 패턴에 대해 알아보기"

categories:
  - Algorithm
tags:
  - [DesignPattern, Chain, Responsibility, Pattern]

toc: true
toc_sticky: true
 
date: 2023-02-13
last_modified_at: 2023-02-13
---

## 1. Chain of Responsibility

### 1.1. 의도

- handler들의 Chain에 따라 요청을 전달할 수 있게 해주는 행동 디자인 패턴
- 각 handler 는 요청을 받으면 요청을 처리할 지 아니면 Chain의 다음 handler로 전달할 지를 결정

### 1.2. 문제

- 온라인 주문 시스템을 개발하고 있다고 가정할 때, 인증된 사용자들만 주문을 생성할 수 있도록 시스템에 대한 접근을 제한
- 관리 권한이 있는 사용자들에게는 모든 주문에 대한 전체 접근 권한을 부여
- 순차 검사들에 대한 구현이 필요

![image](https://user-images.githubusercontent.com/23397039/218373156-e78691ee-3832-489d-b5ef-1f5cfd5fcbe0.png){: .align-center}

- 새로운 기능이 추가될 수록 코드가 복잡해지고 하나의 검사 코드를 바꾸면 다른 검사 코드에도 영향을 받기도 함

![image](https://user-images.githubusercontent.com/23397039/218373190-5d530422-b3d1-4657-b943-1b695b1927e6.png){: .align-center}

### 1.3. 해결책

- 특정 행동들을 handler라는 독립적인 실행 객체들로 변환
- 각각의 검사는 검사를 수행하는 단일 메소드가 있는 자체 클래스로 추출
- handler들을 chain으로 연결하여 각각 다음 handler에 대한 참조를 저장하는 필드를 가짐
- handler는 요청을 처리하는 것 외에도 chain에 따라 요청을 다른 handler로 이동 가능

![image](https://user-images.githubusercontent.com/23397039/218373232-d9f25472-5d67-474c-a84a-ec347e4a5160.png){: .align-center}

- 정식적인 접근 방법은 handler가 요청을 받으면 handler가 요청을 처리할 수 있는지 판단
- 처리가 가능한 경우, handler는 이 요청을 더 이상 전달하지 않음
- 요청을 처리하는 handler는 하나뿐이거나 아무 handler도 요청을 처리하지 않음
- 이는 GUI 내에서 요소들의 stack에서 이벤트들을 처리할 때 일반적임

![image](https://user-images.githubusercontent.com/23397039/218373272-0214b41f-840a-4c4e-b84c-012c96617db8.png){: .align-center}

- 모든 handler 클래스들이 같은 인터페이스를 구현하는 것이 매우 중요
- 각 handler는 execute 메소드가 있는 다음 handler에만 신경을 써야 함
- 이렇게 하면 다양한 handler들을 사용하여 코드를 handler들의 concrete class들에 결합하지 않고도 런타임에 chain들을 구성할 수 있음

## 2. 구조

![image](https://user-images.githubusercontent.com/23397039/218373073-51c74208-622e-478a-b7cf-a20cb132410a.png){: .align-center}

- Handler : 공통적인 인터페이스를 선언, 일반적으로 여기에는 요청을 처리하기 위한 단일 메소드만 포함하지만 때로는 chain의 다음 handler를 셋팅하기 위한 메소드를 포함할 수 있음
- BaseHandler : optional이며 모든 handler에 대한 공통적인 코드를 구현할 수 있음
- ConcreteHandlers : 요청을 처리하기 위한 실제 코드가 구현
- Client : 로직에 따라 chain들을 한번만 구성하거나 동적으로 구성할 수 있음

## 3. 사용

- 이 패턴은 프로그램이 다양한 방식으로 다양한 종류의 요청에 대한 순서를 미리 알 수 없는 경우 사용
    - 여러 handler를 하나의 chain으로 연결할 수 있도록 해주고, 요청을 받으면 각 handler에게 요청을 처리할 수 잇는지 질문하여 모든 handler가 요청을 처리할 기회를 얻게 해줌
- 특정 순서로 여러 handler를 실행해야 할 때 사용
- handler들의 집합과 순서가 런타임에 변경되어야 할 때 사용

## 4. Pros and Cons

- Pros
    - 요청의 처리 순서를 제어할 수 있음
    - Single Responsibility Principle
    - Open/Closed Principle
- Cons
    - 일부 요청들은 처리되지 않을 가능성이 있음

## 5. 코드로 알아보기

```cpp
#include <iostream>
#include <string>
#include <vector>

// Handler 인터페이스
class Handler {
 public:
  virtual Handler *SetNext(Handler *handler) = 0;
  virtual std::string Execute(std::string request) = 0;
};

// Base Handler : 공통된 동작을 구현
class AbstractHandler : public Handler {
  /**
   * @var Handler
   */
 private:
  Handler *next_handler_;

 public:
  AbstractHandler() : next_handler_(nullptr) {
  }
  Handler *SetNext(Handler *handler) override {
    this->next_handler_ = handler;
    // link된 handler로 이동
    return handler;
  }
  std::string Execute(std::string request) override {
    if (this->next_handler_) {
      return this->next_handler_->Execute(request);
    }

    return {};
  }
};

// concrete handler 1
class MonkeyHandler : public AbstractHandler {
 public:
  std::string Execute(std::string request) override {
    // 요청이 관련되지 않으면 Base Handler 메소드를 다시 호출하여 다음 handler로 넘김
    if (request == "Banana") {
      return "Monkey: I'll eat the " + request + ".\n";
    } else {
      return AbstractHandler::Execute(request);
    }
  }
};
// concrete handler 2
class SquirrelHandler : public AbstractHandler {
 public:
  std::string Execute(std::string request) override {
    if (request == "Nut") {
      return "Squirrel: I'll eat the " + request + ".\n";
    } else {
      return AbstractHandler::Execute(request);
    }
  }
};
// concrete handler 3
class DogHandler : public AbstractHandler {
 public:
  std::string Execute(std::string request) override {
    if (request == "MeatBall") {
      return "Dog: I'll eat the " + request + ".\n";
    } else {
      return AbstractHandler::Execute(request);
    }
  }
};

// 클라이언트 코드
void ClientCode(Handler &handler) {
  std::vector<std::string> food = {"Nut", "Banana", "Cup of coffee"};
  for (const std::string &f : food) {
    std::cout << "Client: Who wants a " << f << "?\n";
    const std::string result = handler.Execute(f);
    if (!result.empty()) {
      std::cout << "  " << result;
    } else {
      std::cout << "  " << f << " was left untouched.\n";
    }
  }
}

int main() {
  // chain을 만들어 줌
  MonkeyHandler *monkey = new MonkeyHandler;
  SquirrelHandler *squirrel = new SquirrelHandler;
  DogHandler *dog = new DogHandler;
  monkey->SetNext(squirrel)->SetNext(dog);

  std::cout << "Chain: Monkey > Squirrel > Dog\n\n";
  ClientCode(*monkey);
  std::cout << "\n";
  std::cout << "Subchain: Squirrel > Dog\n\n";
  ClientCode(*squirrel);

  delete monkey;
  delete squirrel;
  delete dog;

  return 0;
}
```


## 참고
[wikipedia](https://en.wikipedia.org/wiki/Chain-of-responsibility_pattern)  
[refactoring.guru](https://refactoring.guru/design-patterns/chain-of-responsibility)  

<br>

    This is personal diary for study documents.
    Please comment if I'm wrong or missing something else 😄. 

[Top](#){: .btn .btn--primary }{: .align-right}