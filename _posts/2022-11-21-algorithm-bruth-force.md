---
published: true
title:  "[알고리즘] 완전 탐색(Bruth Force)"
excerpt: "알고리즘 완전 탐색"

categories:
  - Algorithm
tags:
  - [DataStructure, BruthForce]

toc: true
toc_sticky: true
author: chachayelmo
sitemap:
  changefreq : daily
  priority : 1.0
comment: true
date: 2022-11-22
last_modified_at: 2022-12-04
---

## 1. 완전 탐색
- 문제를 해결하기 위해 확인해야 하는 모든 경우의 수를 전부 탐색하는 방법
- 그중에서도 back-tracking을 통해야 하는 상황을 해결하기
- 모든 코테 문제에서 기본적으로 접근해야 됨
  - 고를 수 있는 값의 종류 파악하기
  - 중복을 허용하는 지
  - 순서가 중요한 지

## 2. 완전 탐색 종류
- N개 중 {중복을 허용/ 중복 없이} M개를 {순서 있게 나열하기 / 고르기}

### 2.1. N개 중 중복 허용 / M개를 순서 있게 나열하기
[N과 M(3)](https://www.acmicpc.net/problem/15651)
- 1부터 N까지 자연수 중에서 M개를 고른 수열
- 같은 수를 여러 번 골라도 된다.

```python
import sys
input = lambda: sys.stdin.readline().rstrip()
n, m = map(int, input().split())
selected = [0] * m

# 재귀함수
# M 개를 전부 고름 -> 조건에 맞는 탐색을 한가지 성공
# 아직 M 개를 고르지 않음 -> k 번째부터 M 번재 원소를 조건에 맞게 고르는 모든 방법을 시도
def rec_func(k):
  if k == m:
    # 다 고른 배열 출력
    print(*selected)
  else:
    # 1 ~ n 까지를 k 번 원소로 한번씩 정하고
    # 매번 k+1 번부터 m 번 원소로 재귀호출
    for cand in range(1, n + 1):
      selected[k] = cand # k 번째 인자로 1 ~ n 까지 원소 입력
      rec_func(k + 1)
      selected[k] = 0 # 클리어

rec_func(0) # 실행
```

### 2.2. N개 중 중복 없이 / M개를 순서 있게 나열하기
[N과 M(1)](https://www.acmicpc.net/problem/15649)
- 1부터 N까지 자연수 중에서 중복 없이 M개를 고른 수열

```python
# 1 <= M, N <= 8
# 시간: O(nPm) 8! = 40,320
# 공간: O(M)

import sys
input = lambda: sys.stdin.readline().rstrip()
n, m = map(int, input().split())
selected = [0] * m

def rec_func(k):
  if k == m:
    # 다 고른 배열 출력
    print(*selected)
  else:
    for cand in range(1, n + 1):
      is_used = False
      if cand in selected:
        is_used = True
      # cand 가 올 수 있으면
      if not is_used:
        selected[k] = cand
        rec_func(k + 1)
        selected[k] = 0

rec_func(0)
```

```python
# 1 <= M, N <= 8
# 시간: O(nPm) 8! = 40,320
# 공간: O(M)

import sys
input = lambda: sys.stdin.readline().rstrip()
n, m = map(int, input().split())
selected = [0] * m
is_used = [0] * (n + 1)

def rec_func(k):
  if k == m:
    # 다 고른 배열 출력
    print(*selected)
  else:
    for cand in range(1, n + 1):
      if is_used[cand] == 1:
          continue
      selected[k] = cand
      is_used[cand] = 1
      rec_func(k + 1)
      selected[k] = 0
      is_used[cand] = 0

rec_func(0)
```
### 2.3. N개 중 중복 허용 / M개를 고르기
[N과 M(4)](https://www.acmicpc.net/problem/15652)
- 1부터 N까지 자연수 중에서 M개를 고른 수열
- 같은 수를 여러 번 골라도 된다.
- 고른 수열은 비내림차순이어야 한다.
  - 길이가 K인 수열 A가 A1 ≤ A2 ≤ ... ≤ AK-1 ≤ AK를 만족하면, 비내림차순이라고 한다.

```python
import sys
input = lambda: sys.stdin.readline().rstrip()
n, m = map(int, input().split())

selected = [0] * m

def rec_func(k):
  if k == m:
    # 다 고른 배열 출력
    print(*selected)
  else:
    start = selected[k - 1]
    if start == 0: start = 1
    for cand in range(start, n + 1):
      selected[k] = cand
      rec_func(k + 1)
      selected[k] = 0

rec_func(0)
```

### 2.4. N개 중 중복없이 / M개를 고르기
[N과 M (2)](https://www.acmicpc.net/problem/15650)
- 1부터 N까지 자연수 중에서 중복 없이 M개를 고른 수열
- 고른 수열은 오름차순이어야 한다.

```python
import sys
input = lambda: sys.stdin.readline().rstrip()
n, m = map(int, input().split())

selected = [0] * m
is_used = [0] * (n + 1)

def rec_func(k, idx):
  if k == m:
    # 다 고른 배열 출력
    print(*selected)
  else:
    for cand in range(idx, n + 1):
      if is_used[cand] == 1: # 사용 중인 숫자를 제외하고 탐색
        continue
      selected[k] = cand
      is_used[cand] = 1
      rec_func(k + 1, cand)
      selected[k] = 0
      is_used[cand] = 0

rec_func(0, 1)
```

```python
# 시간 : O(nCm) = 70
# 공간 : O(m)

import sys
input = lambda: sys.stdin.readline().rstrip()
n, m = map(int, input().split())

selected = [0] * m
is_used = [0] * (n + 1)

def rec_func(k):
  if k == m:
    # 다 고른 배열 출력
    print(*selected)
  else:
    for cand in range(selected[k-1] + 1, n + 1):
      selected[k] = cand
      rec_func(k + 1)
      selected[k] = 0

rec_func(0)
```

## 참고
[FastCampus 류호석강사](https://github.com/rhs0266/FastCampus/blob/main/%EA%B0%95%EC%9D%98%20%EC%9E%90%EB%A3%8C/02-%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98/01~02-%EC%99%84%EC%A0%84%20%ED%83%90%EC%83%89/02-%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98-01-%EC%99%84%EC%A0%84%ED%83%90%EC%83%89.pdf)


<br>

    This is personal diary for study documents.
    Please comment if I'm wrong or missing something else 😄. 

[Top](#){: .btn .btn--primary }{: .align-right}