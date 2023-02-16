---
published: true
title:  "[Design Pattern] Command in C++"
excerpt: "다양한 디자인 패턴에 대해 알아보기"

categories:
  - Algorithm
tags:
  - [DesignPattern, Command, Pattern]

toc: true
toc_sticky: true
 
date: 2023-02-15
last_modified_at: 2023-02-16
---


## 1. Command

### 1.1. 의도

- 요청에 대한 모든 정보가 포함된 독립적인 객체로 변환하는 행동 디자인 패턴
- 다양한 요청들이 있는 메소드의 argument로 인수화 할 수 있으며, 요청의 실행을 지연 또는 대기열에 넣을 수 있고, 실행 취소도 할 수 있는 작업을 지원

### 1.2. 문제

- 텍스트 편집기를 개발한다고 가정했을 때, 편집기의 도구 모음(toolbar)를 만든다면
- 도구 모음의 버튼들과 다양한 대화 상자들의 일반 버튼들에 사용할 수 있는 깔끔한 버튼 클래스를 만듬
- 버튼들은 모두 비슷해 보이지만 각각 다른 기능을 수행
- 따라서 각각 버튼들에 대한 클릭 handler들은 각각 자식 클래스들을 만듬

![image](https://user-images.githubusercontent.com/23397039/218973057-01f9f537-5704-4d56-9c62-fd8ce826396e.png){: .align-center}

- 그런 후, 버튼에 엄청난 수의 자식 클래스들이 있으며, 기초 버튼 클래스를 수정할 때마다 이러한 자식 클래스의 코드를 깨뜨릴 위험이 있음

### 1.3. 해결책

- 좋은 소프트웨어 디자인은 종종 *principle of separation of concerns* 를 기반으로 함
- 일반적인 예로 GUI용 layer와 비즈니스 로직용 layer를 분리
- GUI layer는 모든 입력을 캡처하고 화면에 그림을 렌더링하고 사용자와 앱이 수행하는 작업의 결과를 나타내는 역할
- 달의 행성 궤도를 계산하거나 연간 보고서를 작성하는 것과 같은 중요한 작업을 수행할 때는 비즈니스 layer에 작업을 위임
- GUI 객체가 비즈니스 로직 객체의 메소드를 호출하고 일부 argument를 전달
- 즉, 한 객체가 다른 객체에 요청을 보내는 것

![image](https://user-images.githubusercontent.com/23397039/218973146-490a3ace-de45-4e56-aa42-45de4a873ee4.png){: .align-center}

- 커맨드 패턴은 GUI 객체들이 이러한 요청을 직접 보내서는 안되기 때문에 대신 모든 요청과 관련된 세부 정보들을 작동시키는 단일 메소드를 가진 별도의 커맨드 클래스로 추출
- 커맨드 객체는 GUI와 비즈니스 로직 객체들 간의 링크 역할
- 이제부터 GUI 객체는 어떤 비즈니스 로직 객체가 요청을 받을지와 처리할 지에 대해 알 필요가 없어짐
- 커맨드에서 모든 세부 사항을 처리

![image](https://user-images.githubusercontent.com/23397039/218973242-79e65c14-1c4a-49c7-a8ee-02584670badc.png){: .align-center}

- 다음 단계로 커맨드 인터페이스를 구현
- 일반적으로 커맨드는 매개 변수들을 받지 않는 단일 실행 메소드만 가짐
- 이 인터페이스는 다양한 커맨드들을 커맨드들의 concrete 클래스들과 결합하지 않고 사용
- 이는 연결된 커맨드 객체들을 전환할 수 있고, 런타임에 행동을 변경할 수 있음
- 커맨드 실행 메소드에 매개변수들이 없는데 이는 데이터로 미리 설정해놓거나 데이터를 자체적으로 가져올 수 있어야 됨

![image](https://user-images.githubusercontent.com/23397039/218973342-6440c45b-f0ab-4a80-b985-248e1ff61619.png){: .align-center}

- 커맨드 패턴을 적용한 후에는 다양한 버튼이 필요하지 않고 1개의 버튼 클래스에 커맨드 객체를 참조하여 이를 실행하도록 하면 됨
- 모든 작업에 대해 많은 커맨드 클래스들을 구현하고 이를 의도된 동작에 따라 특정 객체와 연결해야 됨

## 2. 구조

![image](https://user-images.githubusercontent.com/23397039/219294729-068e1e2b-846e-4db6-b3e4-72593b3337d3.png){: .align-center}

- **Invoker** :
    - 들어온 요청을 시작하는 역할
    - 이 클래스에는 커맨드 객체에 대한 참조를 저장하고 있고, 커맨드 객체의 메소드를 실행
- **Command** :
    - 인터페이스이며 일반적으로 커맨드를 실행하기 위한 단일 메소드만 선언
- **Concrete Command** :
    - 다양한 유형의 요청을 구현
    - 이 클래스 자체적으로 작업을 수행해서는 안되고 비즈니스 로직 객체 중 하나에 호출을 전달해야 됨
    - receiver 객체에서 메소드를 실행하는 데 필요한 매개 변수들은 필드로 선언할 수 있음
- **Receiver** :
    - 비즈니스 로직을 포함
    - 대부분 커맨드들은 요청이 Receiver에게 전달되는 방법에 대한 처리를 하면 Receiver는 실제 작업을 수행
- **Client** :
    - Concrete Command 객체들을 만들고 설정
    - Receiver 객체를 포함한 매개 변수들을 커맨드의 생성자로 전달

## 3. 사용

- 작업을 위한 객체를 매개변수화하려는 경우
    - 커맨드 패턴은 특정 메소드 호출을 독립적인 객체로 전환이 가능
- 작업들의 Queue 대기열에 넣거나 실행 또는 원격으로 실행하려는 경우
    - serialize(파일이나 데이터베이스가 쉽게 쓸 수 있는 문자열로 변환) 가 가능
    - 커맨드의 실행을 지연하고 예약이 가능
- reversible 작업을 구현하려고 하는 경우
    - undo/redo 구현에 대해서 커맨드 패턴이 가장 많이 사용
    - revert operation을 위해 history를 구현해야 함
    - 커맨드 history는 앱 상태 관련 백업들과 함께 실행된 커맨드 객체들을 포함한 **stack**
    - 단점
        - 앱 일부가 비공개일 경우 앱의 상태를 저장하는 것이 쉽지 않지만 Memento 패턴으로 완화할 수 있음
        - 상태 백업들은 상당히 많은 RAM을 소모

## 4. Pros and Cons

- Pros
    - Single Responsibility Principle, 작업을 호출하는 클래스를 분리 가능
    - Open/Closed Principle, 기존 클라이언트 코드에 영향없이 새로운 커맨드 도입 가능
    - undo/redo 실행 가능
    - 작업들의 지연된 실행 가능
    - 간단한 커맨드들의 집합을 복잡한 커맨드로 조합할 수 있음
- Cons
    - Sender/Receiver 사이에 새로운 레이어를 도입하기 때문에 코드가 복잡해짐

## 5. 코드로 알아보기

```cpp
#include <iostream>
#include <string>

// Command 인터페이스
class Command {
 public:
  virtual ~Command() {
  }
  // 단일 메소드를 가짐
  virtual void Execute() const = 0;
};

// Concrete class
class SimpleCommand : public Command {
 private:
  std::string pay_load_;

 public:
  explicit SimpleCommand(std::string pay_load) : pay_load_(pay_load) {
  }
  void Execute() const override {
    std::cout << "SimpleCommand: See, I can do simple things like printing (" << this->pay_load_ << ")\n";
  }
};

// Receiver 클래스에서 특정 비즈니스 로직을 실행
class Receiver {
 public:
  void DoSomething(const std::string &a) {
    std::cout << "Receiver: Working on (" << a << ".)\n";
  }
  void DoSomethingElse(const std::string &b) {
    std::cout << "Receiver: Also working on (" << b << ".)\n";
  }
};

// Concrete 커맨드 클래스에서 receiver와 여러 input을 매개변수로 받아서 전달
class ComplexCommand : public Command {
  /**
   * @var Receiver
   */
 private:
  Receiver *receiver_;

  // receiver function에서 사용할 매개변수들을 필드로 가짐
  std::string a_;
  std::string b_;

 public:
  ComplexCommand(Receiver *receiver, std::string a, std::string b) : receiver_(receiver), a_(a), b_(b) {
  }

  // receiver로 작업을 위임
  void Execute() const override {
    std::cout << "ComplexCommand: Complex stuff should be done by a receiver object.\n";
    this->receiver_->DoSomething(this->a_);
    this->receiver_->DoSomethingElse(this->b_);
  }
};

// 커맨드를 참조하고 들어오는 request를 실행시키는 클래스
class Invoker {
  /**
   * @var Command
   */
 private:
  Command *on_start_;
  /**
   * @var Command
   */
  Command *on_finish_;

 public:
  ~Invoker() {
    delete on_start_;
    delete on_finish_;
  }

  void SetOnStart(Command *command) {
    this->on_start_ = command;
  }
  void SetOnFinish(Command *command) {
    this->on_finish_ = command;
  }

  // 커맨드 객체를 setter로 저장하고 아래와 같이 커맨드 객체의 Execute()를 실행시켜줌
  void DoSomethingImportant() {
    std::cout << "Invoker: Does anybody want something done before I begin?\n";
    if (this->on_start_) {
      this->on_start_->Execute();
    }
    std::cout << "Invoker: ...doing something really important...\n";
    std::cout << "Invoker: Does anybody want something done after I finish?\n";
    if (this->on_finish_) {
      this->on_finish_->Execute();
    }
  }
};

int main() {
  // 클라이언트에서 특정 커맨드 객체를 할당하고 매개변수와 함께 전달
  Invoker *invoker = new Invoker;
  invoker->SetOnStart(new SimpleCommand("Say Hi!"));
  Receiver *receiver = new Receiver;
  invoker->SetOnFinish(new ComplexCommand(receiver, "Send email", "Save report"));
  invoker->DoSomethingImportant();

  delete invoker;
  delete receiver;

  return 0;
}
```

## 참고
[wikipedia](https://en.wikipedia.org/wiki/Command_pattern)  
[refactoring.guru](https://refactoring.guru/design-patterns/command)  

<br>

    This is personal diary for study documents.
    Please comment if I'm wrong or missing something else 😄. 

[Top](#){: .btn .btn--primary }{: .align-right}