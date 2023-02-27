---
published: true
title:  "[Design Pattern] 플라이웨이트 패턴(Flyweight) in C++"
excerpt: "다양한 디자인 패턴에 대해 알아보기"

categories:
  - DesignPattern
tags:
  - [플라이웨이트, 캐시, DesignPattern, Flyweight, Structural, Pattern]

toc: true
toc_sticky: true
author: chachayelmo
sitemap:
  changefreq : daily
  priority : 1.0
comment: true
date: 2023-01-16
last_modified_at: 2023-01-16
---

## 1. Flyweight known as Cache

- 데이터의 일부를 캐싱을 통해 다른 유사한 객체와 공유하여 메모리 사용을 최소화하는 방법
- 이 패턴은 구현, 변경, 테스트 및 재사용이 유연한 디자인을 구현하게 함
- 데이터 구조를 공유한다는 개념을 [Hash_consing](https://en.wikipedia.org/wiki/Hash_consing)이라고 함
    - 해시컨싱은 구조적으로 동일한 값을 공유하는 데 사용되는 기술
    - 메모리 할당의 페널티를 피하면서 이전에 구성된 cons 셀을 재사용 하려고 시도하는 Lisp의 구현에서 참조

### 1.1. 의도

- 각 객체에 모든 데이터를 유지하는 대신 여러 객체들 간에 공통 부분들을 공유하여 사용할 수 있는 RAM에 더 많은 객체들을 포함할 수 있도록 하는 구조적 디자인 패턴
    
![image](https://user-images.githubusercontent.com/23397039/212621321-3c92f66b-adfe-4dcc-926d-d5147a2fa216.png){: .align-center}

### 1.2. 문제

- 플레이어들이 지도를 돌아다니며 서로 총을 쏘는 비디오 게임을 만든다고 가정
- 폭발로 인한 총알들, 미사일들 및 파편들을 사실적인 Particle(입자) 시스템을 구현하여 제공하였는데 사용자의 컴퓨터의 RAM이 충분하지 않아 게임이 충돌했음을 알게 되었음
- 문제의 원인은 Particle(입자)를 가진 많은 데이터를 포함하는 객체들이 새로 생성될 때 마다 계속 늘어나 RAM이 감당하지 못하는 문제가 생김

![image](https://user-images.githubusercontent.com/23397039/212620958-c54c379a-0e3e-43dc-9231-a7810282507c.png){: .align-center}


### 1.3. 해결책

- 위의 그림에서 Particle(입자) 클래스를 보면 color 및 sprite가 다른 필드들보다 더 많은 메모리를 사용한 다는 것을 알 수 있음
- 여기서 더 안 좋은 것은 아래와 같이 다른 필드들에 대비해 두 필드가 모든 입자에 대해 거의 같은 데이터를 저장한다는 것임

![image](https://user-images.githubusercontent.com/23397039/212621044-549b3f82-0ae9-4897-b5bf-fac821a1612d.png){: .align-center}


- 객체의 이러한 상수 데이터를 일반적으로 *"고유한 상태"* 라고 하여 다른 객체들은 이 데이터를 읽을 수만 있고 변경할 수는 없음
- Flyweight 패턴은 객체 내부에 *"공유한 상태"* 의 저장을 중단하는 대신 상태에 의존하는 특정 메소드들에 전달
- *"고유한 상태"* 만 객체 내에 유지되므로 해당 상태는 컨텍스트가 다른 곳에서 재사용할 수 있음
- 이러한 객체들은 *"공유한 상태"* 보다 변형이 훨씬 적은 *"고유한 상태"* 에서만 달라지므로 더 적은 수의 객체만 있으면 됨

![image](https://user-images.githubusercontent.com/23397039/212621113-c1887215-0e86-47ad-92b9-45852266b6b7.png){: .align-center}
    
- *"고유한 상태"* 만 저장하는 객체를 Flyweight(캐시)라고 부름

## 2. 구조

![image](https://user-images.githubusercontent.com/23397039/212621431-4eb78e93-b148-4560-a514-787ce45b2671.png){: .align-center}

1. Flyweight
    1. Flyweight 패턴은 단지 최적화를 하는 방법이라 패턴을 적용하기 전에 프로그램이 동시에 메모리에 유사한 객체들을 대량으로 보유하는 것과 관련된 RAM 사용에 문제가 있는지 확인이 필요
2. Flyweight
    1. 여러 객체들 간에 공유할 수 있는 원래 객체의 state를 포함
    2. 같은 Flyweight 객체가 다양한 Context 에서 사용될 수 있음
    3. 내부에 저장된 state를 *"고유한 상태"* 라고 하며, Flyweight의 메소드에 전달된 상태를 *"공유한 상태"* 라고 함
3. Context : *"공유한 상태"*를 포함하며, 이 상태는 모든 원본 객체들에서 고유함
4. Context : 객체의 동작은 Flyweight 클래스에 남아있으며, Context 객체에서 연결된 Flyweight 객체를 사용 
5. Client : Flyweight 객체들의 *"공유한 상태"*를 저장하거나 계산
6. FlyweightFactory
    1. Flyweight 객체들의 풀을 관리
    2. 이 팩토리로 인해 클라이언트들은 Flyweight를 직접 만들지 않는 대신 원하는 Flyweight의 *"고유한 상태"*의 일부를 전달하여 호출
    3. 팩토리에서 이전에 생성된 Flyweight 들을 확인하고 검색 기준과 일치하는 기존 Flyweight를 반환하거나 없을 경우 새로 생성

## 3. 사용

1. Flyweight 패턴은 프로그램이 많은 수의 객체들을 지원해야 하는데 사용할 수 있는 RAM을 거의 다 사용했을 때만 사용
    - 앱이 수많은 유사 객체들을 생성해야 할 때
    - 사용할 수 있는 모든 RAM을 소모할 때
    - 객체들에 여러 중복 상태들이 포함되어 있으며, 이 상태들이 추출된 후 객체 간에 공유될 수 있을 때

## 4. Pros and Cons

### 4.1. Pros

- 프로그램에 유사한 객체들이 많다고 가정하면 많은 RAM을 절약할 수 있음

### 4.2. Cons

- Flyweight 메소드를 호출할 때마다 context 데이터의 일부를 다시 계산해야 한다면 CPU와 RAM을 교환한다고 볼 수 있음
- 코드가 복잡해지므로 새로운 팀원들은 개체의 상태가 분리되었는지 궁금해 할 것 임

## 5. 코드로 알아보기

```cpp
#include <iostream>
#include <string>
#include <unordered_map>

// 공유한 상태
struct SharedState
{
  std::string brand_;
  std::string model_;
  std::string color_;

  SharedState(const std::string &brand, const std::string &model, const std::string &color)
      : brand_(brand), model_(model), color_(color)
  {
  }

  friend std::ostream &operator<<(std::ostream &os, const SharedState &ss)
  {
      return os << "[ " << ss.brand_ << " , " << ss.model_ << " , " << ss.color_ << " ]";
  }
};

// 고유한 상태
struct UniqueState
{
  std::string owner_;
  std::string plates_;

  UniqueState(const std::string &owner, const std::string &plates)
      : owner_(owner), plates_(plates)
  {
  }

  friend std::ostream &operator<<(std::ostream &os, const UniqueState &us)
  {
      return os << "[ " << us.owner_ << " , " << us.plates_ << " ]";
  }
};

// shared_state를 공통의 부분으로 저장하고 나머지 상태는 메소드를 통해 accept
class Flyweight
{
private:
  SharedState *shared_state_;

public:
  Flyweight(const SharedState *shared_state) : shared_state_(new SharedState(*shared_state))
  {
  }
  Flyweight(const Flyweight &other) : shared_state_(new SharedState(*other.shared_state_))
  {
  }
  ~Flyweight()
  {
      delete shared_state_;
  }
  SharedState *shared_state() const
  {
      return shared_state_;
  }
  void Operation(const UniqueState &unique_state) const
  {
      std::cout << "Flyweight: Displaying shared (" << *shared_state_ << ") and unique (" << unique_state << ") state.\n";
  }
};

// FlyweightFactoy는 Flyweight 객체를 생성 및 관리
class FlyweightFactory
{
    /**
     * @var Flyweight[]
     */
private:
  std::unordered_map<std::string, Flyweight> flyweights_;

  // Flyweight의 shared state에서 가지는 string 해쉬를 리턴
  std::string GetKey(const SharedState &ss) const
  {
      return ss.brand_ + "_" + ss.model_ + "_" + ss.color_;
  }

public:
  FlyweightFactory(std::initializer_list<SharedState> share_states)
  {
      for (const SharedState &ss : share_states)
      {
          this->flyweights_.insert(std::make_pair<std::string, Flyweight>(this->GetKey(ss), Flyweight(&ss)));
      }
  }

  // shared_state key를 가지고 기존의 Flyweight가 있으면 return 없으면 새로 객체를 만들고 map에 저장
  Flyweight GetFlyweight(const SharedState &shared_state)
  {
      std::string key = this->GetKey(shared_state);
      if (this->flyweights_.find(key) == this->flyweights_.end())
      {
          std::cout << "FlyweightFactory: Can't find a flyweight, creating new one.\n";
          this->flyweights_.insert(std::make_pair(key, Flyweight(&shared_state)));
      }
      else
      {
          std::cout << "FlyweightFactory: Reusing existing flyweight.\n";
      }
      return this->flyweights_.at(key);
  }
  // 현재 map에 가지고 있는 Flyweights를 출력
  void ListFlyweights() const
  {
      size_t count = this->flyweights_.size();
      std::cout << "\nFlyweightFactory: I have " << count << " flyweights:\n";
      for (std::pair<std::string, Flyweight> pair : this->flyweights_)
      {
          std::cout << pair.first << "\n";
      }
  }
};

// ...
void AddCarToPoliceDatabase(
    FlyweightFactory &ff, const std::string &plates, const std::string &owner,
    const std::string &brand, const std::string &model, const std::string &color)
{
  std::cout << "\nClient: Adding a car to database.\n";
  // 공유한 상태를 찾는 메소드
  const Flyweight &flyweight = ff.GetFlyweight({brand, model, color});

  // 클라이언트 코드는 상태를 저장하거나 계산하여 Flyweight 메소드에 전달
  flyweight.Operation({owner, plates});
}

int main()
{
  // Client code
  FlyweightFactory *factory = new FlyweightFactory({
    {"Chevrolet", "Camaro2018", "pink"},
    {"Mercedes Benz", "C300", "black"},
    {"Mercedes Benz", "C500", "red"},
    {"BMW", "M5", "red"},
    {"BMW", "X6", "white"}
  });
  factory->ListFlyweights();

  AddCarToPoliceDatabase(*factory,
                          "CL234IR",
                          "James Doe",
                          "BMW",
                          "M5",
                          "red");

  AddCarToPoliceDatabase(*factory,
                          "CL234IR",
                          "James Doe",
                          "BMW",
                          "X1",
                          "red");
  factory->ListFlyweights();
  delete factory;
  std::cout << "Check!!";
  return 0;
}
```

## 참고
[wikipedia](https://en.wikipedia.org/wiki/Flyweight_pattern)  
[refactoring.guru](https://refactoring.guru/design-patterns/flyweight)  

<br>

    This is personal diary for study documents.
    Please comment if I'm wrong or missing something else 😄. 

[Top](#){: .btn .btn--primary }{: .align-right}