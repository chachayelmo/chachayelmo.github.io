---
published: true
title:  "[Design Pattern] Creational - Builder pattern"
excerpt: "다양한 디자인패턴에 대해 알아보기"

categories:
  - Algorithm
tags:
  - [DesignPattern, Builder, Creational, Pattern]

toc: true
toc_sticky: true
 
date: 2023-01-05
last_modified_at: 2023-01-05
---

## 1. Builder pattern
- 복잡한 객체를 생성하는 방법과 표현하는 방법을 분리
- Product의 속성을 결정하는 부분과 생성하는 부분을 분리시키는 것
- Product를 생성할 때 Product에 전달해야 할 파라미터가 많다면 고려

![image](https://user-images.githubusercontent.com/23397039/210322853-63db5580-e248-4534-b7a8-b99c0d8ea952.png)

- Director : Builder를 가지고 특정 configuration만 실행이 가능하도록 함 Director가 없이도 Builder pattern 사용 가능
- Builder : build 관련된 특정 인터페이스
- ConcreteBuilder : Builder 를 상속받은 구체적인 클래스로 인터페이스에 대한 구현뿐만 아니라 product에 관련된 메소드를 추가적으로 구현 필요
- Product : 특정 일을 수행하는 프로덕트

## 2. 코드로 알아보기

```cpp
#include <iostream>
#include <vector>

// 특정 Product
class Product{
public:
    std::vector<std::string> parts_;
    void ListParts()const{
        std::cout << "Product parts: ";
        for (size_t i=0; i<parts_.size(); i++) {
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
    virtual void buildPartA() const = 0;
    virtual void buildPartB() const = 0;
    virtual void buildPartC() const = 0;
};

// Concrete 클래스 구현
class ConcreteBuilder : public Builder
{
public:
    ConcreteBuilder() { this->Reset(); }
    ~ConcreteBuilder(){ delete product; }

    void Reset(){
        this->product= new Product();
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
    void set_builder(Builder* builder){
        this->builder = builder;
    }

    void BuildMinimalViableProduct(){
        this->builder->buildPartA();
    }
    
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
    // 1개만 빌드하는 설정
    director.BuildMinimalViableProduct();
    
    Product* p= builder->GetProduct();
    p->ListParts();
    delete p;

    std::cout << "Full products:\n"; 
    // 3개 다 빌드하는 설정
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
[d-yong](https://d-yong.tistory.com/42)  
[refactoring.guru](https://refactoring.guru/design-patterns/builder/cpp/example)

<br>

    This is personal diary for study documents.
    Please comment if I'm wrong or missing something else 😄. 

[Top](#){: .btn .btn--primary }{: .align-right}