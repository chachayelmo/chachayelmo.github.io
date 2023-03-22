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

### 2.3  [Class fiber](https://www.boost.org/doc/libs/1_80_0/libs/fiber/doc/html/fiber/fiber_mgmt/fiber.html)

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

    // 파이버가 joinable인 동안 소멸자가 실행되지 않도록 해야 됨
    // 파이버가 완료된 것을 알고 있더라도 파이버를 제거하기 전에 join() or detach()를 호출해야 됨
    bool joinable() const noexcept;

    id get_id() const noexcept;

    // 실행중인 파이버가 detach되고 더 이상 관련된 파이버가 없음
    void detach();

    // 실행중인 파이버가 완료될 때까지 기다림
    void join();

    // *this에 대한 스케줄러 속성 인스턴스에 대한 참조
    template< typename PROPS >
    PROPS & properties();
};

bool operator<( fiber const&, fiber const&) noexcept;

void swap( fiber &, fiber &) noexcept;

template< typename SchedAlgo, typename ... Args >
void use_scheduling_algorithm( Args && ...) noexcept;

// 스케줄러에 실행할 파이버가 있으면 true
bool has_ready_fibers() noexcept;

}}
```

### 2.4. [Class fiber::id](https://www.boost.org/doc/libs/1_80_0/libs/fiber/doc/html/fiber/fiber_mgmt/id.html)

```cpp
#include <boost/fiber/fiber.hpp>

namespace boost {
namespace fibers {

class id {
public:
    constexpr id() noexcept;
    bool operator==( id const&) const noexcept;
    bool operator!=( id const&) const noexcept;
    bool operator<( id const&) const noexcept;
    bool operator>( id const&) const noexcept;
    bool operator<=( id const&) const noexcept;
    bool operator>=( id const&) const noexcept;

    template< typename charT, class traitsT >
    friend std::basic_ostream< charT, traitsT > &
    operator<<( std::basic_ostream< charT, traitsT > &, id const&);
};

}}
```

### 2.5. [Namespace this_fiber](https://www.boost.org/doc/libs/1_80_0/libs/fiber/doc/html/fiber/fiber_mgmt/this_fiber.html)

```cpp
namespace boost {
namespace this_fiber {

fibers::fiber::id get_id() noexcept;
void yield() noexcept;
template< typename Clock, typename Duration >
void sleep_until( std::chrono::time_point< Clock, Duration > const&);
template< typename Rep, typename Period >
void sleep_for( std::chrono::duration< Rep, Period > const&);
template< typename PROPS >
PROPS & properties();

}}
```

- Non-member functions

```cpp
#include <boost/fiber/operations.hpp>

namespace boost {
namespace fibers {

// 현재 실행중인 파이버 id를 나타냄
fiber::id get_id() noexcept;

// abs_time에 도달할 때까지 현재 파이버를 일시 중단
template< typename Clock, typename Duration >
void sleep_until( std::chrono::time_point< Clock, Duration > const& abs_time);

// rel_time에서 지정한 시간이 경과할 때까지 현재 파이버를 일시 중단
template< class Rep, class Period >
void sleep_for( std::chrono::duration< Rep, Period > const& rel_time);
}}

// 실행 제어를 포기하고 다른 파이버가 실행되도록 허용
void yield() noexcept;

// 현재 실행 중인 파이버에 대한 스케줄러 속성 인스턴스에 대한 참조
template< typename PROPS >
PROPS & properties();
```

## 3. 스케줄링

- 스레드 파이버는 fiber manager에 의해 조정
- 파이버는 협력적으로 제워권을 교환하기 때문에, 현재 실행 중인 파이버는 제어권을 manager에게 전달하는 작업이 호출될 때까지 제어권을 유지
- 파이버가 일시 중지 또는 yield 될 대마다 manger는 다음에 실행될 파이버를 결정하기 위해 스케줄러를 참조
- 각 스레드에는 자체 스케줄러가 있으며, 프로세스의 다른 스레드는 다른 스케줄러를 사용할 수 있음
- 기본적으로 round robin을 각 스레드의 스케줄러로 만들며 교체가 가능

### 3.1. use_scheduling_algorithm

- 알고리즘 sub class는 use_scheduling_algorithm 을 호출하여 특정 스레드에 참여하는 것이 가능

```cpp
void thread_fn() {
    boost::fibers::use_scheduling_algorithm< my_fiber_scheduler >();
    ...
}
```

### 3.2. 스케줄러 클래스

- 스케줄러 클래스는 인터페이스 알고리즘을 구현해야 함
- 파이버는 round_robin, work_stealing, numa::work_stealing 및 shared_work와 같은 스케줄러를 제공

```cpp
void thread( std::uint32_t thread_count) {
    // thread registers itself at work-stealing scheduler
    boost::fibers::use_scheduling_algorithm< boost::fibers::algo::work_stealing >( thread_count);
    ...
}

