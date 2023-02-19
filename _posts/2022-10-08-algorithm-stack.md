---
published: true
title:  "[Data structure] 자료구조 - 스택(Stack)"
excerpt: "다양한 자료구조에 대해 알아보기"

categories:
  - Algorithm
tags:
  - [자료구조, 스택, DataStructure, stack]

toc: true
toc_sticky: true
author: chachayelmo
sitemap:
  changefreq : daily
  priority : 1.0
date: 2022-10-11
last_modified_at: 2022-10-11
---

## 3. Stack

- 스택
- LIFO, 가장 나중에 쌓은 데이터를 가장 먼저 뺄 수 있는 데이터 구조

![stack.png](https://user-images.githubusercontent.com/23397039/194865341-4e181a69-ce01-40e1-8ac5-65bd045e2c6f.png){: .align-center}

- push - 스택에 값을 넣음
- pop - 스택에서 값을 뺌
- 컴퓨터 내부의 프로세스 구조의 함수 동작 방식
- 스택 구조와 프로세스 스택
    - recursive는 스택

- C++ 스택 함수
    - top() : 최상위 데이터(맨 마지막에 넣은 값)를 반환
    - empty() : element가 있는지 조사
    - size() : 현재 element의 갯수를 반환
    - push(a) : a element를 추가
    - pop() : 맨 뒤의 element를 삭제

## 3.1. C++ 구현
- [web_compiler](https://godbolt.org/) 에서 확인

```cpp
#include <iostream>
#include <stack> 

int main(){
	std::stack<int> s;

	// 값 추가
	s.push(2);
	s.push(5);
	s.push(3);
	s.push(7);

  // 사이즈 확인
	std::cout << "stack size: " << s.size() << '\n';

	// 맨 마지막 원소 반환
	std::cout << "top element: " << s.top() << '\n';

    while (!s.empty()) {
        std::cout << s.top() << '\n';
        // 삭제
        s.pop();
    }

	// 빈 값 체크
	std::cout << "Is it empty?: " << (s.empty() ? "Yes" : "No") << '\n';

	return 0;

}
```

## 3.2. Python 구현
- [web_python_compiler](https://www.onlinegdb.com/online_python_compiler) 에서 확인

```python
def main():
    '''
    *** List 사용해서 쉽게 구현 가능
    '''
    stack = list()

    # 추가
    stack.append(3)
    stack.append(5)
    stack.append(2)
    stack.append(7)
    print("Stack add ", stack)

    # 제거
    top = stack.pop()
    print("revmoed val ", top)

    # 사이즈
    print("size ", len(stack))

    # Top 값 체크
    print("top ", stack[-1])
    
    # 빈 값 체크
    print("isEmpty? ", len(stack) == 0)

if __name__ == "__main__":
	main()

```

## 참고
[잔재미코딩](https://www.fun-coding.org/DS&AL1-2.html)  
[CPP Stack](https://coding-factory.tistory.com/597)  
[CPP Reference](https://en.cppreference.com/w/cpp/container/stack)

<br>

    This is personal diary for study documents.
    Please comment if I'm wrong or missing something else 😄. 

[Top](#){: .btn .btn--primary }{: .align-right}