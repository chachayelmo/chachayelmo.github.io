---
published: true
title:  "[Programming] 자료구조 - Array"
excerpt: "다양한 자료구조에 대해 알아보기"

categories:
  - Algorithm
tags:
  - [DataStructure, array]

toc: true
toc_sticky: true
 
date: 2022-10-08
last_modified_at: 2022-10-08
---

## 1. Array

- 배열
- 데이터를 나열하고 각 인덱스에 대응하도록 구성한 데이터구조
- 순차적으로 연결된 공간에 데이터를 나열
- 장점
    - 빠른 접근 가능
- 단점
    - 추가/삭제가 어려움
    - 미리 공간을 확보해야 됨

```cpp
#include <stdio.h>
#include <array>
#include <algorithm>
#include <iterator>
#include <iostream>

void print_func(int arr[], size_t len) {
    for (int i = 0 ; i < len ; ++i) {
        printf("%d ", arr[i]);
    }
    printf("\n");
}

template <typename T>
void print_func(T& arr) { // 오버로드
    for (const auto& i : arr) {
        printf("%d ", i);
    }
    printf("\n");
}

int main() {
    // C 스타일
    int arr[10] = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10}; // 10개짜리 배열 추가

    // C 스타일 array size 구하기 
    size_t sz = sizeof(arr)/sizeof(*arr);
    print_func(arr, sz);

    // 값 변경
    arr[5] = 100;
    print_func(arr, sz);

    // C++ 스타일
    std::array<int, 5> cpp_array = {5,2,1,4,3};

    // C++ 스타일 array size 구하기
    printf("size = %d \n", cpp_array.size());
    print_func(cpp_array);

    // 컨테이너 함수들 제공
    std::sort(cpp_array.begin(), cpp_array.end());
    print_func(cpp_array);
    std::reverse_copy(cpp_array.begin(), cpp_array.end(), 
                      std::ostream_iterator<int>(std::cout, " "));

    return 0;
}
```



## 참고
[잔재미코딩](https://www.fun-coding.org/DS&AL1-2.html)
[CPP Reference](https://en.cppreference.com/w/cpp/container/array)

<br>

    This is personal diary for study documents.
    Please comment if I'm wrong or missing something else 😄. 

[Top](#){: .btn .btn--primary }{: .align-right}