// count of logical cpus
std::uint32_t thread_count = std::thread::hardware_concurrency();
// start worker-threads first
std::vector< std::thread > threads;
for ( std::uint32_t i = 1 /* count start-thread */; i < thread_count; ++i) {
    // spawn thread
    threads.emplace_back( thread, thread_count);
}
// start-thread registers itself at work-stealing scheduler
boost::fibers::use_scheduling_algorithm< boost::fibers::algo::work_stealing >( thread_count);
...
```

### 3.3. Class algorithm

```cpp
#include <boost/fiber/algo/algorithm.hpp>

namespace boost {
namespace fibers {
namespace algo {

struct algorithm {
    virtual ~algorithm();

    // 파이버 f가 실행될 준비가 되었음을 스케줄러에 알림
    virtual void awakened( context *) noexcept = 0;

    // 다음에 재개할 파이버 or 준비된 파이버가 없으면 nullptr
    virtual context * pick_next() noexcept = 0;

    // 스케줄러에 실행할 파이버가 있으면 true
    virtual bool has_ready_fibers() const noexcept = 0;

    // abs_time까지 파이버가 준비되지 않음을 스케줄러에 알림
    virtual void suspend_until( std::chrono::steady_clock::time_point const&) noexcept = 0;

    // suspend_until() 에 대한 대기 중인 call 에서 return 하도록 스케줄러에 요청
    virtual void notify() noexcept = 0;
};

}}}
```

- class round_robin

```cpp
#include <boost/fiber/algo/round_robin.hpp>

namespace boost {
namespace fibers {
namespace algo {

class round_robin : public algorithm {
    // 파이버 f를 ready queue에 넣음
    virtual void awakened( context *) noexcept;

    // ready queue의 head에 있는 파이버 or empty인 경우 nullptr
    virtual context * pick_next() noexcept;

    // 스케줄러에 실행할 파이버가 있으면 true
    virtual bool has_ready_fibers() const noexcept;

    // abs_time까지 사용할 수 있는 파이버가 없음을 알림
    virtual void suspend_until( std::chrono::steady_clock::time_point const&) noexcept;

    // suspend_util()에 대한 대기 중인 call을 깨우면 일부 파이버가 준비되었을 수 있음
    // 이 구현은 std::condition_variable::notify_all()을 통해 suspend_until()을 꺠움
    virtual void notify() noexcept;
};

}}}
```

- class work_stealing
    - local ready-queue에 준비된 파이버가 부족하면 다른 스케줄러에서 준비된 파이버를 훔침
    - 희생 스케줄러(준비된 파이버를 뺏김)는 무작위로 선택

```cpp
#include <boost/fiber/algo/work_stealing.hpp>

namespace boost {
namespace fibers {
namespace algo {

class work_stealing : public algorithm {
public:
    work_stealing( std::uint32_t thread_count, bool suspend = false);

    work_stealing( work_stealing const&) = delete;
    work_stealing( work_stealing &&) = delete;

    work_stealing & operator=( work_stealing const&) = delete;
    work_stealing & operator=( work_stealing &&) = delete;

    // 파이버 f를 shared ready queue에 넣음
    virtual void awakened( context *) noexcept;

    // ready queue의 head에 있는 파이버 또는 empty면 nullptr
    virtual context * pick_next() noexcept;

    // 스케줄러에 실행할 파이버가 있으면 true
    virtual bool has_ready_fibers() const noexcept;

    // abs_time까지 사용할 수 있는 준비된 파이버가 없음을 알림
    virtual void suspend_until( std::chrono::steady_clock::time_point const&) noexcept;

    // suspend_util()에 대한 대기 중인 call을 깨우면 일부 파이버가 준비되었을 수 있음
    // 이 구현은 std::condition_variable::notify_all()을 통해 suspend_until()을 꺠움
    virtual void notify() noexcept;
};

}}}
```

- Class shared_work
    - round_robin 방식으로 파이버를 스케줄링하는 알고리즘
    - 준비된 파이버는 shared_work의 모든 instance(서로 다른 스레드에서 실행) 간에 공유되므로 작업이 모든 스레드에 균등하게 분배

```cpp
#include <boost/fiber/algo/shared_work.hpp>

namespace boost {
namespace fibers {
namespace algo {

class shared_work : public algorithm {
    shared_work();
    shared_work( bool suspend);

    // 파이버 f를 shared ready queue에 넣음
    virtual void awakened( context *) noexcept;

    // ready queue의 head에 있는 파이버 또는 empty면 nullptr
    virtual context * pick_next() noexcept;

