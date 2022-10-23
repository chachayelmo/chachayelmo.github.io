---
published: true
title:  "[Programming] C++ New"
excerpt: "C++에 대해 알아보기, New"

categories:
  - Cpp
tags:
  - [C++, Cpp, New, PlacementNew]

toc: true
toc_sticky: true
 
date: 2022-10-16
last_modified_at: 2022-10-16
---

## 1. C++ New 방법
- 2가지 방법
    1. New = 메모리 할당 + 생성자 호출(객체생성)
    2. Placement New = 있는 메모리에 생성자만 호출 (e.q. “new (p) Point”)

## 2. 코드로 알아보기
- [web_compiler](https://godbolt.org/) 에서 확인

### 2.1. New의 동작 방식 
1. operator new()라는 함수로 메모리 할당 
2. 1번이 성공하면 생성자 호출 
3. 주소를 해당 타입으로 반환 

```cpp
#include <iostream> 
using namespace std; 

class Point 
{ 
    int x, y; 
public: 
    Point() { cout << "Point()" << endl; } 
    ~Point() { cout << "~Point()" << endl; } 
}; 
  
int main() 
{ 
    // A. New 하는 방법
    // Point* p1 = new Point; 
    // delete p1;
  
    // B. New를 단계별로 하는 방법
    // 1. 생성자 호출없이 메모리만 할당하는 방법 
    Point* p2 = static_cast<Point*>(operator new(sizeof(Point))); 

    // 2. 할당된 메모리에 생성자를 호출 
    new (p2) Point; 

    // 3. 소멸자 호출 
    p2->~Point(); 

    // 4. 메모리 해지
    operator delete(p2); // 메모리 해지 
}
```

### 2.2. new 재정의
- new가 메모리할당할때 사용하는 operator new()는 재정의 가능
- operator new()함수는 2개 이상 가능, 단 첫번째 인자는 반드시 size_t

```cpp
#include <iostream> 
using namespace std; 
  
// 1. 인자 1개일 경우
void* operator new(size_t sz) 
{ 
    cout << "my new : " << sz << endl; 
    return malloc(sz); 
} 

// 2. 인자 여러개일 경우
// 첫번째 인자는 size_t
void* operator new(size_t sz, const char* s, int n) 
{ 
    cout << "my new2 : " << s << " " << sz << endl; 
    return malloc(sz); 
} 

void operator delete(void* p) 
{ 
    free(p); 
} 

int main() 
{ 
    int* p1 = new int;            // 인자가 1개
    int* p2 = new("AAA", 10) int; // 인자가 3개
 
    delete p1;
    delete p2;
}
```

### 2.3. Placement New
- 메모리 할당이 아닌 기존 메모리에 생성자 호출을 위한 것
- C++ 표준에서 placement new라고 지칭

```cpp
#include <iostream> 
using namespace std; 
  
class Point 
{ 
    int x, y; 
public: 
    Point()  { cout << "Point()" << endl; } 
    ~Point() { cout << "~Point()" << endl; } 
}; 

int main() 
{ 
    Point p; 

    new(&p) Point; // 기존메모리에 사용
    p.~Point();// 소멸자의 명시적 호출 가능
}
```

### 2.4. Placement New 사용

```cpp
#include <iostream> 
using namespace std;

class Point
{
    int x, y;
public:
    Point(int a, int b) {
        cout << "Point(x, y)" << endl;
        x = a;
        y = b;
    }
};

int main() 
{ 
    // Heap에 Point 한개를 만들기
    Point* p1 = new Point(0, 0); 

    // 힙에 Point 10개를 만들기
    // Point* p2 = new Point[10]; // error

    // 10개에 대한 메모리 할당
    Point* p3 = static_cast<Point*> ( 
        operator new(sizeof(Point) * 10)); 

    // 생성자 호출
    for (int i = 0; i < 10; i++) 
        new( &p3[i] ) Point(0, 0); 

    // 소멸자 호출
    for (int i = 0; i < 10; i++) 
        p3[i].~Point(); 

    // 메모리 해지 
    operator delete(p3);
}
```

### 2.5. Vector에서의 사용

```cpp
#include <iostream> 
#include <vector> 
using namespace std; 
  
int main() 
{ 
    vector<int> v(10, 0); // 10개를 0으로 초기화 
    v.resize(7); 
  
    cout << v.size() << endl;     // 7 
    cout << v.capacity() << endl; // 10 
  
    v.emplace_back(0);
  
    cout << v.size() << endl;     // 8 
    cout << v.capacity() << endl; // 10 
  
    v.shrink_to_fit(); // 여분의 메모리 제거 
  
    cout << v.size() << endl;     // 8 
    cout << v.capacity() << endl; // 8 
  
    v.emplace_back(0); 
  
    cout << v.size() << endl;     // 9 
    cout << v.capacity() << endl; // 12 
} 
```

### 2.6. Vector에서의 사용 2nd

```cpp
#include <iostream> 
#include <vector> 
using namespace std; 

class Point 
{
public: 
    Point()  { cout << "Point() " << endl; } 
    ~Point() { cout << "~Point()" << endl; } 
}; 
int main() 
{ 
    vector<Point> v(10); 

    cout << '\n';
    v.resize(7); // 줄어든 3개의 객체에 대해 소멸자 호출이 불림
    cout << '\n';

    cout << "capacity " << v.capacity() << '\n';

    v.resize(8); // 8번째 메모리는 이미 있고, 생성자만 호출해서 다시 Point를 붙임
    cout << '\n';
} 
```

## 참고
codenuri 강석민 강사 강의 내용기반으로 정리한 내용입니다.  
[코드누리](https://github.com/codenuri)

<br>

    This is personal diary for study documents.
    Please comment if I'm wrong or missing something else 😄. 

[Top](#){: .btn .btn--primary }{: .align-right}