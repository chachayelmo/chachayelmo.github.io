---
published: true
title:  "[Design Pattern] 빌더 패턴(Builder pattern) in C++"
excerpt: "다양한 디자인패턴에 대해 알아보기"

categories:
  - DesignPattern
tags:
  - [빌더, 디자인패턴, DesignPattern, Builder, Creational, Pattern]

toc: true
toc_sticky: true
author: chachayelmo
sitemap:
  changefreq : daily
  priority : 1.0
comment: true
date: 2023-01-05
last_modified_at: 2023-01-06
---

## 1. Builder pattern

- 복잡한 객체의 생성하는 방법과 표현하는 방법을 분리
- Product의 속성을 결정하는 부분과 생성하는 부분을 분리하는 것
- Product를 생성할 때 Product에 전달해야 할 파라미터가 많다면 고려하는 것이 좋음

### 1.1. 의도

- 복잡한 객체들을 단계별로 생성할 수 있도록 하는 생성 디자인 패턴
- 이 패턴을 사용하면 같은 제작 코드를 사용하여 객체의 다양한 유형들과 표현을 만들 수 있음

### 1.2. 문제

- 많은 필드와 중첩된 객체들을 단계 별로 초기화해야 하는 복잡한 객체라고 가정할 때,
- 이러한 초기화 코드는 일반적으로 많은 매개변수가 있는 생성자에 있기 마련임

