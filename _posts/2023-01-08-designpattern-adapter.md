---
published: true
title:  "[Design Pattern] 어댑터 패턴(Adapter) in C++"
excerpt: "다양한 디자인 패턴에 대해 알아보기"

categories:
  - DesignPattern
tags:
  - [어댑터, 디자인패턴, DesignPattern, Adapter, Structural, Pattern]

toc: true
toc_sticky: true
author: chachayelmo
sitemap:
  changefreq : daily
  priority : 1.0
comment: true
date: 2023-01-09
last_modified_at: 2023-01-09
---

## 1. Adapter

- 한 클래스의 인터페이스를 클라이언트에서 사용하고자 하는 다른 인터페이스로 변환
- 어댑터를 이용하면 인터페이스 호환성 문제로 같이 쓸 수 없는 클래스들을 연결해서 사용 가능
- 호환되지 않는 인터페이스를 사용하는 클라이언트를 그대로 활용 가능
- 예를 들어 전기 콘센트를 보면 한국의 플러그를 일본 소켓에 끼우려면 어댑터를 사용해야 함
- 레거시 코드를 기반으로 하는 시스템에서 자주 사용

### 1.1. 의도

- 호환되지 않는 인터페이스를 가진 객체들이 협업할 수 있도록 하는 구조적 디자인 패턴

### 1.2. 문제

- 모니터링 앱을 만들고 있고, 이 앱은 여러 소스에서 데이터를 XML 형식으로 다운로드 한 후,
- 사용자에게 보기 좋은 차트들과 다이어그램들을 제공한다고 가정
- 타사의 분석 라이브러리를 통합하여 앱을 개선하기로 했는데 해당 라이브러리가 JSON 형식의 데이터로만 작동하는 경우

