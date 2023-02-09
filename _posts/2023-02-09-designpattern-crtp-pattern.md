---
published: true
title:  "[Design Pattern] CTRP in C++"
excerpt: "다양한 디자인 패턴에 대해 알아보기"

categories:
  - Algorithm
tags:
  - [DesignPattern, CTRP, Pattern]

toc: true
toc_sticky: true
 
date: 2023-02-09
last_modified_at: 2023-02-09
---

## 1. CRTP

- Curiously Recurring Template Pattern : 묘하게 되풀이되는 템플릿 패턴
- 기반 클래스가 템플릿인데 파생클래스를 만들 때 자신의 이름을 템플릿 인자로 전달하는 기술

## 2. 코드로 알아보기

- 기본적으로 파생클래스에 자신의 이름을 템플릿으로 전달

```cpp
#include <iostream>  
#include <vector>  
using namespace std;  

class Window  
{  
public:  
  void msgLoop()  
  {
    cout << "Window" << endl;
  }  
  void Click() { cout << "0" << endl; }  
};  
   
template<typename T> class MsgWindow : public Window  
{  
public:  
  void msgLoop()
  {   
    static_cast<T*>(this)->Click();  
  }  
  void Click()  
  {   
    cout << "MsgWindow" << endl;   
  }  
};  

// 파생클래스에 자신의 이름을 템플릿으로 전달
class MyWindow : public MsgWindow<MyWindow> 
{  
public:  
  void Click() { cout << "MyWindow" << endl; }  
};  
   
class MyWindow2 : public MsgWindow<MyWindow2>  
{  
public:  
  void Click() { cout << "MyWindow2" << endl; }  
};  
   
int main()  
{  
  vector<Window*> v;
  MyWindow w;  
  w.msgLoop(); // MyWindow

  MyWindow2 w2;  
  w2.msgLoop(); // MyWindow2

  Window* p = new MyWindow;  
  p->msgLoop(); // Window
}
```

- 아래 코드와 같이 특정 클래스 형태의 객체가 얼마나 생성된 지 추적이 가능

```cpp
#include <iostream> 
using namespace std; 
  
template<typename T> class Car 
{ 
  // 초기화를 위해 static으로 선언
  static int cnt; 
public: 
  Car() { ++cnt; } 
  ~Car() { --cnt; } 

  static int getCount() { return cnt; } 
}; 

template<typename T> int Car<T>::cnt = 0; 
  
// 다른 타입의 부모를 만듬 
class Truck : public Car<Truck> 
{ 
}; 
class Bike : public Car<Bike> 
{ 
}; 
  
int main() 
{ 
  Truck t1, t2; 
  Bike b1, b2, b3; 
  cout << t1.getCount() << endl; // 2
  cout << b1.getCount() << endl; // 3
}
```

## 3. CTRP Singleton

```cpp
#include <iostream> 
using namespace std;

template<typename T> class Singleton 
{ 
protected: 
  Singleton() = default; 

  Singleton(const Singleton&) = delete; 
  void operator=(const Singleton&) = delete; 
public: 
  static T& getInstance() 
  { 
    static T instance; 
    return instance; 
  }
}; 
  
class Keyboard : public Singleton<Keyboard> 
{
public:
  Keyboard() { cout << "Keyboard()" << endl; }
  void print() { cout << "Keyboard" << endl; }
}; 
  
class Cursor: public Singleton<Cursor> 
{
public:
  Cursor() { cout << "Cursor()" << endl; }
  void print() { cout << "Cursor" << endl; }
}; 

int main() 
{ 
  Cursor& c1 = Cursor::getInstance(); 
  Keyboard& k1 = Keyboard::getInstance();

  Cursor& c2 = Cursor::getInstance(); 
  Keyboard& k2 = Keyboard::getInstance();

  c2.print();
  k2.print();
}
```

## 참고
codenuri 강석민 강사 강의 내용기반으로 정리한 내용입니다.  
[코드누리](https://github.com/codenuri)  

<br>

    This is personal diary for study documents.
    Please comment if I'm wrong or missing something else 😄. 

[Top](#){: .btn .btn--primary }{: .align-right}v