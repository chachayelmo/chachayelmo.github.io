---
published: true
title:  "[Design Pattern] Abstract Factory pattern"
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

## 1. Abstract Factory
- Creational 패턴 중의 하나
- 구체적인 Concrete Class를 지정하지 않고 공통된 테마를 가진 개별 팩토리를 캡슐화하는 방법
- 인터페이스를 제공하는 패턴
    - 관련성 있는 여러 종류의 객체를 일관된 방식으로 생성하는 경우
    - 싱글톤, 팩토리 메서드 패턴을 사용함
    - 클라이언트 소프트웨어는 Concrete 구현을 통해 구체적인 객체를 생성

![image](https://user-images.githubusercontent.com/23397039/210299773-0f8fa5e4-c2db-4640-a6f1-df77f5edadfc.png)

- AbstractFactory : 실제 팩토리 클래스의 공통 인터페이스
- ConcreteFactory : 구체적인 팩토리 클래스로 AbstractFactory 클래스의 추상 메소드를 오버라이드하여 구체적인 제품을 생성
- AbstractProduct : 제품 별 공통 인터페이스
- Product : ConcreteFactory에서 생성되는 객체

## 2. 코드로 알아보기

```cpp
#include <iostream>
#include <memory>

// AbstractProduct
class Button
{
public:
    virtual void paint() = 0;
    virtual ~Button(){}
};

// ProductA
class LinuxButton : public Button
{
public:
    void paint() override final {
        std::cout << "Linux button" << "\n";
    }
};

// ProductB
class WindowsButton : public Button
{
public:
    void paint() override final {
        std::cout << "Windows button" << "\n";
    }
};

// ProductC
class MacButton : public Button
{
public:
    void paint() override final {
        std::cout << "Mac button" << "\n";
    }
};

// AbstractFactory
class GUIFactory
{
public:
    virtual Button* create_button() = 0;
    virtual ~GUIFactory(){}
};

// ConcreteFactoryA
class LinuxFactory : public GUIFactory
{
public:
    Button* create_button() override final {
        return new LinuxButton();
    }
};

// ConcreteFactoryB
class WindowsFactory : public GUIFactory
{
public:
    Button* create_button() override final {
        return new WindowsButton();
    }
};

// ConcreteFactoryC
class MacFactory : public GUIFactory
{
public:
    Button* create_button() override final {
        return new MacButton();
    }
};

// Client
int main()
{
    std::unique_ptr<GUIFactory> linux_factory{new LinuxFactory};
    std::unique_ptr<GUIFactory> windows_factory{new WindowsFactory};
    std::unique_ptr<GUIFactory> mac_factory{new MacFactory};

    Button* button = linux_factory->create_button();
    button->paint();
    button = windows_factory->create_button();
    button->paint();
    button = mac_factory->create_button();
    button->paint();

    return 0;
}
```

## 참고
[https://en.wikipedia.org/wiki/Abstract_factory_pattern](https://en.wikipedia.org/wiki/Abstract_factory_pattern)  
[https://4z7l.github.io/2020/12/25/design_pattern_GoF.html](https://4z7l.github.io/2020/12/25/design_pattern_GoF.html)  
[https://gmlwjd9405.github.io/2018/08/08/abstract-factory-pattern.html](https://gmlwjd9405.github.io/2018/08/08/abstract-factory-pattern.html)

<br>

    This is personal diary for study documents.
    Please comment if I'm wrong or missing something else 😄. 

[Top](#){: .btn .btn--primary }{: .align-right}