    // 스케줄러에 실행할 파이버가 있으면 true
    virtual bool has_ready_fibers() const noexcept;

    // abs_time까지 사용할 수 있는 준비된 파이버가 없음을 알림
    virtual void suspend_until( std::chrono::steady_clock::time_point const&) noexcept;

    // suspend_util()에 대한 대기 중인 call을 깨우면 일부 파이버가 준비되었을 수 있음
    // 이 구현은 std::condition_variable::notify_all()을 통해 suspend_until()을 꺠움
    virtual void notify() noexcept;
};

}}}
```

- Class context
    - 알고리즘 구현은 전달된 context instance를 관리하려는 모든 컨테이너를 사용할 수 있음
    - ready_queue_t는 일반적인 STL 컨테이너의 일부 오버헤드를 방지함

```cpp
#include <boost/fiber/context.hpp>

namespace boost {
namespace fibers {

enum class type {
  none               = unspecified,
  main_context       = unspecified, // fiber associated with thread's stack
  dispatcher_context = unspecified, // special fiber for maintenance operations
  worker_context     = unspecified, // fiber not special to the library
  pinned_context     = unspecified  // fiber must not be migrated to another thread
};

class context {
public:
    class id;

    // 현재 파이버에 대한 포인터
    static context * active() noexcept;

    context( context const&) = delete;
    context & operator=( context const&) = delete;

    // *this 가 실행 파이버를 참조하는 경우 해당 파이버를 나타내는 id의 instance
    // 그렇지 않으면 기본 구성된 id를 반환
    id get_id() const noexcept;

    // *this를 실행하는 스케줄러에서 *this 파이버를 분리함
    void detach() noexcept;

    // *this를 실행하는 스케줄러에 파이버 f를 연결
    void attach( context *) noexcept;

    // *this가 지정된 type인 경우 true
    bool is_context( type) const noexcept;

    // *this 가 더이상 유효하지 않은 경우 true
    bool is_terminated() const noexcept;

    // *this가 알고리즘 구현의 ready-queue에 저장되어 있으면 true
    bool ready_is_linked() const noexcept;

    // *this가 fiber manager의 remote-ready-queue에 있으면 true
    bool remote_ready_is_linked() const noexcept;

    // *this 가 일부 동기화 object의 wait-queue에 있으면 true
    bool wait_is_linked() const noexcept;

    // *this를 ready-queue list에 저장
    template< typename List >
    void ready_link( List &) noexcept;

    // *this를 remote-ready-queue list에 저장
    template< typename List >
    void remote_ready_link( List &) noexcept;

    // *this를 wait-queue list에 저장
    template< typename List >
    void wait_link( List &) noexcept;

    // ready-queue에서 *this를 제거
    void ready_unlink() noexcept;

    // remote-ready-queue에서 *this를 제거
    void remote_ready_unlink() noexcept;

    // wait-queue에서 *this를 제거
    void wait_unlink() noexcept;

    // 다른 파이버가 this를 context::schedule()로 전달할 때까지 실행 중인 파이버를 일시 중단
    // *this는 not-ready로 표시되며 실행할 다른 파이버를 선택하기 위해 제어를 스케줄러로 전달
    void suspend() noexcept;

    // context *ctx와 연결된 파이버를 실행할 준비가 된 것으로 표시
    // 이것은 해당 파이버를 즉시 시작하지는 않음
    // 후속 재개를 위해 파이버를 스케줄러로 전달
    void schedule( context *) noexcept;
};

bool operator<( context const& l, context const& r) noexcept;

}}
```

## 4. Synchronization

- 일반적으로, 파이버 synchronization object는 이동하거나 복사할 수 없음
- synchronization object는 서로 다른 파이버 간에 mutually-agreed rendezvous point 역할을 함
- object가 다른 곳에 복사된 경우 새로운 복사본에 대한 소비자가 없게 동작
- 파이버 synchronization object는 기본적으로 서로 다른 스레드에서 실행되는 파이버를 안전하게 동기화 시킴
- 성능을 위해 BOOST_FIBERS_NO_ATOMICS가 정의된 library를 빌드하면 위의 동기화를 제거할 수 있음

### 4.1. Mutex Types

- Class mutex
    - mutex는 독점 소유권(exclusive-ownership)을 제공
    - mutex instance는 최대 하나의 파이버를 언제든지 lock 을 소유할 수 있음
    - lock(), try_lock(), unlock()에 대한 multiple concurrent 호출을 허용 

```cpp
#include <boost/fiber/mutex.hpp>

namespace boost {
namespace fibers {

class mutex {
public:
    mutex();
    ~mutex();

    mutex( mutex const& other) = delete;
    mutex & operator=( mutex const& other) = delete;