![image](https://user-images.githubusercontent.com/23397039/211190219-3213cceb-ceba-474e-a282-c356c41691d5.png){: .align-center}

- 라이브러리를 XML과 작동하도록 변경할 수 있으나, 그러면 라이브러리에 의존하는 일부 기존 코드가 손상될 수 있음

### 1.3. 해결책

- 어댑터 패턴을 이용하여 해결할 수 있음
- 어댑터는 한 객체의 인터페이스를 다른 객체가 이용할 수 있도록 변환하는 객체
- 어댑터를 만드는 과정에서 복잡성을 피하기 위해 클래스 중 하나를 wrapping 함
- 데이터를 다양한 형식으로 변환할 수 있을 뿐만 아니라 다른 인터페이스를 가진 객체들이 협업하는 데에도 도움을 줄 수 있음
- 동작 과정
    - 어댑터는 기존에 있던 객체 중 하나와 호환되는 인터페이스를 받음
    - 이 인터페이스를 사용하면 기존 객체는 어댑터의 메소드들을 안전하게 호출 가능
    - 호출을 수신하면 어댑터는 이 요청을 두 번째 객체에 해당 객체가 예상하는 형식과 순서대로 전달 가능

![image](https://user-images.githubusercontent.com/23397039/211190237-8814ad32-03e1-49ad-93fa-de557965dd80.png){: .align-center}

## 2. 구조

### 2.1. Object Adapter
- Object composition principle을 사용
- 한 객체의 인터페이스로 구현하고 다른 객체는 wrapping 함
        
![image](https://user-images.githubusercontent.com/23397039/211190284-3b899dac-5fd4-45a5-af9e-afb37f603ca5.png){: .align-center}

- Client
  - 프로그램의 기존 비즈니스 로직을 포함하는 클래스
- Client Interface
  - 다른 클래스들이 클라이언트 코드와 공동 작업할 수 있도록 따라야 하는 인터페이스
- Service
  - 3rd party or Legacy의 사용중인 클래스
  - 클라이언트가 이를 직접 사용하는 것은 불가 (왜냐하면 서비스 class는 호환되지 않는 인터페이스를 가지기 때문에)
- Adapter
  - 클라이언트와 서비스 양쪽에서 작동할 수 있도록 하는 class
  - 서비스 객체를 wrapping 하면서 클라이언트 인터페이스를 기반으로 구현
  - 클라이언트로부터 호출을 수신한 후 이 호출을 wrapping된 서비스 객체가 이해할 수 있는 형식의 호출로 변환

### 2.2. Class Adapter
- 이 구현은 상속을 사용하며, Adapter는 동시에 두 객체의 인터페이스를 상속
- 이 방식은 C++와 같이 다중 상속을 지원하는 프로그래밍 언어에서만 구현 가능

![image](https://user-images.githubusercontent.com/23397039/211190309-39a124d7-c1d8-4a1d-b35a-30a5e78454df.png){: .align-center}

- 클래스 어댑터는 객체를 wrapping할 필요가 없음(이유는 클라이언트와 서비스 양쪽에서 상속 받기 때문에)

## 3. 사용

1. 기존 클래스를 사용하고 싶지만 인터페이스가 나머지 코드와 호환되지 않을 때 사용
  - 이 패턴은 legacy 코드와 3rd party 클래스 또는 special한 인터페이스가 있는 다른 클래스 간의 변환기 역할을 하는 중간 layer
2. 부모 클래스에 추가할 수 없는 어떤 공통 기능들을 가지지 않은 여러 legacy 서브클래스들이 재사용하려는 경우에 사용
  - 문제
    - 각 서브클래스를 확장한 후 누락된 기능들을 새로운 서브클래스들에 넣을 수 있지만 해당 코드를 모든 새로운 클래스들에 복제해야 되지 때문에 smell 코드임
  - 해결
    - 깔끔한 해결책은 누락된 기능을 어댑터 클래스에 넣는 것
    - 어댑터 내부에 누락된 기능이 있는 개체들을 wrapping 하면 필요한 기능들을 얻을 수 있음
    - 이 해결책은 대상 클래스들에 반드시 공통 인터페이스가 있어야 하며 어댑터의 field는 해당 인터페이스를 따라야 함 ( Decorator 패턴과 매우 유사 )

## 4. Pros and Cons

### 4.1. Pros
  - Single Responsibility Principle, 프로그램의 기존 비즈니스 로직에서 인터페이스 또는 데이터 변환 코드를 분리할 수 있음
  - Open/Closed Principle, 클라이언트 코드가 클라이언트 인터페이스를 통해 어댑터와 동작하면 기존의 클라이언트 코드를 손상하지 않고 새로운 코드를 도입할 수 있음

### 4.2. Cons
  - Code complexity, 다수의 새로운 인터페이스와 클래스들을 도입해야 하므로 코드의 복잡성이 증가
  - 코드의 남은 부분과 작동하도록 서비스 클래스를 변경하는 편이 효율적일 수 있음

## 2. 코드로 알아보기

### Object Adapter

```cpp
#include <iostream>
#include <string>
#include <algorithm>

class Target {
 public:
  virtual ~Target() = default;
  virtual std::string Request() const {
    return "Target: The default target's behavior.";
  }
};

class Adaptee {
 public:
  std::string SpecificRequest() const {
    return ".eetpadA eht fo roivaheb laicepS";
  }
};

// Adapter 클래스
class Adapter : public Target {
public:
  // 생성자에 Adaptee를 받아서 넘김
  // 경우에 따라 다양한 Adaptee를 만들어서 Adapter에 적용할 수 있음
  Adapter(Adaptee *adaptee) : adaptee_(adaptee) {}
  std::string Request() const override {
    std::string to_reverse = this->adaptee_->SpecificRequest();
    std::reverse(to_reverse.begin(), to_reverse.end());
    return "Adapter: (TRANSLATED) " + to_reverse;
  }
private:
  Adaptee *adaptee_;
};

// 클라이언트
void ClientCode(const Target *target) {
  std::cout << target->Request();
}

int main() {
  // 레거시 코드라고 한다면
  Target *target = new Target;
  ClientCode(target);
  std::cout << "\n\n";

  // 새로 추가된 기능인데 문장이 반대로 써 있는 경우
  Adaptee *adaptee = new Adaptee;
  std::cout << "Client: The Adaptee class has a weird interface. See, I don't understand it:\n";
  std::cout << "Adaptee: " << adaptee->SpecificRequest();
  std::cout << "\n\n";

  // 어댑터를 적용하여 올바른 문장으로 만듬
  std::cout << "Client: But I can work with it via the Adapter:\n";
  Adapter *adapter = new Adapter(adaptee);
  ClientCode(adapter);
  std::cout << "\n";

  delete target;
  delete adaptee;
  delete adapter;

  return 0;
}
```

### Class adapter ( 다중상속 )

```cpp
#include <iostream>
#include <string>
#include <algorithm>

class Target {
public:
  virtual ~Target() = default;
  virtual std::string Request() const {
    return "Target: The default target's behavior.";
  }
};

class Adaptee {
public:
  std::string SpecificRequest() const {
    return ".eetpadA eht fo roivaheb laicepS";
  }
};

// Adapter 클래스가 다중 상속인 경우
class Adapter : public Target, public Adaptee {
public:
  Adapter() {}
  std::string Request() const override {
    std::string to_reverse = SpecificRequest();
    std::reverse(to_reverse.begin(), to_reverse.end());
    return "Adapter: (TRANSLATED) " + to_reverse;
  }
};

// 클라이언트
void ClientCode(const Target *target) {
  std::cout << target->Request();
}

int main() {
  // 레거시 코드라고 한다면
  Target *target = new Target;
  ClientCode(target);
  std::cout << "\n\n";

  // 새로 추가된 기능인데 문장이 반대로 써 있는 경우
  Adaptee *adaptee = new Adaptee;
  std::cout << "Client: The Adaptee class has a weird interface. See, I don't understand it:\n";
  std::cout << "Adaptee: " << adaptee->SpecificRequest();
  std::cout << "\n\n";

  // 어댑터를 적용하여 올바른 문장으로 만듬
  std::cout << "Client: But I can work with it via the Adapter:\n";
  Adapter *adapter = new Adapter;
  ClientCode(adapter);
  std::cout << "\n";

  delete target;
  delete adaptee;
  delete adapter;

  return 0;
}
```

## 참고
[wikipedia](https://en.wikipedia.org/wiki/Adapter_pattern)  
[refactoring.guru](https://refactoring.guru/design-patterns/adapter)  

<br>

    This is personal diary for study documents.
    Please comment if I'm wrong or missing something else 😄. 

[Top](#){: .btn .btn--primary }{: .align-right}