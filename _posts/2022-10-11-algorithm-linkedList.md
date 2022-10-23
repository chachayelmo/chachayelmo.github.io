---
published: true
title:  "[Programming] 자료구조 - LinkedList"
excerpt: "다양한 자료구조에 대해 알아보기"

categories:
  - Algorithm
tags:
  - [DataStructure, LinkedList]

toc: true
toc_sticky: true
 
date: 2022-10-12
last_modified_at: 2022-10-12
---

## 1. Linked list

- 링크드 리스트
    - 순차적으로 연결된 공간에 데이터를 나열하는 배열과 달리, 떨어진 곳에 존재하는 데이터를 화살표로(주소값) 연결해서 관리하는 구조
![image](https://user-images.githubusercontent.com/23397039/194874192-15fc6191-83cc-4b92-bcd3-cb3942ac3e2a.png){: .align-center}

- Node: 데이터 저장 단위
- Pointer: 각 노드에서 다음이나 이전 노드의 연결 정보를 가지고 있는 공간
- 장점
    - 미리 데이터 공간을 할당하지 않아도 됨
    - 배열은 미리 공간을 할당
- 단점
    - 연결을 위한 별도 데이터 공간이 필요
    - 접근 속도가 느림
    - 중간 데이터 삭제시, 앞뒤 데이터의 연결을 재구성해야됨

- List 함수
- li.front() : 맨 앞의 원소를 반환 ,참조
- li.back() : 맨 뒤의 원소를 반환, 참조
- li.begin() : 맨 앞의 원소를 가리키는 iterator를 반환
- li.end() : 맨 마지막의 다음 원소를 가리키는 iterator 반환
- li.rbegin() : 뒤에서부터 순차적으로 접근할때 사용
- li.rend() : 뒤에서부터 순차적으로 접근할때 사용
- li.push_back(k) : 꼬리로 원소 삽입
- li.push_front(k) : 머리로 원소 삽입
- li.pop_back() : 맨 마지막 원소 제거
- li.pop_front() : 맨 앞 원소 제거
- li.insert(i,k) : i가 가리키는 위치에 원소 삽입
- li.erase(i) : i가 가리키는 원소를 삭제
- li.size() : 원소의 개수 반환
- li.remove(k) : k와 같은 원소를 모두 제거
- li.remove_if(predicate) : 단항 조건자 predicate에 해당하는 원소를 모두 제거
- li.reverse() " 원소들의 순차열을 뒤집음
- li.sort() : 모든 원소를 오름차순(default)으로 정렬

## 2. C++ 구현
- [web_compiler](https://godbolt.org/) 에서 확인

```cpp
// constructing lists
#include <iostream>
#include <list>

void print(std::list<int> li) {
  std::cout << "The contents of fifth are: ";

  // begin, end
  for (std::list<int>::iterator it = li.begin(); it != li.end(); it++)
    std::cout << *it << ' ';

  std::cout << '\n';
}

int main ()
{
  // constructors used in the same order as described above:
  std::list<int> first;                                // empty list of ints
  std::list<int> second (4,100);                       // four ints with value 100
  std::list<int> third (second.begin(),second.end());  // iterating through second
  std::list<int> fourth (third);                       // a copy of third

  // the iterator constructor can also be used to construct from arrays:
  int myints[] = {16,2,77,29};
  std::list<int> fifth (myints, myints + sizeof(myints) / sizeof(int) );

  print(fifth);

  // add front, back
  fifth.push_front(100);
  fifth.push_back(200);

  print(fifth);

  // front
  std::cout << "front " << fifth.front() << '\n';
  // back
  std::cout << "back " << fifth.back() << '\n';

  // remove front, back
  fifth.pop_front();
  fifth.pop_back();

  print(fifth);

  return 0;
}
```

## 3. Python 구현
- [web_python_compiler](https://www.onlinegdb.com/online_python_compiler) 에서 확인

```python
# 노드
class Node:
  # constructor
  def __init__(self, data, next=None): 
    self.data = data
    self.next = next

# 링크드리스트
class LinkedList:
  def __init__(self):  
    self.head = None

  # insert 함수
  def insert(self, data):
    # 새로운 노드 생성
    new_node = Node(data)

    # Head가 있는지 없는지 체크
    if(self.head):
      current = self.head
      # 마지막 leaf node를 찾아서
      while(current.next):
        current = current.next
      # 새로운 노드 삽입
      current.next = new_node
    else:
      # Head가 없다면 새로운 노드가 head가 됨
      self.head = new_node

  # remove 함수
  def remove(self, data):
    current = self.head
    # Head가 data면 다음 노드를 Head로 바꿈
    if current.data == data:
        self.head = current.next
        return

    # 마지막 노드까지 돌면서
    while (current.next):
        # 제거될 data가 있으면 prev와 next를 연결시켜줌
        # prev -> current
        # current -> current.next
        # next -> current.next.next
        if current.next.data == data:
            current.next = current.next.next
        else:
            current = current.next

  # print 함수
  def print(self):
    print("LinkedList:", end= " ")
    current = self.head
    while(current):
      print(current.data, end=", ")
      current = current.next

def main():
    # Node 생성
    first = Node(3)
    print("Node ", first.data)

    ll = LinkedList()
    # Node 추가
    ll.insert(3)
    ll.insert(4)
    ll.insert(5)
    ll.print()

    # Node 제거
    ll.remove(4)
    ll.print()

if __name__ == "__main__":
	main()
```


## 참고
[CPP LinkedList](https://velog.io/@alslahdk/C-STL-linked-list)  
[Python LinkedList](https://www.educative.io/answers/how-to-create-a-linked-list-in-python)  
[CPP Reference](https://cplusplus.com/reference/list/list/list/)

<br>

    This is personal diary for study documents.
    Please comment if I'm wrong or missing something else 😄. 

[Top](#){: .btn .btn--primary }{: .align-right}