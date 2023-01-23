---
published: true
title:  "[Programming] STL stream iterator"
excerpt: "stream iterator에 대해 알아보기"

categories:
  - Cpp
tags:
  - [C++, Cpp, Iterator, Stream]

toc: true
toc_sticky: true
 
date: 2023-01-23
last_modified_at: 2023-01-23
---

## 1. stream iterator

- 입/출력 스트림에서 요소를 읽거나 쓰기 위한 반복자
```cpp
#include <iterator>
```
- 4가지 종류의 반복자 제공
    - ostream_iterator
        - basic_ostream
        - 서식화된 출력
    - ostreambuf_iterator
        - basic_ostreambuf
        - CharT 출력
    - istream_iterator
        - basic_istream
        - 서식화된 입력
    - istreambuf_iterator
        - basic_istreambuf
        - CharT 입력
    
    ```cpp
    #include <iostream>
    #include <list>
    #include <iterator>
    using namespace std;
    
    int main()
    {
        int n = 10;
    
        cout << n << endl;
    
        ostream_iterator<int> p(cout, ", ");
    
        *p = 20; // cout << 20 << ", "
        *p = 30; // cout << 30 << ", "
    
        list<int> s = {1,2,3};
    
        copy(begin(s), end(s), p);
    
        fill_n(p, 3, 0);
    }
    ```
    
    ## 2. ostream_iterator
    
- 생성 방법
    
    ```cpp
    ostream_iterator(ostream_type& stream, const CharT* delimeter)
    ostream_iterator(ostream_type& stream)
    ```
    
- 연산
    - ++
    - *
    - =

```
#include <iostream>
#include <iterator>
#include <fstream>
using namespace std;

int main()
{
    ofstream f("a.txt");

    ostream_iterator<int> p(f, ", ");

    // ostream_iterator<int> p(cout); //Null

    *p = 20;
    ++p;
    *p = 30;
}
```

## 3. ostreambuf_iterator

- rdbuf()
    - stream 객체가 사용하는 streambuf의 포인터를 반환하는 stream의 멤버 함수
- basic_streambuf<>::sputc()
    - streambuf에 한 문자를 출력하는 함수
- ostream_iterator
    - stream(cout)을 사용해서 화면 출력 - 서식화된 출력
- ostreambuf_iterator
    - streambuf를 사용해서 화면 출력 - 문자 타입만 출력

![image](https://user-images.githubusercontent.com/23397039/214056975-1d6d6b29-50d5-46d2-87f5-b9cc566541f2.png){: .align-center}

```cpp
#include <iostream>
#include <iterator>
using namespace std;

int main()
{
    // ostreambuf_iterator<int> p(cout); // 에러, char형만 받을 수 있음
    ostreambuf_iterator<char> p(cout);

    *p = 65; // 아스키코드로 변환 A

    streambuf* buf = cout.rdbuf();
    buf->sputc(65); // A를 화면 출력

    ostreambuf_iterator<char> p(cout);
    *p = 'Z';
    ostreambuf_iterator<char> p2(cout.rdbuf());
    *p2 = 'A';

    ostreambuf_iterator<wchar_t> p3(wcout.rdbuf());
    *p3 = L'B';
}
```

## 4. ostream_iterator 구현해보기

```cpp
#include <iostream>
#include <iterator>
using namespace std;

template<typename T,
         typename CharT = char,
         typename Traits = char_traits<CharT>>
class eostream_iterator
{
    // ostream* stream;
    basic_ostream<CharT, Traits>* stream;
    const CharT* delimeter;
public:
    using iterator_category = output_iterator_tag;
    using value_type = void;
    using pointer = void;
    using reference = void;
    using difference_type = void;

    using char_type = CharT;
    using traits_type = Traits;
    using ostream_type = basic_ostream<CharT, Traits>;

    eostream_iterator(ostream& os, const CharT* const deli = 0)
        : stream(&os), delimeter(deli) 
    {}

    eostream_iterator& operator*() { return *this; }
    eostream_iterator& operator++() { return *this; }
    eostream_iterator& operator++(int) { return *this; }
    eostream_iterator& operator=(const T& v) {
        *stream << v;
        if (delimeter != 0)
            *stream << delimeter;
        return *this;
    }  
};

int main()
{
    eostream_iterator<int> p(cout, ", ");
    *p = 10; // ( p.operator*() ).operator=(10)

    istream_iterator<int> p1(cin);
    int n = *p1;

    cout << n << endd;
}
```

## 5. istream_iterator

- 입력 받을 때
    - cin : 버퍼에 입력된 whitespace를 제외하고 입력 받음
    - streambuf에서 직접 꺼내오면 whitespace를 입력 받을 수 있음

## 참고
codenuri 강석민 강사 강의 내용기반으로 정리한 내용입니다.  
[코드누리](https://github.com/codenuri)  

<br>

    This is personal diary for study documents.
    Please comment if I'm wrong or missing something else 😄. 

[Top](#){: .btn .btn--primary }{: .align-right}