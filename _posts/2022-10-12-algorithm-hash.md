---
published: true
title:  "[Programming] 자료구조 - Hash"
excerpt: "다양한 자료구조에 대해 알아보기"

categories:
  - Algorithm
tags:
  - [DataStructure, Hash]

toc: true
toc_sticky: true
 
date: 2022-10-12
last_modified_at: 2022-10-12
---

## 1. Hash

- 해쉬
- key, value를 저장하는 데이터구조
- key를 통해 바로 데이터를 받아올 수 있으므로, 탐색속도가 매우 빠름 O(1)
- 파이썬 딕셔너리, 보통 배열로 미리 Hash Table 사이즈만큼 생성 후에 사용 (공간과 탐색 시간을 맞바꾸는 기법)
- 해쉬: 임의 값을 고정 길이로 변환
- 해쉬테이블 키 값의 연산에 의해 접근이 가능한 데이터 구조
- 해싱 함수: key에 대한 산술 연산을 이용해 데이터 위치를 찾음
- 슬롯: 한개의 데이터를 저장할 수 있는 공간
![image](https://user-images.githubusercontent.com/23397039/194878598-f8427961-c24f-4771-9a2d-22201fb0a3e2.png)
- 장점
    - 검색 빠름, O(1)
    - 키에 대한 데이터 중복 확인 쉬움
- 단점
    - 저장공간이 많이 필요
    - 여러 키에 해당하는 주소가 동일할 경우 충돌 해결하기 위한 별도 자료구조 필요
- 용도
    - 검색, 저장, 삭제, 읽기가 빈번한 경우, 캐쉬 구현시(중복확인)
- 충돌 해결 알고리즘 (해쉬충돌)
    - chaining 기법
        - open hashing 기법 중 하나, 해쉬 테이블 저장공간 외의 공간을 활용하는 기법
        - 충돌이 일어나면 링크드 리스트라는 자료구조를 사용해서, 링크드 리스트로 데이터를 추가로 뒤에 연결시켜서 저장하는 기법
        
    - linear probing 기법
        - close hashing 기법 중 하나, 해쉬 테이블 저장공간 안에서 충돌 문제를 해결하는 기법
        - 충돌이 일어나면, 해당 hash address 의 다음 address 부터 맨 처음 나오는 빈공간에 저장하는 기법 (저장공간 활용도를 높이기 위한 기법)
    
    - 빈번한 충돌을 개선하는 기법
        - 해쉬 함수를 재정의 및 해쉬 테이블 저장공간을 확대
        - SHA(Secure Hash Algorithm) 안전한 해쉬 알고리즘
            - 어떤 데이터도 유일한 고정된 크기의 고정값을 리턴

## 2. C++ 구현
TODO

## 3. Python 구현
TODO


## 참고
[CPP LinkedList](https://velog.io/@alslahdk/C-STL-linked-list)  
[Python LinkedList](https://www.educative.io/answers/how-to-create-a-linked-list-in-python)  
[CPP Reference](https://en.cppreference.com/w/cpp/utility/hash)

<br>

    This is personal diary for study documents.
    Please comment if I'm wrong or missing something else 😄. 

[Top](#){: .btn .btn--primary }{: .align-right}