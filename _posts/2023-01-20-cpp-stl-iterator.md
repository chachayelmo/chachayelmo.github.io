---
published: true
title:  "[Programming] C++ STL iterator"
excerpt: "iterator에 대해 알아보기"

categories:
  - Cpp
tags:
  - [C++, Cpp, Iterator]

toc: true
toc_sticky: true
 
date: 2023-01-20
last_modified_at: 2023-01-20
---

## 1. iterator

- 복합 개체의 내부 구조에 상관 없이 순차적으로 요소에 접근하기 위한 방법을 제공
- STL에서는 iterator처럼 동작하는 모든 것이 iterator임
- ++ 연산자로 이동 가능하고, * 연산자로 요소에 접근 가능
- iterator의 다양한 형태
    - Raw pointer
    - 컨테이너의 요소를 열거하기 위한 객체 ex) begin()
    - 스트림 iterator( stream iterator)
    - 삽입 iterator( insert iterator)
    - directory_iterator 등등

## 1.1. 코드와 함께 알아보기

- iterator type
    - 컨테이너이름<요소타입>::iterator
    - auto 사용
- iterator를 얻는 방법
    - 멤버 함수 사용 : begin() / end()
    - 일반 함수 사용 : STL 컨테이너와 배열 모두 사용 가능
    - size(), empty()
- past-the-end iterator
- end() 로 얻은 iterator는 마지막 다음 요소를 가리키므로 dereference(*)하면 안됨

```cpp
#include <iostream>
#include <list>
#include <vector>

using namespace std;

int main()
{
    // list<int> s = {1,2,3,4,5};
    // vector<int> s = {1,2,3,4,5};
    int s[5] = {1,2,3,4,5};

    //list<int>::iterator p1 = s.begin();
    // C++11 부터 auto 사용
    // auto p1 = s.begin(); // 배열은 멤버함수가 없어서 error
    auto p1 = begin(s); // 일반함수로 STL container와 배열 모두 사용 가능

    int n = size(s); // s.size() 대신 일반함수를 사용
    cout << n << endl;

    auto p2 = end(s);
    *p2 = 10 // runtime error
}
```

- iterator 무효화 (invalidate)
    - vector 등의 컨테이너의 내부 버퍼가 재 할당 될 때
    - iterator가 가리키던 요소가 제거 될 때
    - 컨테이너의 종류에 따라 무효화 되는 조건이 다름

```cpp
#include <iostream>
#include <list>
#include <vector>

using namespace std;

int main()
{
    vector<int> v = {1,2,3,4,5};
    auto p = begin(v);
    v.resize(100); // 버퍼 재할당
                   // p를 사용할 수 없음

    cout << *p << endl; // 쓰레기값

    list<int> v2 = {1,2,3,4,5};
    auto p2 = begin(v2);
    v2.resize(100); // 버퍼 재할당
                    // p2를 사용할 수 없음

    cout << *p2 << endl; // 1
}
```

- iterator의 구간 - range
    - [first, last) : 시작과 마지막 다음 요소를 가리키는 iterator의 쌍
    - 유효한 구간 : first부터 시작해서 ++ 연산으로 last에 도달할 수 있는 구간
    - 빈 구간 : first == last인 경우
- Copy 알고리즘
    - 하나의 구간의 내용을 다른 구간으로 복사하는 알고리즘
    - #include <algorithm>
    - 반복문의 일반화된 표현
    - 1번째 구간은 완전한 구간(first, last)가 전달되지만 2번째 구간은 시작만 전달

```cpp
#include <iostream>
#include <list>
#include <vector>
#include <algorithm>

using namespace std;

int main()
{
    // range
    list<int> s1 = {1,2,3};
    list<int> s2 = {4,5,6};
    auto p1 = begin(s1);
    auto p2 = end(s1);

    while ( p1 != p2){
        ++p1;
    }

    list<int> s3;
    if (s3.empty()) {}
    if (begin(s3) == end(s3)) {}

    // copy
    int x[5] = { 1,2,3,4,5 };
    int y[5] = {0, };

    list<int> l = {0, 0, 0, 0, 0};

    // for(int i = 0; i < 5; i++) {
    //     y[i] = x[i];
    // }

    // copy(x, x + 5, y);
    copy(x, x + 5, begin(l));

    for (auto& n : l){
        cout << n << ", ";
    }
};
```

