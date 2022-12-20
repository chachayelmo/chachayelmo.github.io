---
published: true
title:  "[Programming] C++ trivial"
excerpt: "C++에 대해 알아보기, trivial"

categories:
  - Cpp
tags:
  - [C++, Cpp, Trivial]

toc: true
toc_sticky: true
 
date: 2022-12-19
last_modified_at: 2022-12-19
---

## 1. Trivial
- Trivial : 사소한, 하찮은, 하는 일이 없다
- Trivial default constructor : 기본생성자가 하는 일이 없는 경우

- Trivial 조건
  1. user define 생성자가 없어야 함
  2. 가상함수가 없어야 함
  3. 가상 기반 클래스가 없어야 됨
  4. 초기화된 변수가 없어야 됨
  5. 상속을 받았다면 기반 클래스도 위의 조건을 만족해야 됨

## 2. 코드로 알아보기
- [web_compiler](https://godbolt.org/) 에서 확인

```cpp
#include <iostream>
#include <type_traits>
using namespace std;

class A
{
public:
  // A(); // 0
	// virtual void foo() {} // 0
  void foo() {} // 1
};

int main()
{
	cout << is_trivially_constructible<A>::value << endl;
}
```

```cpp
#include <iostream>
#include <cstring>
#include <type_traits>
using namespace std;

template<typename T> void copy_type(T* dst, T* src, int sz)
{
	if (is_trivially_copyable<T>::value)
	{
		cout << "복사 생성자가 trivial" << endl;
		memcpy(dst, src, sizeof(T) * sz);
	}
	else
	{
		cout << "복사 생성자가 trivial 하지 않은 경우" << endl;
		while (sz--)
		{
			new(dst) T(*src); // 복사 생성자 호출
			++dst, ++src;
		}
	}
}

int main()
{
  char s1[10] = "hello";
  char s2[10] = {0};

  copy_type(s1, s2, 10); // 모든 타입의 배열을 복사하는 함수
}
```

## 참고
codenuri 강석민 강사 강의 내용기반으로 정리한 내용입니다.  
[코드누리](https://github.com/codenuri)  
[cppreference_default_constructor](https://en.cppreference.com/w/cpp/language/default_constructor)

<br>

    This is personal diary for study documents.
    Please comment if I'm wrong or missing something else 😄. 

[Top](#){: .btn .btn--primary }{: .align-right}