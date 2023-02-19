---
published: true
title:  "[OS] 가상메모리(Virtual Memory)"
excerpt: "가상메모리에 대해 알아보기"

categories:
  - OS
tags:
  - [운영체제, 가상메모리, 메모리, OS, Memory, Virtual]

toc: true
toc_sticky: true
author: chachayelmo
sitemap:
  changefreq : daily
  priority : 1.0
date: 2022-09-31
last_modified_at: 2022-09-31
---

## 1. 메모리
- 가상 메모리도 굉장히 큰 부분 프로세스 관련 기술
- 실제 각 프로세스마다 충분한 메모리를 할당하기에는 메모리 크기가 한계가 있음
- 리눅스 하나의 프로세스가 4GB
- 통상 메모리는 8GB? 16GB?, 폰노이만 구조 기반이므로, 코드는 메모리에 반드시 있어야 함

## 2. 가상메모리가 필요한 이유?

- 하나의 프로세스만 실행 가능한 시스템(배치 처리 시스템 등)
    1. 프로그램을 메모리로 로드
    2. 프로세스 실행
    3. 프로세스 종료 (메모리 해제)
- 여러 프로세스 동시 실행 시스템
    1. 메모리 용량 부족 이슈
    2. 프로세스 메모리 영역간에 침범 이슈

## 3. 가상메모리
- 메모리가 실제 메모리보다 많아 보이게 하는 기술
- 실제 사용하는 메모리는 작다는 점에서 착안해서 고안된 기술
- 프로세스 간 공간 분리로 프로세스 이슈가 전체 시스템에 영향을 주지 않을 수 있음
- 기본아이디어
  - 프로세스는 가상주소를 사용하고, 실제 해당 주소에서 데이터를 읽고/쓸때만 물리 주소로 바꿔주면 됨
  - virtual address: 프로세스가 참조하는 주소
  - physical address: 실제 메모리 주소

## 4. MMU
- MMU = Memory Management Unit
- CPU에 코드 실행 시, 가상 주소 메모리 접근이 필요할 때, 해당 주소를 물리 주소값으로 변환해주는 하드웨어 장치
- CPU는 가상 메모리를 다루고, 실제 해당 주소 접근시 MMU라는 HW를 통해 물리 메모리에 접근
- HW 장치를 이용해야 주소 변환이 빠르기 때문에 별도 장치를 둠

![mmu.png](../../assets/images/mmu.png){: .align-center}

## 5. 페이징 시스템

### 5.1. 페이징 개념
- 크기가 동일한 페이지로 가상 주소 공간과 이에 매칭하는 물리 주소 공간을 관리
- 하드웨어 지원이 필요, Intel x86 시스템(32bit)에서는 4KB, 2MB, 1GB 지원
- 프로세스(4GB) 의 PCB에 page table 구조체를 가리키는 주소가 들어 있음
- page table에는 가상 주소와 물리 주소간 매핑 정보가 있음

![page_table.png](../../assets/images/page_table.png){: .align-center}

### 5.2. 페이징 시스템 구조

- page frame: 고정된 크기의 block (4KB)
- paging system
  - 가상 주소 v = (p, d)
  - p : 가상 메모리 페이지
  - d: p 안에서 참조하는 위치

![virtual_address.png](../../assets/images/virtual_address.png){: .align-center}

- 가상 주소의 0비트에서 11비트가 변위 (d)를 나타내고, 12 비트 이상이 페이지 번호가 될 수 있음
- 프로세스가 4GB를 사용하는 이유 ? 32bit 시스템에서 2^32 = 4GB

### 5.3. 페이징 시스템 동작

- 해당 프로세스에서 특정 가상 주소 엑세스를 하려면:
- 해당 프로세스의 page table에 해당 가상 주소가 포함된 page 번호가 있는지 확인
- page 번호가 있으면 이 페이지가 매핑된 첫 물리주소를 알아내고(p)
- p + d 가 실제 물리주소가 됨

## 참고
[잔재미코딩](https://www.fun-coding.org/virtualmemory.html)

<br>

    This is personal diary for study documents.
    Please comment if I'm wrong or missing something else 😄. 

[Top](#){: .btn .btn--primary }{: .align-right}