## 2. iterator category

- iterator는 적용할 수 있는 연산에 따라 6가지로 분류
- 입력 반복자 (input iterator)
    - 입력(=*i), ++
- 출력 반복자 (output iterator)
    - 출력(*i=), ++
- 전진형 반복자 (forward iterator)
    - 입력, ++, multiple pass
- 양방향 반복자 (bidirectional iterator)
    - 입력, ++, --, multiple pass
- 임의 접근 반복자  (random access iterator)
    - 입력, ++, --, -, []
- 인접한 반복자  (contiguous iterator)
    - 입력, ++, --, -, [],
    - *(i+n) == *(std::addressof(*a) + n)

![image](https://user-images.githubusercontent.com/23397039/213634581-dee3ce9d-9f07-427e-91b0-be37152acefa.png){: .align-center}

![image](https://user-images.githubusercontent.com/23397039/213634631-f9221745-b78e-4f10-a184-29dddc355b67.png){: .align-center}

- sort는 vector는 가능하고 list가 불가능

```cpp
#include <iostream>
#include <list>
#include <vector>
#include <algorithm>
#include <forward_list>

using namespace std;

int main()
{
//    vector<int> v = {1,3,5,7,9,2,4,6,8,10};
   list<int> v = {1,3,5,7,9,2,4,6,8,10};

   sort(begin(v), end(v)); // error

   for (auto& n : v)
    cout << n << ",";
};
```

- list의 반복자 : ++, -- 모두 가능
- forward_list의 반복자 : ++ 만 가능

```cpp
#include <iostream>
#include <list>
#include <forward_list>

using namespace std;

int main()
{
    forward_list<int> s1 = {1, 2, 3};
    // list<int> s1 = {1, 2, 3};

    auto p = begin(s1);
    ++p;
    --p;
};
```

```cpp
/*
list iterator : ++ 연산은 제공되지만 + 연산은 제공되지 않음
vector iterator : ++과 +연산 모두 제공
*/

#include <iostream>
#include <list>
using namespace std;

int main()
{
    list<int> s1 = {1, 2, 3};
    auto i = begin(s1);

    int n = *i; // 입력
    *i = 20; // 출력

    ++i; // ok
    // i = i + 2; // error
};
```

- multipass guarantee란?
    - 2개의 iterator i1, i2에 대해서 다음을 만족하는지?
        - i1 == i2 라면 *i1 == *i2
        - i1 == i2 라면 ++i1 == ++i2
    - 2개 이상의 iterator가 컨테이너의 요소에 동일하게 접근함을 보장
    - list의 iterator는 multipass를 보장
    - stream iterator 와 insert iterator는 multipass를 지원하지 못함
    
    ```cpp
    #include <iostream>
    #include <list>
    
    using namespace std;
    
    int main()
    {
        list<int> s1 = {1, 2, 3};
        auto i1 = begin(s1);
        auto i2 = i1;
    
        // 아래 조건을 만족하면 multipass를 지원함
        if (i1 == i2)
        {
            cout << (*i1 == *i2) << endl;
            cout << (++i1 == ++i2) << endl;
        }
    };
    ```
    
- iterator category 쓰는 이유?
    - 다양한 알고리즘의 인자로 요구하는 iterator category가 무엇 인지를 알아야 함

```cpp
#include <iostream>
#include <list>
#include <vector>
#include <algorithm>
#include <forward_list>

using namespace std;

int main()
{
    vector<int> v = {1,2,3,4,5,6,7,8,9,10};

    // find의 최소 요구 조건은 입력 반복자
    auto p = find(begin(v), end(v), 5);

    // reverse는 양방향 반복자
    reverse(begin(v), end(v));

    // quick sort 는 뺄셈이 가능해야되서 임의 접근 반복자
    sort(begin(v), end(v));

    forward_list<int> s = {1,2,3};
    // reverse(begin(s), end(s)); // error

    list<int> s2 = {1,2,3};
    // sort(begin(s2), end(s2)); // error

    s2.sort(); // 멤버함수 sort ok, quick sort가 아님

    // v.sort(); // vector 멤버함수 sort 없음, 있을 이유가 없음

    for (auto& n : v)
        cout << n << ", ";
};
```

## 3. advance 함수

- #include <iterator>
- iterator를 N 만큼 전진(후진) 하는 함수

```cpp
#include <iostream>
#include <list>
#include <vector>
#include <iterator>
using namespace std;

int main()
{
    list<int> s = {1,2,3,4,5,6,7,8,9,10};

    auto p = begin(s);
    // iterator p를 5칸 전진하고 싶을 때
    // p = p + 5;
    advance(p, 5);

    cout << *p << endl;
};
```

## 4. iterator member type

- value_type = T
- differnce_type = unsigned integer type(using std::ptrdiff_t)
- reference =value_type&
- pointer = value_type*
- iterator_category = 컨테이너마다 다른 정의

```cpp
#include <iostream>
#include <list>
#include <vector>
#include <iterator>
using namespace std;

template <typename Category,
          typename T,
          typename Distance = std::ptrdiff_t,
          typename Pointer = T*,
          typename Reference = T&>
struct xiterator
{
    using iterator_category = Category;
    using valuetype = T;
    using pointer = Pointer;
    using reference = Reference;
    using difference_type = Distance;
};

template<typename T> class slist_iterator : 
    public xiterator<forward_iterator_tag, T>
{};

int main()
{
    
};
```

## 5. iterator traits

- 반복자타입::value_type

```cpp
#include <iostream>
#include <list>
#include <iterator>
using namespace std;

template<typename T>
typename T::value_type sum(T first, T last)
{
    typename T::value_type s = 0;
    while (first != last){
        s = s + *first;
        ++first;
    }
    return s;
}
int main()
{
    list<int> s= {1,2,3,4,5,6,7,8,9};

    int n = sum(begin(s), end(s));

    cout << n << endl;
};
```

- iterator의 2가지 형태
    - User Define type 으로 만들어진 iterator - value_type이 있음
    - Raw pointer - member type이 없기 때문에 value_type이 없음
    
![image](https://user-images.githubusercontent.com/23397039/213634735-51804b43-35ca-464c-a3eb-a9adeab64e65.png){: .align-center}

- iterator_traits
    - #include <iterator>
    - Raw pointer에 member type이 없다는 문제를 해결하기 위한 도구
    - 반복자의 value_type이 필요할 때 iterator_traits를 통해서 value_type을 사용
    - 사용방법
    - T::value_type
    - iterator_traits<T>::value_type

![image](https://user-images.githubusercontent.com/23397039/213634785-e99969cb-562c-4f9a-961c-e7d9c8d815bc.png){: .align-center}

```cpp
#include <iostream>
#include <list>
#include <iterator>
using namespace std;

/*
template<typename T> struct iterator_traits
{
    using value_type = typename T::value_type;
};

// 포인터 버전 부분 특수화
template<typename T> struct iterator_traits<T*>
{
    using value_type = T;
};
*/

template<typename T>
typename iterator_traits<T>::value_type sum(T first, T last)
{
    // T : int* - error
    // typename T::value_type s = 0;
    typename iterator_traits<T>::value_type s = 0;
    while (first != last){
        s = s + *first;
        ++first;
    }
    return s;
}

int main()
{
    int s[10] = {1,2,3,4,5,6,7,8,9};

    int n = sum(begin(s), end(s));

    cout << n << endl;
};
```

```cpp
template<class T> struct iterator_traits
{
    using iterator_category = typename T::iterator_category;
    using value_type = typename T::value_type;
    using difference_type = typename T::difference_type;
    using pointer = typename T::pointer;
    using reference = typename T::reference; 
};

// Raw Pointer
template<class T> struct iterator_traits<T*>
{
    using iterator_category = random_access_iterator_tag;
    using value_type = T;
    using difference_type = typename T::difference_type;
    using pointer = typename T::pointer;
    using reference = typename T::reference; 
};
```

## 참고
codenuri 강석민 강사 강의 내용기반으로 정리한 내용입니다.  
[코드누리](https://github.com/codenuri)  

<br>

    This is personal diary for study documents.
    Please comment if I'm wrong or missing something else 😄. 

[Top](#){: .btn .btn--primary }{: .align-right}