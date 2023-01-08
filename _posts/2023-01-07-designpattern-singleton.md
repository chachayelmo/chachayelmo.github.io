---
published: true
title:  "[Design Pattern] Creational - Singleton in C++"
excerpt: "다양한 디자인패턴에 대해 알아보기"

categories:
  - Algorithm
tags:
  - [DesignPattern, Singleton, Creational, Pattern]

toc: true
toc_sticky: true
 
date: 2023-01-07
last_modified_at: 2023-01-07
---

## 1. Singleton

- 전역변수와 거의 비슷한 pros, cons를 가짐
- 생성자가 여러 차례 호출되더라도 실제 생성되는 객체는 하나이고 최초의 생성자가 생성한 객체를 리턴

### 1.1. 의도

- 싱클톤은 클래스에 인스턴스가 하나만 있도록 하면서 이 인스턴스에 대한 전역 액세스를 제공하는 디자인 패턴

### 1.2. 문제

- 싱클톤은 한번에 두 가지의 문제를 동시에 해결함으로써 Single Responsibility Principle을 위반함
  - 클래스에 인스턴스가 하나만 있도록 함
    - 생성자 호출은 반드시 새 객체를 반호나해야 하므로 이는 일반 생성자로 구현할 수 없음
  - 해당 인스턴스에 대한 전역 접근을 제공
    - 전역 변수들을 잠재적으로 변수를 덮어 쓸 수 있고 그로 인해 충돌이 발생할 수 있음

### 1.3. 해결책

- 다른 객체들이 싱클톤 class와 함께 new 연산자를 사용하지 못하도록 default 생성자를 private에 넣음
- 생성자 역할을 하는 static 메소드를 만들어서, 메소드 내부에서 private에 넣은 default 생성자를 만들고 그 다음 호출들은 캐시된 객체를 반환

## 2. 구조

- 싱클톤 class는 static 메소드인 getInstance()를 선언하고 default 생성자는 private으로 숨겨야 함

![image](https://user-images.githubusercontent.com/23397039/211189249-cec0de46-56ba-4193-8be2-a13cf7c71de4.png){: .align-center}

## 3. 사용

- 프로그램의 클새스에 모든 클라이언트가 사용할 수 있는 단일 인스턴스만 있어야 할 경우에 사용
- 예를 들면 프로그램의 다른 부분들에세 공유되는 단일 데이터베이스 객체
  - 싱글톤 패턴은 getInstance() 함수를 제외하고 클래스 객체들을 생성할 수 있는 모든 다른 수단들을 비활성화 해야 됨

## 4. 장단점

- 장점
  - 클래스가 하나의 인스턴스만 갖는다는 것을 확신할 수 있음
  - 인스턴스에 대한 전역 접근 지점을 얻음
  - 해당 객체는 처음 요청될 때만 초기화됨
- 단점
    - Single Responsibility Principle을 위반
    - 잘못된 디자인을 가릴 수 있음(예를 들어 프로그램 컴포넌트들이 서로에 대해 많이 아는 경우)
    - 다중 스레드 환경에서 여러 스레드가 싱글톤 객체를 여러번 생성하지 않도록 핸들링 필요
    - 클라이언트 코드를 유닛 테스트하기가 어려울 수 있음
        - 많은 테스트 프레임워크들이 mock 객체들을 생성할 때 상속에 의존하기 때문
        - 싱클톤 클래스의 생성자는 private이고 대부분의 언어에서 static method를 오버라이딩 하는 것이 불가능

## 5. 코드로 알아보기

```cpp
#include <iostream>
#include <string>

// Singleton 클래스
class Singleton
{
private:
  Singleton(const std::string value): value_(value)
  {
    std::cout << "Singleton constructor" << "\n";
  }

  static Singleton* singleton_;
  std::string value_;
public:
  // copy constructor 방지
  Singleton(Singleton &other) = delete;
  // copy assignement 방지
  void operator=(const Singleton &) = delete;

  // static 함수로 singleton 객체를 컨트롤함
  // static 함수로 하는 이유는 초기에 객체가 없어도 함수 콜을 위해 
  static Singleton *GetInstance(const std::string& value);
  void SomeBusinessLogic()
  {
    // ...
  }

  std::string value() const{
      return value_;
  } 
};

// 초기에 nullptr로 설정
Singleton* Singleton::singleton_= nullptr;;

// GetInstance 함수 바디 구현
Singleton *Singleton::GetInstance(const std::string& value)
{
  // singleton 객체가 있는지 없는지 유무를 체크하여 없으면 초기에 1번 생성
  // 다음부턴 존재하는 객체를 사용
  if(singleton_ == nullptr){
    singleton_ = new Singleton(value);
  }
  return singleton_;
}

int main()
{
  Singleton* singleton = Singleton::GetInstance("FOO");
  std::cout << singleton->value() << "\n";
  
  singleton = Singleton::GetInstance("BAR");
  std::cout << singleton->value() << "\n";

  return 0;
}
```

- Thread-safe 싱글톤

```cpp
#include <iostream>
#include <mutex>
#include <thread>

class Singleton
{
// private으로 instance와 mutex를 관리
private:
  static Singleton * pinstance_;
  static std::mutex mutex_;

protected:
  Singleton(const std::string value): value_(value)
  {
  }
  ~Singleton() {}
  std::string value_;

public:
  // copy 생성자 대입연산자 불가능
  Singleton(Singleton &other) = delete;
  void operator=(const Singleton &) = delete;

  // static 메소드
  static Singleton *GetInstance(const std::string& value);
  void SomeBusinessLogic()
  {
      // ...
  }
  
  std::string value() const{
      return value_;
  } 
};

Singleton* Singleton::pinstance_{nullptr};
std::mutex Singleton::mutex_;

// lock_guard를 통해 스레드 간 접근 제어
Singleton *Singleton::GetInstance(const std::string& value)
{
  std::lock_guard<std::mutex> lock(mutex_);
  if (pinstance_ == nullptr)
  {
    pinstance_ = new Singleton(value);
  }
  return pinstance_;
}

void ThreadFoo(){
  // Following code emulates slow initialization.
  std::this_thread::sleep_for(std::chrono::milliseconds(1000));
  Singleton* singleton = Singleton::GetInstance("FOO");
  std::cout << singleton->value() << "\n";
}

void ThreadBar(){
  // Following code emulates slow initialization.
  std::this_thread::sleep_for(std::chrono::milliseconds(1000));
  Singleton* singleton = Singleton::GetInstance("BAR");
  std::cout << singleton->value() << "\n";
}

int main()
{   
  std::cout <<"If you see the same value, then singleton was reused (yay!\n" <<
              "If you see different values, then 2 singletons were created (booo!!)\n\n" <<
              "RESULT:\n";   
  std::thread t1(ThreadFoo);
  std::thread t2(ThreadBar);
  t1.join();
  t2.join();
  
  return 0;
}
```

## 참고
[wikipedia](https://en.wikipedia.org/wiki/Singleton_pattern)  
[refactoring.guru](https://refactoring.guru/design-patterns/singleton)  

<br>

    This is personal diary for study documents.
    Please comment if I'm wrong or missing something else 😄. 

[Top](#){: .btn .btn--primary }{: .align-right}