    // 소유권을 얻을 떄 까지 현재 파이버는 block
    void lock();
    // 현재 파이버를 block하지 않고 소유권을 얻으려고 시도
    bool try_lock();
    // 현재 파이버에 의해 *this에 대한 lock을 release
    void unlock();
};

}}
```

- Class timed_mutex
    - mutex와 같이 독점 소유권을 제공 및 최대 하나의 파이버를 언제든지 잠금을 소유할 수 있음
    - lock(), try_lock(), try_lock_until(), try_lock_for() and unlock() 에 대한 다중 동시 호출 허용

```cpp
#include <boost/fiber/timed_mutex.hpp>

namespace boost {
namespace fibers {

class timed_mutex {
public:
    timed_mutex();
    ~timed_mutex();

    timed_mutex( timed_mutex const& other) = delete;
    timed_mutex & operator=( timed_mutex const& other) = delete;

    // 소유권을 얻을 떄 까지 현재 파이버는 block
    void lock();

    // 현재 파이버를 block하지 않고 소유권을 얻으려고 시도
    bool try_lock();

    // 현재 파이버에 의해 *this에 대한 lock을 release
    void unlock();

    // 현재 파이버에 대한 소유권을 얻으려고 시도
    // 소유권을 얻을 수 있을 때까지 or 지정된 시간에 도달할 떄까지 block, 지정된 시간이 경과한 경우 timed_mutex::try_lock()으로 동작
    template< typename Clock, typename Duration >
    bool try_lock_until( std::chrono::time_point< Clock, Duration > const& timeout_time);
    // 현재 파이버에 대한 소유권을 얻으려고 시도
    // 소유권을 얻을 수 있을 때까지 or 지정된 시간에 도달할 떄까지 block, 지정된 시간이 경과한 경우 timed_mutex::try_lock()으로 동작
    template< typename Rep, typename Period >
    bool try_lock_for( std::chrono::duration< Rep, Period > const& timeout_duration);
};

}}
```

- Class recursive_mutex
    - exclusive-ownership recursive mutex를 제공
    - lock(), try_lock(), unlock()에 대한 다중 동시 호출을 허용
    - 이미 recursive_mutex 소유권을 가지고 있는 파이버는 lock() or try_lock()을 호출하여 추가적인 레벨의 소유권을 획득할 수 있음
    - 다른 파이버에서 소유권을 획득하기 전에 단일 파이버에서 획득한 각 소유권 레벨에 대해 unlcok()을 한번 호출해야 됨

```cpp
#include <boost/fiber/recursive_mutex.hpp>

namespace boost {
namespace fibers {

class recursive_mutex {
public:
    recursive_mutex();
    ~recursive_mutex();

    recursive_mutex( recursive_mutex const& other) = delete;
    recursive_mutex & operator=( recursive_mutex const& other) = delete;

    // mutex와 동일
    void lock();
    bool try_lock() noexcept;
    void unlock();
};

}}
```

- Class recursive_timed_mutex

```cpp
#include <boost/fiber/recursive_timed_mutex.hpp>

namespace boost {
namespace fibers {

class recursive_timed_mutex {
public:
    recursive_timed_mutex();
    ~recursive_timed_mutex();

    recursive_timed_mutex( recursive_timed_mutex const& other) = delete;
    recursive_timed_mutex & operator=( recursive_timed_mutex const& other) = delete;

    // timed_mutex와 동일
    void lock();
    bool try_lock() noexcept;
    void unlock();

    template< typename Clock, typename Duration >
    bool try_lock_until( std::chrono::time_point< Clock, Duration > const& timeout_time);
    template< typename Rep, typename Period >
    bool try_lock_for( std::chrono::duration< Rep, Period > const& timeout_duration);
};

}}
```

### 4.2. Condition Variables

- 해당 클래스는 파이버가 다른 파이버의 notification을 기다리는 메커니즘을 제공
- 파이버가 대기에서 깨어나면 condition이 true인지 확인하고 true이면 계속함
- condition이 false면 파이버 call은 resume을 위해 waiting

```cpp
boost::fibers::condition_variable cond;
boost::fibers::mutex mtx;
bool data_ready = false;

void process_data();

void wait_for_data_to_process() {
    {
        std::unique_lock< boost::fibers::mutex > lk( mtx);
        while ( ! data_ready) {
            cond.wait( lk);
        }
    }   // release lk
    process_data();
}
```


### 4.3. Barriers

### 4.4. Channels

### 4.5. Futures



## 참고
[boost.org fiber](https://www.boost.org/doc/libs/1_80_0/libs/fiber/doc/html/index.html)  

<br>

    This is personal diary for study documents.
    Please comment if I'm wrong or missing something else 😄. 

[Top](#){: .btn .btn--primary }{: .align-right}