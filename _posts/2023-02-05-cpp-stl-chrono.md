---
published: true
title:  "[C++] STL chrono"
excerpt: "chrono 에 대해 알아보기"

categories:
  - Cpp
tags:
  - [C++, Cpp, STL, Ratio, Duration, Chrono]

toc: true
toc_sticky: true
author: chachayelmo
sitemap:
  changefreq : daily
  priority : 1.0
comment: true
date: 2023-02-05
last_modified_at: 2023-02-05
---

## 1. ratio

- 컴파일 시간 분수 값을 나타내는 템플릿
- <ratio> 헤더
- 2개의 static member data로 구성
    - num - 분자
    - den - 분모
- 분자, 분모는 컴파일 타임 연산을 통해 저장
- ratio 자체가 런타임에 메모리에 보관하는 값이 없기 때문에 empty이고, 컴파일 시간에 사용되는 상수 값

```cpp
#include <iostream>
#include <ratio>
using namespace std;

/* 간단한 구현, 실제 STL의 구현은 더 많은 것들을 포함
template<intmax_t _Nx, intmax_t _Dx = 1>
struct Ratio
{
    static constexpr intmax_t num = _Nx;
    static constexpr intmax_t den = _Dx;

    typedef Ratio<num, den> type;
};
*/

int main()
{
    ratio<2, 4> r1; // 2/4 -> 1/2

    cout << sizeof(r1) << endl; // 컴파일 시간에 연산되므로 empty로 크기는 1

    cout << r1.num << endl;
    cout << r1.den << endl;

    cout << ratio<2, 4>::num << endl;
    cout << ratio<2, 4>::den << endl;
}
```

- 연산
    - Arithmetic = ratio_add / ratio_subtract / ratio_multiply / rati_divide
    - Comparison = ratio_equal / ratio_not_equal / ratio_less / ratio_greater 등
- constants
    - yocta, zepto, pico, nano, micro, milli, centi 등등 자주 사용되는 단위는 상수로 가지고 있음

```cpp
#include <iostream>
#include <ratio>
using namespace std;

int main()
{
    ratio_add< ratio<1,4>, ratio<2,4> > r2; // 3/4

    cout << r2.num << endl;
    cout << r2.den << endl;

    ratio<1, 1000> r3; // milli
    ratio<1000, 1> r4; // kilo
    milli m;
    kilo k;
    cout << m.num << endl; // 1
    cout << m.den << endl; // 1000
    
}
```

## 2. duration

- <chrono> 헤더
- namespace chrono
- ratio로 표현되는 단위에 대한 값을 보관
- 오직 하나의 값만 보관
- duration을 사용하면 단위에 맞게 자동으로 연산이 수행

```cpp
#include <iostream>
#include <chrono>
#include <ratio>
using namespace std;
using namespace chrono;

int main()
{
    double dist = 3; // 3m, 3km, 3cm?

    duration<double, ratio<1,1>> d1(3); // 3m
    // duration<double, ratio<1,1000>> d2(d1); // 3000 milli
    duration<double, milli> d2(d1); // 3000 milli

    cout << d2.count() << endl; // 3000

    // duration<double, ratio<1000,1>> d3(d1); // km
    duration<double, kilo> d3(d1); // km

    cout << d3.count() << endl; // 0.003

    using MilliMeter = duration<double, milli>;
    using KiloMeter = duration<double, kilo>;
    using Meter = duration<double, ratio<1,1>>;

    Meter m(3);
    KiloMeter km(m);    
}
```

- 캐스팅
    - duration_cast : Convers a duration to another
    - floor : rounding down, C++17
    - ceil : rounding up, C++17
    - round : rounding to nearest, ties to even, C++17
    - abs : absolute value, C++17

```cpp
#include <iostream>
#include <chrono>
#include <ratio>
using namespace std;
using namespace chrono;

int main()
{
    using MilliMeter = duration<int, milli>;
    using KiloMeter = duration<int, kilo>;
    using Meter = duration<int, ratio<1,1>>;

    Meter m(600);
    MilliMeter mm(m);
    // KiloMeter km(m); // 0.6 -> 0 or 1 error

    KiloMeter km = duration_cast<KiloMeter>(m); // 버림
    KiloMeter km2 = round<KiloMeter>(m); // 반올림

    cout << km.count() << endl;  // 0
    cout << km2.count() << endl; // 1
}
```

## 3. chrono

- 시간 관련 타입
    - nanoseconds
    - microseconds
    - milliseconds
    - minutes
    - hours

```cpp
#include <iostream>
#include <chrono>
using namespace std;
using namespace chrono;

int main()
{
    /* 이미 STL에 있음
    using seconds = duration<int>;
    using minutes = duration<int, ratio<60,1>>;
    using hours = duration<int, ratio<3600,1>>;
    using milliseconds = duration<int, milli>;
    */

    hours h(1);
    minutes m(h); // 60
    seconds s(h); // 3600
    cout << m.count() << endl;
    cout << s.count() << endl;

    // hours h2(s); // error
    hours h2 = duration_cast<hours>(s); 

    using days = duration<int, ratio<3600*24, 1>>;

    days d(1);
    minutes m2(d);
    cout << m2.count() << endl; // 60 * 24
}
```

- 초기화 방법
    - explicit 생성자 이므로 direct initialization
    - 시간 관련 user defined literal 사용 - 복사 생성 가능
        - operator””h
        - operator””min
        - operator””s
        - operator””ms
        - operator””us
        - operator””ns

```cpp
#include <iostream>
#include <chrono>
using namespace std;
using namespace chrono;

void foo( seconds s) {}

int main()
{
    seconds s1(3);  // ok
    // seconds s2 = 3; // error
    seconds s3 = 3s; // ok, seconds operator""s(3)

    seconds s4 = 3min;

    cout << s4.count() << endl; // 180

    // foo(3);  // error, 복사 초기화이므로
    foo(3s); // ok

    seconds s5 = 3min + 40s;
    cout <<  s5.count() << endl; // 220
}
```

- time_point
    - 기간의 시작과 경과 개수를 나타내는 타입

```cpp
#include <iostream>
#include <chrono>
#include <string>
using namespace std;
using namespace chrono;

int main()
{
    system_clock::time_point tp = system_clock::now();

    // 1980년 1월 1일 0시 기준
    nanoseconds ns = tp.time_since_epoch();

    cout << ns.count() << endl;

    hours h = duration_cast<hours>(ns);
    cout << h.count() << endl;

    // 현재 날짜 출력
    time_t t = system_clock::to_time_t(tp);
    string s = ctime(&t);
    cout << s << endl;
}
```

## 참고
codenuri 강석민 강사 강의 내용기반으로 정리한 내용입니다.  
[코드누리](https://github.com/codenuri)  

<br>

    This is personal diary for study documents.
    Please comment if I'm wrong or missing something else 😄. 

[Top](#){: .btn .btn--primary }{: .align-right}