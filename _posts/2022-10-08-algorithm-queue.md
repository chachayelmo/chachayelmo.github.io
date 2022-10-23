---
published: true
title:  "[Programming] 자료구조 - Queue"
excerpt: "다양한 자료구조에 대해 알아보기"

categories:
  - Algorithm
tags:
  - [DataStructure, Queue]

toc: true
toc_sticky: true
 
date: 2022-10-10
last_modified_at: 2022-10-10
---

## 1. Queue

- 큐
- FIFO
- 멀티 태스킹을 위한 프로세스 스케쥴링 방식을 구현하기 위해 많이 사용
- Enqueue : 큐에 데이터를 넣는 기능
- Dequeue: 큐에 데이터를 꺼내는 기능

![image](https://user-images.githubusercontent.com/23397039/194700788-f749a2db-f935-4351-8331-9e42dd84ca2f.png){: .align-center}

- C++ 큐 함수	 
    - empty() :	비어있으면 true 아니면 false를 리턴
    - size() : 원소의 수를 리턴
    - front() : 맨 앞 원소 리턴
    - back() : 맨 뒤 원소 리턴
    - push(a) : 맨 뒤에 원소 a를 추가
    - pop() : 맨 앞 원소 삭제

## 2. C++ 구현

```cpp
#include <iostream>
#include <queue>

int main() {
    std::queue<int> q;

    // 추가
    for (int i = 0; i < 5; ++i) {
        q.push(i);
    }

    // 가장 앞에 있는 원소 빼기
    std::cout << "front element: " << q.front() << '\n';

    // 가장 뒤에 있는 원소 빼기
    std::cout << "back element: " << q.back() << '\n';

    // 컨테이너 사이즈 구하기
    std::cout << "queue size: " << q.size() << '\n';

    // Empty 체크
    std::cout << "Is it empty?: " << (q.empty() ? "Yes" : "No") << '\n';

    // 삭제
    // pop으로 삭제 전 queue { 0, 1, 2, 3, 4 }
    // pop으로 삭제 후 queue { 3, 4 }
    q.pop();
    q.pop();
    q.pop();

    // 확인
	std::cout << "front element: " << q.front() << '\n';
	std::cout << "queue size: " << q.size() << '\n';

    return 0;
}
```

## 3. Python 구현

```python
from collections import deque

def main():
    ''' 
    *** list 를 Queue 로 사용하기 
    '''
    q_list = [5, 3, 1]

    # 추가
    q_list.append(10)
    q_list.append(20)
    print("list add ", q_list)

    # 제거, 큐이기 때문에 FIFO 원칙에 의해 맨 앞에 있는 원소를 빼야 됨
    # pop(0) 는 O(n) 시간복잡도라서 느림
    q_list.pop(0)
    q_list.pop(0)
    print("list pop ", q_list)

    # 사이즈
    print("list size ", len(q_list))
    
    ''' 
    *** deque 를 Queue 로 사용하기 
    '''
    q_deque = deque([5, 3, 1])

    # 추가
    q_deque.append(10)
    q_deque.append(20)
    print("deque add ", q_deque)

    # 제거
    # popleft 는 O(1)
    q_deque.popleft()
    q_deque.popleft()
    print("deque pop ", q_deque)

    # 사이즈
    print("deque size ", len(q_deque))

if __name__ == "__main__":
	main()
```



## 4. PriorityQueue
- 우선순위 큐는 Heap(힙)을 이용하여 구현 가능
- 데이터마다 우선순위를 넣어서 우선순위가 높은 순으로 출력
- 원소 중에서 가장 큰 값이 TOP을 유지하도록 함
- C++ 우선순위 큐 함수
    - push() :  　 우선순위 큐에 원소를 추가한다
    - pop()  :       우선순위 큐에서 top의 원소를 제거한다
    - top() :         우선순위 큐에서 top에 있는 원소 즉 우선순위가 높은 원소를 반환한다.
    - empty() :   우선순위 큐가 비어있으면 true를 반환하고 그렇지 않으면 false를 반환한다
    - size() :        우선순위 큐에 포함되어 있는 원소의 수를 반환한다

### 4.1. C++ 구현
- [web_compiler](https://godbolt.org/) 에서 확인

```cpp
#include <functional>
#include <queue>
#include <vector>
#include <iostream>
#include <string_view>
 
template<typename T>
void print(std::string_view name, T const& q) {
    std::cout << name << ": \t";
    for (auto const& n : q)
        std::cout << n << ' ';
    std::cout << '\n';
}
 
template<typename Q>
void print_queue(std::string_view name, Q q) {
    for (std::cout << name << ": \t"; !q.empty(); q.pop())
        std::cout << q.top() << ' ';
    std::cout << '\n';
}
 
int main() {
    const auto data = {1,8,5,6,3,4,0,9,7,2};
    print("data", data);
 
    // 방법1 Max priority queue, 최대값을 top으로 지정
    std::priority_queue<int> q1; 
    for(int n : data)
        q1.push(n);
 
    print_queue("q1", q1);

    // top 찾기
    std::cout << "q1 top= " << q1.top() << '\n';
 
    // 사이즈 구하기
    std::cout << "q1 size= " << q1.size() << '\n';

    // 추가
    q1.push(20);
    q1.push(10);
    print_queue("push", q1);

    // 제거
    q1.pop();
    q1.pop();
    print_queue("pop", q1);

    // 방법2 Min priority queue, 최소값을 top으로 지정
    // std::greater<int> makes the max priority queue act as a min priority queue
    std::priority_queue<int, std::vector<int>, std::greater<int>>
        minq1(data.begin(), data.end());
 
    print_queue("minq1", minq1);
 
    // 방법2.1
    // Second way to define a min priority queue
    std::priority_queue minq2(data.begin(), data.end(), std::greater<int>());
 
    print_queue("minq2", minq2);
 
    return 0;
}

```

### 4.2. Python 구현
- [web_python_compiler](https://www.onlinegdb.com/online_python_compiler) 에서 확인

```python
from queue import PriorityQueue
import heapq

def main():
    '''
    *** PriorityQueue 사용
    '''
    # Min PriorityQueue
    pq = PriorityQueue()

    # 추가
    pq.put(3)
    pq.put(5)
    pq.put(2)
    pq.put(7)
    print("PriorityQueue add ", pq.queue)

    # 제거
    top = pq.get()
    print("top val ", top)
    print("PriorityQueue remove ", pq.queue)

    # 사이즈
    print("size ", len(pq.queue))

    # Max PriorityQueue
    pq2 = PriorityQueue()

    # 추가, 값을 역으로 해주려면 마이너스 값을 가지도록 튜플을 사용
    pq2.put((-3, 3))
    pq2.put((-5, 5))
    pq2.put((-2, 2))
    pq2.put((-7, 7))
    print("PriorityQueue add ", pq2.queue)

    # 제거
    top = pq2.get()
    print("top val ", top[1])
    print("PriorityQueue remove ", pq2.queue)

    '''
    *** heapq 사용
    '''
    hq = []

    # 추가
    heapq.heappush(hq, 3)
    heapq.heappush(hq, 5)
    heapq.heappush(hq, 2)
    heapq.heappush(hq, 7)

    print("heapq add ", hq)

    # 제거
    top = heapq.heappop(hq)
    print("top ", top)
    print("heapq pop ", hq)


if __name__ == "__main__":
	main()
```

## 참고
[https://sanghyu.tistory.com/83](https://sanghyu.tistory.com/83)  
[https://kbj96.tistory.com/15](https://kbj96.tistory.com/15)  
[CPP Reference](https://en.cppreference.com/w/cpp/container)  
[Python heapq](https://docs.python.org/ko/3/library/heapq.html)
<br>

    This is personal diary for study documents.
    Please comment if I'm wrong or missing something else 😄. 

[Top](#){: .btn .btn--primary }{: .align-right}