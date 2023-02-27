---
published: true
title:  "[Design Pattern] Iterator in C++"
excerpt: "다양한 디자인 패턴에 대해 알아보기"

categories:
  - DesignPattern
tags:
  - [DesignPattern, Iterator, Pattern]

toc: true
toc_sticky: true
 
date: 2023-02-17
last_modified_at: 2023-02-17 
---

## 1. iterator

### 1.1. 의도

- iterator는 collection의 요소들의 표현(list, stack, tree 등)을 노출하지 않고 하나씩 순회할 수 있도록 하는 행동 디자인 패턴

### 1.2. 문제

- collection은 프로그래밍에서 가장 많이 사용되는 데이터 유형이며 그룹의 컨테이너
- 같은 collection을 여러가지 방법들로 순회가 가능
- collection의 주요 책임은 효율적인 데이터 저장, 더 많은 순회 알고리즘들을 추가하여 동작
- 반면 클라이언트 코드는 요소가 어떻게 저장되는지 관심을 두지 않음

### 1.3. 해결

- iterator 패턴의 주된 아이디어는 collection의 순회 동작을 iterator 객체로 추출

![image](https://user-images.githubusercontent.com/23397039/219568046-80c50333-3000-4675-b922-0c3726f49b7b.png){: .align-center}

- iterator 객체는 알고리즘 자체를 구현하는 것도 포함하지만 순회에 대한 세부 정보들(현재 위치, 남은 요소의 수 등) 을 캡슐화, 여러 iterator들이 독립적으로 동시에 같은 collection을 통과할 수 있음
- 모든 iterator는 같은 인터페이스를 구현해야 함

## 2. 구조

![image](https://user-images.githubusercontent.com/23397039/219567972-ec7b714e-f130-4562-9eed-41c0b2007863.png){: .align-center}

- **iterator** : 인터페이스이며 collection의 순회에 필요한 작업들을 선언
- **ConcreteIterator** : 순회를 위한 특정 알고리즘들을 구현
- **Collection** : 인터페이스이며, iterator들을 가져오기 위해 하나 이상의 메소드를 선언
- **ConcreteCollection** : 클라이언트가 요청할 때마다 특정 iterator 객체를 반환
- **Client** : iterator와 collection 인터페이스를 통해 원하는 동작을 실행

## 3. 사용

- collection이 내부에 복잡한 데이터 구조가 있지만 이 구조의 복잡성을 보안이나 편의성의 이유로 클라이언트들로부터 숨기고 싶을 때 사용
    - 복잡한 데이터 구조와 작업 시의 세부 사항을 캡슐화하여 숨기고 클라이언트에게 요소들을 접근할 수 있는 몇가지 간단한 메소드를 제공
- 앱에서 순회 코드의 중복을 줄일 때 사용
- 코드가 다른 데이터 구조들을 순회할 수 있기를 원하거나 이러한 구조들의 유형을 미리 알 수 없을 때 사용
    - collection과 iterator 에 대해 여러 개의 일반 인터페이스를 제공

## 4. Pros and Cons

- Pros
    - Single Responsibility Principle, 덩치가 큰 순회 알고리즘들을 별도의 클래스로 추출
    - Open/Closed Principle, 새로운 유형의 collection/iterator 추가 가능
    - 각 iterator 객체는 고유한 state를 포함하기 때문에 하나의 collection을 병렬로 순회 가능
    - 순회를 지연하거나 계속할 수 있음
- Cons
    - 앱이 단순한 collection들만 작동하는 경우 iterator 패턴 적용이 과도할 수 있음
    - iterator를 사용하는 것이 일부 특수한 collection들의 요소들을 직접 탐색하는 것보다 덜 효율적일 수 있음
    

## 5. 코드로 알아보기

```cpp
#include <iostream>
#include <string>
#include <vector>

// C++의 경우 이미 STL을 제공
template <typename T, typename U>
class Iterator {
 public:
  typedef typename std::vector<T>::iterator iter_type;
  Iterator(U *p_data, bool reverse = false) : m_p_data_(p_data) {
    m_it_ = m_p_data_->m_data_.begin();
  }

  void First() {
    m_it_ = m_p_data_->m_data_.begin();
  }

  void Next() {
    m_it_++;
  }

  bool IsDone() {
    return (m_it_ == m_p_data_->m_data_.end());
  }

  iter_type Current() {
    return m_it_;
  }

 private:
  U *m_p_data_;
  iter_type m_it_;
};

// Collection/Container
template <class T>
class Container {
  friend class Iterator<T, Container>;

 public:
  void Add(T a) {
    m_data_.push_back(a);
  }

  // 특정 Iterator를 가지는 메소드
  Iterator<T, Container> *CreateIterator() {
    return new Iterator<T, Container>(this);
  }

 private:
  std::vector<T> m_data_;
};

// Data
class Data {
 public:
  Data(int a = 0) : m_data_(a) {}

  void set_data(int a) {
    m_data_ = a;
  }

  int data() {
    return m_data_;
  }

 private:
  int m_data_;
};

// 클라이언트 코드
void ClientCode() {
  std::cout << "________________Iterator with int______________________________________" << std::endl;
  Container<int> cont;

  for (int i = 0; i < 10; i++) {
    cont.Add(i);
  }

  Iterator<int, Container<int>> *it = cont.CreateIterator();
  for (it->First(); !it->IsDone(); it->Next()) {
    std::cout << *it->Current() << std::endl;
  }

  Container<Data> cont2;
  Data a(100), b(1000), c(10000);
  cont2.Add(a);
  cont2.Add(b);
  cont2.Add(c);

  std::cout << "________________Iterator with custom Class______________________________" << std::endl;
  Iterator<Data, Container<Data>> *it2 = cont2.CreateIterator();
  for (it2->First(); !it2->IsDone(); it2->Next()) {
    std::cout << it2->Current()->data() << std::endl;
  }
  delete it;
  delete it2;
}

int main() {
  ClientCode();
  return 0;
}
```


## 참고
[wikipedia](https://en.wikipedia.org/wiki/Iterator_pattern)  
[refactoring.guru](https://refactoring.guru/design-patterns/iterator)  

<br>

    This is personal diary for study documents.
    Please comment if I'm wrong or missing something else 😄. 

[Top](#){: .btn .btn--primary }{: .align-right}