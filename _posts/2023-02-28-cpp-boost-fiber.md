---
published: true
title:  "[C++] 부스트 파이버(Boost fiber)"
excerpt: "Boost fiber에 대해 알아보기"

categories:
  - Boost
tags:
  - [C++, Cpp, Boost, fiber]

toc: true
toc_sticky: true
author: chachayelmo
sitemap:
  changefreq : daily
  priority : 1.0
comment: true
date: 2023-02-28
last_modified_at: 2023-01-28
---

## 1. Overview

- 부스트 파이버는 userland thread(fiber)를 위한 framework을 제공
- API에는 표준 스레드와 유사하게 파이버를 관리하고 동기화 하는 클래스와 기능이 포함되어 있음
- 각각의 파이버는 자체 스택을 가짐
- 파이버는 현재 실행 상태를 저장(registers, CPU flags, instruction pointer, and stack pointer)하고 나중에 이 상태를 복원할 수 있음
- 아이디어는 협력 스케줄링(cooperative scheduing)을 사용하여 싱글 스레드에서 여러 실행 경로를 실행하는 것 ( 스레드는 선점적으로 스케줄링 )
- 실행 중인 파이버는 다른 파이버를 실행할 수 있도록 양보해야 하는 시기를 명시적으로 결정(context switch)
- 파이버는 내부적으로 [boost.context](https://www.boost.org/doc/libs/release/libs/context/index.html)의 call/cc를 사용하여 해당 실행 컨텍스트를 관리, 예약하고 필요한 경우 동기화를 해줌
    - 스레드 간의 컨텍스트 스위치 : 수천 CPU cycles의 비용이 듬
    - 파이버 간의 컨텍스트 스위치 : 일반적으로 100 cycles 미만
- 파이버는 언제든지 단일 스레드에서 실행

- [call/cc](https://www.boost.org/doc/libs/1_81_0/libs/context/doc/html/context/cc.html) 란?
    - call with current continuation 로 universal control operator
    - 현재 CONTINUATION을 first-class object로 갭쳐하고 이를 다른 continuation에 argument로 전달하는 연산자
    - CONTINUATION은 주어진 시점에서 control flow의 state를 나타냄
    - CONTINUATION은 control flow를 변경하기 위해 suspended and resumed 가 가능

    ```cpp
    namespace ctx=boost::context;
    int a;
    ctx::continuation source=ctx::callcc(
        [&a](ctx::continuation && sink){
            a=0;
            int b=1;
            for(;;){
                sink=sink.resume();
                int next=a+b;
                a=b;
                b=next;
            }
            return std::move(sink);
        });
    for (int j=0;j<10;++j) {
        std::cout << a << " ";
        source=source.resume();
    }

    output:
        0 1 1 2 3 5 8 13 21 34
    ```

- 파이버 사용을 위한 namespace

    ```cpp
    #include<boost/fiber/all.hpp>
    namespaceboost::fibers
    namespaceboost::this_fiber
    ```

### 1.1. Fibers and Threads

- Control은 주어진 스레드에서 시작되는 fibers 간에 협력적으로 전달, 스레드에서 적어도 하나의 파이버가 running

- Thread
    - 스레드에서 추가 파이버를 생성하는 것은 프로그램이 실행 중인 코어를 더 효과적으로 사용할 수 있지만 더 많은 하드웨어 코어에 프로그램을 배포하지 않음
- Fiber
    - 파이버는 동일한 스레드에서 다른 파이버의  concurrent access에 대해 해당 리소스를 명시적으로 방어할 필요가 없이 부모 스레드가 독점적으로 소유한 모든 리소스에 안전하게 access 가능
    - 해당 스레드의 다른 파이버가 해당 리소스에 동시에 접근하지 않는다는 것이 이미 보장, 이는 레거시 코드에 concurrency(동시성)을 주입할 때 특히 중요
    - asynchronous I/O 를 사용하여 레거시 코드를 실행하는 fiber를 안전하게 생성할 수 있음
    - 특징
        - 파이버는 asynchronous I/O 를 기반으로 concurrent 코드를 구성하는 방법을 제공
        - 파이버에서 실행되는 코드는 blocking function call 처럼 보이는 것을 만들 수 있음, 해당 call은 실행 중인 파이버를 값싼 비용으로  일시 중단하여 동일한 스레드의 다른 파이버가 실행되도록 할 수 있음
        - 작업이 완료되면 상태를 명시적으로 저장하거나 복원할 필요 없이 일시 중단된 파이버가 다시 시작됨
        - 해당 local stack 변수는 call 간 에서 지속됨
        - 파이버는 한 스레드에서 다른 스레드로 마이그레이션할 수 있지만 라이브러리는 기본적으로 이 작업을 수행하지 않음
        - 스레드 간에 파이버를 마이그레이션하는 사용자 지정 스케줄러를 제공하여 허용되는 파이버를 결정하는 데 도움이 되도록 파이버 속성을 지정할 수 있음
        - 부스트 파이버는 multiple cores에서 multiplex fibers를 할 수 있음
        - 특정 스레드에서 시작된 파이버는 마이그레이션되지 않는 한 해당 스레드에서 계속 실행
        - 다른 스레드에 의해 unblocked 될 수 있지만 현재 스레드에서 파이버를 blocked 에서 ready로 바꾸는 것임

### 1.2. thread-local storage

- 파이버는 스레드 로컬 스토리지에 액세스할 수 있음
- 이는 동일한 스레드에서 실행되는 모든 파이버 간에 공유
- `[fiber_specific_ptr](https://www.boost.org/doc/libs/1_80_0/libs/fiber/doc/html/fiber/fls.html#class_fiber_specific_ptr)`을 참조

### 1.3. BOOST_FIBERS_NO_ATOMICS

- 부스트 파이버 라이브러리에서 제공하는 Fiber synchronization 객체는 기본적으로 서로 다른 스레드에서 실행되는 Fiber를 안전하게 동기화시킴
- 그러나, BOOST_FIBERS_NO_ATOMICS가 정의된 라이브러리를 빌드하면 성능을 위해 해당 레벨의 동기화를 제거할 수 있음
- 해당 매크로로 구축되면 특정 동기화 객체를 참조하는 모든 파이버가 동일한 스레드에서 실행되고 있는지 확인이 필요 [Synchronization](https://www.boost.org/doc/libs/1_80_0/libs/fiber/doc/html/fiber/synchronization.html#synchronization).

### 1.4. Blocking

- 특정 파이버가 block된다고 기술하면 제어권을 양보하여 동일한 스레드의 다른 파이버가 실행될 수 있음을 의미
- 부스트 파이버에서 제공하는 동기화 메커니즘
    - 파이버는 일반적인 스레드 동기화 메커니즘을 사용할 수 있음
    - 그러나 이러한 메커니즘을 호출하는 파이버는 전체 스레드를 block 하여 그 동안 다른 파이버가 해당 스레드에서 실행되는 것을 방지
    - 예를 들어 파이버가 동일한 스레드에서 다른 파이버의 값을 기다리고자 할 때 std::future를 사용하는 것은 안좋음
    - std::future::get()은 전체 스레드를 차단하여 다른 파이버가 전달하는 것을 방지, 대신 `[future<>](https://www.boost.org/doc/libs/1_80_0/libs/fiber/doc/html/fiber/synchronization/futures/future.html#class_future)`을 사용해야 함
    - 또한 일반 blocking I/O 작업을 호출하는 파이버는 전체 스레드를 차단함
    - 파이버를 사용할 때는 비동기 I/O를 일관되게 사용하는 것이 좋음
    - [Boost.Asio](https://www.boost.org/doc/libs/release/libs/asio/index.html) 및 기타 비동기 I/O 작업은 적용 가능 [Integrating Fibers with Asynchronous Callbacks](https://www.boost.org/doc/libs/1_80_0/libs/fiber/doc/html/fiber/callbacks.html#callbacks).
    - 부스트 파이버는 부스트 컨텍스트([Boost.Context](https://www.boost.org/doc/libs/release/libs/context/index.html))에 의존!
    
## 2. Fiber management

### 2.1. 개요

```cpp
#include <boost/fiber/all.hpp>

namespace boost {
namespace fibers {

class fiber;
bool operator<( fiber const& l, fiber const& r) noexcept;
void swap( fiber & l, fiber & r) noexcept;

template< typename SchedAlgo, typename ... Args >
void use_scheduling_algorithm( Args && ... args);
bool has_ready_fibers();

namespace algo {

struct algorithm;
template< typename PROPS >
struct algorithm_with_properties;
class round_robin;
class shared_round_robin;

}}

namespace this_fiber {

fibers::id get_id() noexcept;
void yield();
template< typename Clock, typename Duration >
void sleep_until( std::chrono::time_point< Clock, Duration > const& abs_time)
template< typename Rep, typename Period >
void sleep_for( std::chrono::duration< Rep, Period > const& rel_time);
template< typename PROPS >
PROPS & properties();

}
```

### 2.2. 사용

- 각 fiber는 스케줄러에 의해 시작 및 관리됨
- fiber object는 move-only

```cpp
boost::fibers::fiber f1; // not-a-fiber

void f() {
    boost::fibers::fiber f2( some_fn);

    f1 = std::move( f2); // f2 moved to f1
}
```

- 

### 2.1 [Class fiber](https://www.boost.org/doc/libs/1_80_0/libs/fiber/doc/html/fiber/fiber_mgmt/fiber.html)

```cpp
#include <boost/fiber/fiber.hpp>

namespace boost {
namespace fibers {

class fiber {
public:
        // class id는 아래에서 알아보기!
    class id;

        // default 생성자
    constexpr fiber() noexcept;

        // 생성자들, Fn은 copyable or movable
    template< typename Fn, typename ... Args >
    fiber( Fn &&, Args && ...);

    template< typename Fn, typename ... Args >
    fiber( launch, Fn &&, Args && ...);

    template< typename StackAllocator, typename Fn, typename ... Args >
    fiber( std::allocator_arg_t, StackAllocator &&, Fn &&, Args && ...);

    template< typename StackAllocator, typename Fn, typename ... Args >
    fiber( launch, std::allocator_arg_t, StackAllocator &&, Fn &&, Args && ...);

    ~fiber();

    fiber( fiber const&) = delete;

    fiber & operator=( fiber const&) = delete;

    fiber( fiber &&) noexcept;

    fiber & operator=( fiber &&) noexcept;

    void swap( fiber &) noexcept;

    bool joinable() const noexcept;

    id get_id() const noexcept;

    void detach();

    void join();

    template< typename PROPS >
    PROPS & properties();
};

bool operator<( fiber const&, fiber const&) noexcept;

void swap( fiber &, fiber &) noexcept;

template< typename SchedAlgo, typename ... Args >
void use_scheduling_algorithm( Args && ...) noexcept;

bool has_ready_fibers() noexcept;

}}
```

### 2.2. [Class fiber::id](https://www.boost.org/doc/libs/1_80_0/libs/fiber/doc/html/fiber/fiber_mgmt/id.html)

- 

### 2.3. [Namespace this_fiber](https://www.boost.org/doc/libs/1_80_0/libs/fiber/doc/html/fiber/fiber_mgmt/this_fiber.html)



## 참고
[boost.org fiber](https://www.boost.org/doc/libs/1_80_0/libs/fiber/doc/html/index.html)  

<br>

    This is personal diary for study documents.
    Please comment if I'm wrong or missing something else 😄. 

[Top](#){: .btn .btn--primary }{: .align-right}