![image](https://user-images.githubusercontent.com/23397039/210970961-bfc3bc17-74ea-4af6-9c05-3a9b71282e0d.png){: .align-center}

- 위와 같이 House 를 만든다고 생각했을 때, 집을 만들기 위해 벽, 문, 창문, 지붕 등 다양한 것들을 기반으로 만들어져야 함
- 이를 House의 생성자의 전부 넣어 만드는 방법이 있는데 이는 아래와 같은 문제를 발생시킴
    
![image](https://user-images.githubusercontent.com/23397039/210971010-7e4bfc66-af76-4d17-8913-c9c012c31e77.png){: .align-center}

- 대부분의 매개변수가 사용되지 않아 생성자의 코드가 예쁘지 않게 보이지 않음

### 1.3. 해결책

- Builder 패턴은 자신의 클래스에서 객체 생성 코드를 추출하여 builders 라는 별도의 객체들로 이동하도록 제안
    
![image](https://user-images.githubusercontent.com/23397039/210971087-d5cda280-ee6e-43b1-b67e-afd80dffabf6.png){: .align-center} 

- 빌더는 제품이 생성되는 동안 다른 개체들이 제품에 접근하는 것을 허용하지 않음
- 객체 생성을 일련의 단계들로 정리하며, 객체를 생성하고 싶으면 위 단계들을 빌더 객체에 실행하면 되고,
- 모든 단계를 호출할 필요 없이 특정 configuration을 하여 필요한 단계들만 호출하면 됨

## 2. 구조

![image](https://user-images.githubusercontent.com/23397039/210971156-8f0522a2-bfe3-4b01-b07e-dbc8d6395ec0.png){: .align-center} 

- Director
    - Builder를 가지고 특정 configuration만 실행이 가능하도록 함
    - Director가 없이도 Builder pattern 사용하는 데 문제 없음
- Builder
    - 인터페이스로 모든 유형의 빌더들에 위해 공통적인 제품 생성 단계들을 선언
- Concrete Builder
    - Builder를 상속받은 구체적인 클래스로 인터페이스에 대한 구현뿐만 아니라 product에 관련된 메소드를 추가적으로 구현 필요
    - 즉 인터페이스에 따르지 않는 기능도 추가가 가능
- Product
    - 빌더에 의해 output으로 나온 객체
- Client
    - 빌더 객체들 중 하나를 디렉터와 연결해서 사용

## 3. 사용

1. 여러가지 선택적인 매개변수가 있는 생성자를 단순화시키기 위해 Builder 패턴을 사용

    ```cpp
    class Pizza {
        Pizza(int size) { ... }
        Pizza(int size, boolean cheese) { ... }
        Pizza(int size, boolean cheese, boolean pepperoni) { ... }
        // …
    ```

2. 코드가 일부 제품의 다른 표현들을 생성할 수 있도록 하고 싶을 때 사용
    - 빌더 패턴은 제품의 다양한 표현의 생성 과정이 세부 사항만 다른 유사한 단계를 포함할 때 적용 할 수 있음  

3. 복잡한 객체들을 생성할 때 사용

## 4. Pros and Cons

### 4.1. Pros

- 객체의 생성을 단계 별로 하거나 반복적으로 하는 경우 사용하기 좋음
- 제품의 다양한 표현을 구축할 때 동일한 구성 코드를 재사용
- Single Responsibility Principle, 제품의 Business Logic 의 복잡한 구성 코드를 isolate 할 수 있음

### 4.2. Cons

- Code Complexity, 여러 새로운 클래스과 인터페이스 도입이 필요

## 5. 코드로 알아보기

```cpp
#include <iostream>
#include <vector>

// 특정 Product
class Product{
public:
    std::vector<std::string> parts_;
    void ListParts() const {
        std::cout << "Product parts: ";
        for (size_t i=0; i < parts_.size(); i++) {
            if(parts_[i]== parts_.back()) {
                std::cout << parts_[i];
            }
            else {
                std::cout << parts_[i] << ", ";
            }
        }
        std::cout << "\n\n"; 
    }
};

// 빌더 인터페이스
class Builder
{
public:
    virtual ~Builder(){}
    // 인터페이스이므로 마찬가지로 순수가상함수로 선언
    virtual void buildPartA() const = 0;
    virtual void buildPartB() const = 0;
    virtual void buildPartC() const = 0;
};

// Concrete 클래스 구현
class ConcreteBuilder : public Builder
{
public:
    // 생성자에서 새로운 Product를 생성하고, 소멸자에서 Product를 지워서 자원 관리
    ConcreteBuilder() { this->Reset(); }
    ~ConcreteBuilder(){ delete product; }

    void Reset(){
        this->product = new Product();
    }

    void buildPartA() const override {
        this->product->parts_.push_back("PartA1");
    }

    void buildPartB() const override {
        this->product->parts_.push_back("PartB1");
    }

    void buildPartC() const override {
        this->product->parts_.push_back("PartC1");
    }

    Product* GetProduct() {
        Product* result= this->product;
        this->Reset();
        return result;
    }
private:
    Product* product;
};

// Director는 building steps를 실행하는 역할만 함.
// 이는 특정 configuration만 build 하는 것을 도와줌.
class Director{
public:
    // 특정한 builder를 set할 수 있음
    // 즉, 추후 새로운 builder를 도입해서 확장시킬 수 있음
    void set_builder(Builder* builder){
        this->builder = builder;
    }

    // A만 만드는 함수
    void BuildMinimalViableProduct(){
        this->builder->buildPartA();
    }

    // A, B, C를 만드는 함수
    void BuildFullFeaturedProduct(){
        this->builder->buildPartA();
        this->builder->buildPartB();
        this->builder->buildPartC();
    }
private:
    Builder* builder;
};

// Client
void ClientCode(Director& director)
{
    ConcreteBuilder* builder = new ConcreteBuilder();
    director.set_builder(builder);
    std::cout << "One product:\n";
    director.BuildMinimalViableProduct();
    
    Product* p= builder->GetProduct();
    p->ListParts();
    delete p;

    std::cout << "Full products:\n"; 
    director.BuildFullFeaturedProduct();

    p= builder->GetProduct();
    p->ListParts();
    delete p;

    std::cout << "Customized product:\n";
    // 디렉터 없이 특정 파트만 빌드하는 것도 가능
    builder->buildPartA();
    builder->buildPartC();
    p=builder->GetProduct();
    p->ListParts();
    delete p;

    delete builder;
}

int main()
{
    Director* director= new Director();
    ClientCode(*director);
    delete director;
    return 0;   
}
```

## 참고
[wikipedia](https://en.wikipedia.org/wiki/Builder_pattern)  
[refactoring.guru](https://refactoring.guru/design-patterns/builder)

<br>

    This is personal diary for study documents.
    Please comment if I'm wrong or missing something else 😄. 

[Top](#){: .btn .btn--primary }{: .align-right}