---
published: true
title:  "[Data structure] 자료구조 - 배열(Array)"
excerpt: "다양한 자료구조에 대해 알아보기"

categories:
  - Algorithm
tags:
  - [자료구조, 배열, 어레이, DataStructure, array]

toc: true
toc_sticky: true
author: chachayelmo
sitemap:
  changefreq : daily
  priority : 1.0
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

## 2. C++ 구현
- [web_compiler](https://godbolt.org/) 에서 확인

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

## 3. Python 구현
- [web_python_compiler](https://www.onlinegdb.com/online_python_compiler) 에서 확인
- list를 사용하지만 이는 C++로 보면 array, stack 등 여러가지 컨테이너가 합쳐진 형태

```python
def main():
    array_list = [3, 2, 1, 5, 9]

    # 값 변경
    print("before change ", array_list)
    array_list[3] = 100
    print("after change ", array_list)

    # 사이즈
    print("size ", len(array_list))

    # 값 추가
    array_list.append(999)
    print("add ", array_list)

    # 값 제거
    array_list.remove(2)
    print("remove ", array_list)

if __name__ == "__main__":
	main()

```


## 참고
[잔재미코딩](https://www.fun-coding.org/DS&AL1-2.html)  
[CPP Reference](https://en.cppreference.com/w/cpp/container/array)

<br>

    This is personal diary for study documents.
    Please comment if I'm wrong or missing something else 😄. 

[Top](#){: .btn .btn--primary }{: .align-right}