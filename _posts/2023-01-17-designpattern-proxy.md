---
published: true
title:  "[Design Pattern] 프록시 패턴(Proxy) in C++"
excerpt: "다양한 디자인 패턴에 대해 알아보기"

categories:
  - Algorithm
tags:
  - [프록시, DesignPattern, Proxy, Structural, Pattern]

toc: true
toc_sticky: true
author: chachayelmo
sitemap:
  changefreq : daily
  priority : 1.0
comment: true
date: 2023-01-17
last_modified_at: 2023-01-17
---

## 1. Proxy

- 일반적인 형태는 다른 항목에 대한 인터페이스 역할을 하는 클래스
- 네트워크 connection, 메모리가 큰 객체, 파일, 비용이 많이 들거나 복제가 불가능한 자원에 대한 인터페이스가 될 수 있음
- Proxy는 실제 사용 중인 객체에 접근하기 위해 클라이언트가 호출하는 wrapper 또는 agent 객체
- Proxy를 사용하면 객체 전달 뿐 만 아니라 추가적인 로직도 제공 가능

### 1.1. 의도

- 다른 객체에 대한 대체자 또는 placeholder를 제공할 수 있는 구조적 디자인 패턴
- 원래 객체에 대한 접근을 제어하므로 요청이 원래 객체에 전달되기 전 또는 후에 무언가를 수행할 수 있도록 함

### 1.2. 문제

- 객체에 대한 접근을 제한하는 이유는?
    - 방대한 양의 시스템 자원을 소모하는 객체가 있다고 가정하고,
    - 이 객체는 필요할 때가 있지만 항상 필요한 것은 아님
        
![image](https://user-images.githubusercontent.com/23397039/212868286-088f7e1b-ad98-4df1-9ec7-ea668490b587.png){: .align-center}        
    - 필요할 때만 객체를 만들어서 lazy initialization 을 구현하면,
    - 객체의 모든 클라이언트들은 초기화 코드를 실행해야 되는데 이는 많은 중복 코드를 만들 것임
    - 코드를 클래스에 직접 넣을 수도 있지만 폐쇄된 타사 라이브러리의 일부일 수도 있음

### 1.3. 해결책

- 서비스 객체와 같은 인터페이스로 새로운 proxy 클래스를 생성
- proxy 객체를 원래 객체의 모든 클라이언트들에 전달하도록 앱을 업데이트할 수 있음
- 클라이언트로부터 요청을 받으면 proxy는 실제 서비스 객체를 생성하고 모든 작업을 위임

![image](https://user-images.githubusercontent.com/23397039/212868490-a46d85e2-ef75-4c80-a132-e3828ba1157b.png){: .align-center}

## 2. 구조

![image](https://user-images.githubusercontent.com/23397039/212868562-e99a327f-fdde-429b-8c5f-ac0b80e49242.png){: .align-center}

- ServiceInterface
    - 서비스의 인터페이스를 선언하여 proxy가 서비스 객체로 위장하려면 이 인터페이스를 따라야 함
- Service
    - 어떤 비즈니스 로직을 제공하는 클래스
- Proxy
    - 서비스 객체를 가리키는 참조 field가 있음
    - proxy가 요청에 대한 처리(로깅, 액세스 제어, 캐싱 등)를 완료하면, 그 후 처리된 요청을 서비스 객체로 전달
    - 일반적으로 서비스 객체들의 lifecycle을 관리함
- Client
    - 같은 인터페이스를 통해 서비스들과 proxy와 함께 동작

## 3. 사용

1. lazy initialization - *virtual proxy*
    - 어쩌다 필요한 무거운 서비스 객체가 항상 가동되어 있어 시스템 자원을 낭비하는 경우
    - 앱이 시작될 때 객체를 생성하는 대신, 객체 초기화를 실제로 초기화가 필요한 시점까지 지연할 수 있음
2. Access control - *protection proxy*
    - 특정 클라이언트들만 서비스 객체를 사용할 수 있도록 하려는 경우 사용
    - 클라이언트의 자격 증명이 어떤 정해진 기준과 일치하는 경우에만 서비스 객체에 요청
3. Local execution of a remote service - *remote proxy*
    - proxy는 네트워크를 통해 클라이언트 요청을 전달하여 복잡한 세부 사항을 처리
4. Logging requests - *logging proxy*
    - 각 요청을 서비스에 전달하기 전에 로깅을 할 수 있음
5. Caching request reulsts - *caching proxy*
    - 항상 같은 결과를 생성하는 반복 요청들에 대해 캐싱을 구현할 수 있고, 요청들의 매개변수들을 캐시의 key로 사용할 수 있음
6. Smart reference
    - 서비스 객체 또는 결과에 대한 참조를 얻은 클라이언트들을 추적할 수 있음
    - 클라이언트들을 점검하여 상태를 확인할 수 있고, 비어 있으면 서비스 객체를 닫고 시스템 자원을 확보할 수 있음

## 4. Pros and Cons

- Pros
    - 클라이언트들이 알지 못하는 상태에서 서비스 객체를 제어 가능
    - 서비스 객체의 lifecycle 도 관리
    - 서비스 객체가 준비되지 않거나 사용할 수 없는 경우에도 동작
    - Open/Closed Principle : 서비스나 클라이언트들을 변경하지 않고도 새로운 proxy를 도입할 수 있음
- Cons
    - 새로운 클래스들을 많이 도입해야 하므로 코드가 복잡해질 수 있음
    - 서비스의 응답이 늦어질 수 있음

## 5. 코드로 알아보기

```cpp
#include <iostream>

// ServiceInterface로 service와 Proxy가 같이 사용
class ServiceInterface {
 public:
  virtual void Request() const = 0;
};

// 특정한 비지니스 로직을 포함
class Service : public ServiceInterface {
 public:
  void Request() const override {
    std::cout << "Service: Handling request.\n";
  }
};

// Proxy는 Service와 같은 인터페이스를 사용
class Proxy : public ServiceInterface {
  /**
   * @var Service
   */
 private:
  Service *service;

  // 추가적인 동작들을 할 수 있음
  bool CheckAccess() const {
    std::cout << "Proxy: Checking access prior to firing a real request.\n";
    return true;
  }
  void LogAccess() const {
    std::cout << "Proxy: Logging the time of request.\n";
  }

// proxy는 service 객체에 대해 참조를 유지
 public:
  Proxy(Service *service) : service(new Service(*service)) {
  }

  ~Proxy() {
    delete service;
  }

  // proxy는 캐싱, 액세스 제어, 로깅 등 작업을 수행한 다음 결과에 따라 연결된 RealSubject 객체의 동일한 메소드에 실행을 전달
  void Request() const override {
    if (this->CheckAccess()) {
      this->service->Request();
      this->LogAccess();
    }
  }
};

// 클라이언트 코드는 subjects와 proxies 모두 작업 가능
void ClientCode(const ServiceInterface &subject) {
  // ...
  subject.Request();
  // ...
}

int main() {
  std::cout << "Client: Executing the client code with a real service:\n";
  Service *service = new Service;
  ClientCode(*service);
  std::cout << "\n";
  std::cout << "Client: Executing the same client code with a proxy:\n";
  Proxy *proxy = new Proxy(service);
  ClientCode(*proxy);

  delete service;
  delete proxy;
  return 0;
}
```


## 참고
[wikipedia](https://en.wikipedia.org/wiki/Proxy_pattern)  
[refactoring.guru](https://refactoring.guru/design-patterns/proxy)  

<br>

    This is personal diary for study documents.
    Please comment if I'm wrong or missing something else 😄. 

[Top](#){: .btn .btn--primary }{: .align-right}