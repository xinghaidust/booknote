[TOC]

# chapter 1. Hello, world of concurrency in  C++!  
Approaches to concurrency  

1. concurrency with multiple processes 
2. concurrency with multiple threads in a single process  

**C++ Standard** doesn’t provide any intrinsic support for **communication between processes**, so applications that use multiple processes will have to rely on **platform-specific API**s  

***Concurrency*** and ***parallelism*** have largely overlapping meanings but ***parallelism*** is **performance-oriented** while primary concern of ***concurrency*** is **separation of concerns**, or responsiveness.   

There are two main reasons to use concurrency in an application: **separation of concerns** and **performance**.   

## Concurrency and multithreading in C++  

It’s only since the **C++11 Standard** that you’ve been able to write multithreaded code without resorting to platform-specific extensions. Not only is there a **thread-aware memory model**, but the **C++ Standard Library** was extended to include classes for 

1. managing threads 
2. protecting shared data 
3. synchronizing operations between threads
4. low-level atomic operations  

The only specific support for concurrency and parallelism added in `C++14` was a **new mutex type** for protecting shared data. But `C++17` adds considerably more: a full suite of **parallel algorithms** for starters.   

The C++ Standards Committee was aware of ***abstraction penalty*** when designing the C++ Standard Library in general and the goals of **Standard C++ Thread Library** is

1. efficient implementation (with a low ***abstraction penalty***) on most major platforms  
2. provides sufficient **low-level facilities** for those wishing to work close to the metal for the ultimate performance  

**C++ Thread Library** may offer a `native_handle()` member function that allows the underlying implementation to be directly manipulated using a **platform-specific API**  

# chapter 2. Managing threads  

## Launching a thread 

`std::thread` class is used to manage a thread of execution. Its **non-default constructor** starts a new thread of execution and **destructor** calls `std::terminate()`

```C++
class background_task
{
public:
    void operator()() const
    {
        do_something();
        do_something_else();
    }
};
background_task f;
std::thread my_thread(f);
// C++’s most vexing parse, declares a my_thread function 
std::thread my_thread(background_task());
// use following instead
std::thread my_thread((background_task()));
std::thread my_thread{background_task()};
```

<font color=red>you need to ensure that you’ve called either `join()` or `detach()` before a `std::thread` object is destroyed</font>

## Passing arguments to a thread function 

It’s a bad idea to create a thread within a function that has access to the local variables in that function, unless the thread is guaranteed to finish before the function exits 

One common way to handle this scenario is to make the thread function **self-contained** and copy the data into the thread rather than sharing the data 

```C++
template< class F, class... Args >
explicit thread( F&& f, Args&&... args );
```

constructor of `std::thread` <font color=red>**decay-copy**</font> `f` and `args` into internal storage so have to use `std::ref` to pass reference and use `std::move` for move-only object like `std::unique_ptr`

```C++
std::thread t(update_data_for_widget,w,std::ref(data));

void process_big_object(std::unique_ptr<big_object>);
std::unique_ptr<big_object> p(new big_object);
p->prepare_data(42);
std::thread t(process_big_object,std::move(p));
```

`std::thread` is movable to transfer ownership of thread 

## Choosing the number of threads at runtime 

`std::thread::hardware_concurrency()` returns an indication of the number of threads that can
truly run concurrently for a given execution of a program and return 0 if this information isn’t available.

On a multicore system it might be the number of CPU cores, 

## Identifying threads 

`std::this_thread:: get_id()` 
`std::thread:: get_id(),`

# chapter 3. Sharing data between threads 
## Problems with sharing data between threads 

If all shared data is read-only, there’s no problem
But if one or more threads start modifying shared data, there’s a lot of potential for trouble 

The simplest potential problem with modifying data that’s shared between threads is that of **broken invariants** 

## race condition

In concurrency, a **race condition** is anything where the outcome depends on the relative ordering of execution of operations on two or more threads; 

C++ Standard also defines the term **data race** to mean the specific type of race condition that arises because of concurrent modification to a single object 

**Avoiding problematic race conditions:**

1. wrap your data structure with a protection mechanism to ensure that only the thread performing a modification can see the intermediate states where the invariants are broken 
2. modify the design of your data structure and its invariants so that modifications are done as a series of indivisible changes, referred to as ***lock-free programming*** 
3. handle the updates to the data structure as a ***transaction***, referred as **software transactional memory (STM)** with no no direct support in C++

## Protecting shared data with mutexes 

instead of `std::mutex`, use `std::lock_guard` or `std::scoped_lock` which accept any type satisfying the ***mutex concept***: `lock()`, `unlock()`, and `try_lock()` 

**race conditions** still exist if 

1. Any code that has access to that pointer or reference can access (and potentially modify) the protected data without locking the mutex. 

2. interface problem, solved by

    1. pass in a reference to accept popped value, limited to assignable type and assign is expensive 
    2. restrict to types that can safely be returned by value without throwing an exception 
    3. return a pointer such as `std::shared_ptr`, which is freely copied without throwing an exception 
    
   ```C++
   // combine interface to avoid race conditions 
   T new_pop()
   {
       int const value=s.top();
       s.pop();
       return value; // may throw std::bad_alloc exception when copy construct from return value
   }
   
   // split interface but cause race conditions 
   stack<int> s;
   int const value=s.top();
   s.pop(); // void std::stack<T,Container>::pop
   
   // pass in a reference
   void pop(T& value)
   {
       std::lock_guard<std::mutex> lock(m);
       if(data.empty()) throw empty_stack();
       value=data.top();
       data.pop();
   }
   
   // return a pointer
   std::shared_ptr<T> pop()
   {
       std::lock_guard<std::mutex> lock(m);
       if(data.empty()) throw empty_stack();
       std::shared_ptr<T> const res(std::make_shared<T>(data.top()));
       data.pop();
       return res;
   }
   ```
   
## Deadlock: the problem and a solution 

**deadlock** occurs when lock two or more mutexes for a given operation

**deadlock** doesn’t only occur with locks; it can occur with any synchronization construct that can lead to a wait cycle such as thread

guidelines for avoiding deadlock:

1. the simplest: don’t acquire a lock if you already hold one, therefore, avoid calling user-supplied code which may acquire a lock while holding a lock 
2.  use `std::lock` or `std::scoped_lock` if you can acquire two or more locks simultaneously as a single operation 
3. <font color=red>acquire two or more locks in the same order in every thread if you have to acquire them separately, by means of **a lock hierarchy**</font>

### lock hierarchy 

The idea is that you divide your application into layers and identify all the mutexes that may be locked in any given layer. When code tries to lock a mutex, it isn’t permitted to lock that mutex if it already holds a lock from a lower layer.  

A simple hierarchical mutex is as follows. The key is the use of the **thread_local** value representing the hierarchy value

```C++
#include <mutex>
#include <stdexcept>
#include <climits>
class hierarchical_mutex
{
    std::mutex internal_mutex;
    unsigned long const hierarchy_value;
    unsigned long previous_hierarchy_value;
    static thread_local unsigned long this_thread_hierarchy_value;

    void check_for_hierarchy_violation()
    {
        if(this_thread_hierarchy_value <= hierarchy_value)
        {
            throw std::logic_error("mutex hierarchy violated");
        }
    }
    void update_hierarchy_value()
    {
        previous_hierarchy_value=this_thread_hierarchy_value;
        this_thread_hierarchy_value=hierarchy_value;
    }
public:
    explicit hierarchical_mutex(unsigned long value):
        hierarchy_value(value),
        previous_hierarchy_value(0)
    {}
    void lock()
    {
        check_for_hierarchy_violation();
        internal_mutex.lock();
        update_hierarchy_value();
    }
    void unlock()
    {
        this_thread_hierarchy_value=previous_hierarchy_value;
        internal_mutex.unlock();
    }
    bool try_lock()
    {
        check_for_hierarchy_violation();
        if(!internal_mutex.try_lock())
            return false;
        update_hierarchy_value();
        return true;
    }
};
thread_local unsigned long
    hierarchical_mutex::this_thread_hierarchy_value(ULONG_MAX);
```

## Flexible locking with std::unique_lock  

`std::unique_lock` , a more flexible version `std::lock_guard` ,has a flag that indicates whether the mutex is currently owned by that instance and is moveable but not copyable to transferring mutex ownership  

## Locking at an appropriate granularity  

lock at too small a granularity doesn’t cover the entirety of the desired operation 
lock at too large a granularity eliminate any performance benefits of concurrency

a lock should be held for only the minimum possible time needed to perform the required operations.  In particular, don’t do any time-consuming activities like file I/O while holding a lock unless the lock is intended to protect access to the file I/O   

Sometimes, there isn’t an appropriate level of granularity because not all accesses to the data structure require the same level of protection. In this case, it might be appropriate to use an alternative mechanism, instead of a plain std::mutex  

## Protecting shared data during initialization  

One particularly extreme (but remarkably common) case is where the shared data needs protection from concurrent access **only when** it’s being initialized

1. C++ Standard Library provides `std::once_flag` and `std::call_once`    
2. initialization of **static local variable** is defined to occur the first time control passes through its declaration and **no race condition since C++11**

## reader-writer mutex

use `std::shared_mutex` and `std::shared_timed_mutex`  

`std::lock_guard<std::shared_mutex>` and `std::unique_lock<std::shared_mutex>` can be used for exclusive locking  and `std::shared_lock<std::shared_mutex>` to obtain shared access.  

## Recursive locking  

C++ Standard Library provides `std::recursive_mutex` like `std::mutex`, except that you can acquire multiple times on a single instance from the same thread  

A common but not recommended use of **recursive mutexes** is one locking public member function to call another as part of its operation. It’s usually better to extract a new private member function that’s called from both member functions, which does not lock the mutex  

# chapter 4. Synchronizing concurrent operations  
Although it would be possible to **synchronize actions on separate threads** by periodically checking a “task complete” flag or something similar stored in shared data, C++ Standard Library provides facilities to handle it, in the form of **condition variables** and **futures**, **latches** and **barriers**.  

## Waiting for a condition with condition variables  

The Standard C++ Library provides two implementations of a **condition variable**, need to work with a mutex in order to provide appropriate synchronization  

1. `std::condition_variable`, limited to working with `std::mutex` 
2. `std::condition_variable_any`, can work with anything mutex-like with potential additional costs of size, performance, or OS resources  

```C++
template<typename T>
class threadsafe_queue
{
private:
    mutable std::mutex mut;
    std::queue<T> data_queue;
    std::condition_variable data_cond;
public:
    threadsafe_queue() {}
    threadsafe_queue(threadsafe_queue const& other)
    {
        std::lock_guard<std::mutex> lk(other.mut);
        data_queue=other.data_queue;
    }

    void push(T new_value)
    {
        std::lock_guard<std::mutex> lk(mut);
        data_queue.push(new_value);
        data_cond.notify_one();
    }

    void wait_and_pop(T& value)
    {
        std::unique_lock<std::mutex> lk(mut);
        data_cond.wait(lk,[this]{return !data_queue.empty();});
        value=data_queue.front();
        data_queue.pop();
    }

    std::shared_ptr<T> wait_and_pop()
    {
        std::unique_lock<std::mutex> lk(mut); // must unlock the mutex so cannot use std::lock_guard
        data_cond.wait(lk,[this]{return !data_queue.empty();});
        std::shared_ptr<T> res(std::make_shared<T>(data_queue.front()));
        data_queue.pop();
        return res;
    }

    bool try_pop(T& value)
    {
        std::lock_guard<std::mutex> lk(mut);
        if(data_queue.empty)
            return false;
        value=data_queue.front();
        data_queue.pop();
        return true;
    }

    std::shared_ptr<T> try_pop()
    {
        std::lock_guard<std::mutex> lk(mut);
        if(data_queue.empty())
            return std::shared_ptr<T>();
        std::shared_ptr<T> res(std::make_shared<T>(data_queue.front()));
        data_queue.pop();
        return res;
    }

    bool empty() const
    {
        std::lock_guard<std::mutex> lk(mut);
        return data_queue.empty();
    }
};
```

<font color=red>`std::condition_variable::wait` is an optimization over a ***busy-wait*** </font>

```C++
template<typename Predicate>
void minimal_wait(std::unique_lock<std::mutex>& lk,Predicate pred){
while(!pred()){
        lk.unlock();
        lk.lock();
    }
}
```

## Waiting for one-off events with futures  

If the waiting thread is going to wait only once, a **condition variable** might not be the best
choice of synchronization mechanisms. A ***future*** might be more appropriate  

two sorts of futures in the C++ Standard Library: 

1. unique futures (`std::future`) like `std::unique_ptr`, only moveable   
2. shared futures (`std::shared_future`), like `std::shared_ptr`. copyable  

### `std::async`

Rather than giving you a `std::thread` object to wait on, `std::async` returns a `std::future` object, . When call `get()` on the ***future***, and the calling thread blocks until the future is ready and then returns the value.   

```C++
// construct by decay-copy in the same way that std::thread does
template< class F, class... Args >
std::future</* see below */> async( F&& f, Args&&... args );

template< class F, class... Args >
std::future</* see below */> async( std::launch policy,
                                    F&& f, Args&&... args );
```

You can select policy with `std::launch::deferred` or `std::launch::async`. By default, it’s up to the implementation and is recommended

### `std::packaged_task`

The `std::packaged_task` object is a **callable object** and its template parameter is a **function signature** whose return type identifies the type of the `std::future` returned from the `get_future()` member function  

```C++
template< class > class packaged_task; //not defined

template< class R, class ...ArgTypes >
class packaged_task<R(ArgTypes...)>;
```

```C++
std::mutex m;
std::deque<std::packaged_task<void()> > tasks;

bool gui_shutdown_message_received();
void get_and_process_gui_message();

void gui_thread()
{
    while(!gui_shutdown_message_received())
    {
        get_and_process_gui_message();
        std::packaged_task<void()> task;
        {
            std::lock_guard<std::mutex> lk(m);
            if(tasks.empty())
                continue;
            task=std::move(tasks.front());
            tasks.pop_front();
        }
        task();
    }
}

std::thread gui_bg_thread(gui_thread);

template<typename Func>
std::future<void> post_task_for_gui_thread(Func f)
{
    std::packaged_task<void()> task(f);
    std::future<void> res=task.get_future();
    std::lock_guard<std::mutex> lk(m);
    tasks.push_back(std::move(task));
    return res;
}
```

### `std::promise`  

```C++
extern std::promise<double> some_promise;
try
{
	some_promise.set_value(calculate_value());
}
catch(...)
{
	some_promise.set_exception(std::current_exception());
    // some_promise.set_exception(std::make_exception_ptr(std::logic_error("foo ")));
}
```

exception is stored in the ***future*** in place of a stored value, and a call to `get()` rethrows that stored exception  

1.  task throws an exception  
2. call the `set_exception()` member function for `std::promise`
3. destroy the `std::promise` without calling `set_value()` or `std::packaged_task` without invoking. the destructor will store a `std::future_error` exception with an error code of `std::future_errc::broken_promise` 

## Waiting with a time limit  

two sorts of timeout: 

1. a duration-based timeout, `_for` suffix  
2.  an absolute timeout, `_until` suffix   

Defined in header `<chrono>` and  in namespace `std::chrono` 
**clock type** with **static** member such as `std::chrono::system_clock` and `std::chrono::steady_clock`

```C++
template<
    class Rep, //  an arithmetic type, representing the number of ticks
    class Period = std::ratio<1>
> class duration;

template<
    class Clock,
    class Duration = typename Clock::duration
> class time_point;
```

**Implicit conversion** between `std::duration` where not require truncation or **explicit conversions**  with `std::chrono::duration_cast`  

Plain `std::mutex` and `std::recursive_mutex` don’t support timeouts on locking, instead use `try_lock_for()` and `try_lock_until()` member functions of `std::timed_mutex` and `std::recursive_timed_mutex`.

## Using synchronization of operations to simplify code  

1. **functional programming** uses ***pure function*** that doesn’t modify any external state

2. **CSP (Communicating Sequential Processes) paradigm** where threads are conceptually entirely separate, with no shared data but with communication channels that allow messages to be passed between them. This is the paradigm adopted by the **MPI (Message Passing Interface)** environment commonly used for high-performance computing in C and C++. Because C++ threads share an address space, it’s not possible to enforce this requirement, it’s our responsibility to ensure that we don’t share data between the threads.   

3. **Concurrency TS** provides extended versions of ***futures***, **continuations**—additional functions that are run automatically when the future becomes ready, 
	
	`std::experimental::shared_future` objects can have **more than one continuation** and the **return value from the continuation** is a plain `std::experimental::future`
	
	```C++
	// Unwrapping constructor to flatten
	future( std::experimental::future<std::experimental::future<T>> && other ) noexcept;
	// Chaining continuations
	// when a continuation is added with then(), the original future becomes invalid. Instead, the call to then() returns a new future to hold the result of the continuation call
	
	std::future<void> process_login(
	    std::string const& username,std::string const& password)
	{
	    return std::async(std::launch::async,[=](){
	        try {
	            user_id const id=backend.authenticate_user(username,password);
	            // waste thread time while waiting for authenticate_user to complete
	            user_data const info_to_display=
	                backend.request_current_info(id);
	            // waste thread time while waiting for request_current_info to complete
	            update_display(info_to_display);
	        } catch(std::exception& e){
	            display_error(e);
	        }
	    });
	}
	
	// you cannot pass arguments to a continuation function, because the argument is already defined by the library—the continuation is passed a ready future that holds the result that triggered the continuation
	#include <experimental/future>
	std::experimental::future<void> process_login(
	    std::string const& username,std::string const& password)
	{
	    return spawn_async([=](){
	        return backend.authenticate_user(username,password);
	    }).then([](std::experimental::future<user_id> id){
	        return backend.request_current_info(id.get());
	    }).then([](std::experimental::future<user_data> info_to_display){
	        try{
	            update_display(info_to_display.get());
	        } catch(std::exception& e){
	            display_error(e);
	        }
	    });
	}
	```

### Waiting for more than one future 

You pass the set of futures to be waited on to `std::experimental::when_all`, and it returns a new future
that becomes ready when all the futures in the set are ready.  

```C++
std::experimental::future<FinalResult> process_data(
    std::vector<MyData>& vec)
{
    size_t const chunk_size=whatever;
    std::vector<std::experimental::future<ChunkResult>> results;
    for(auto begin=vec.begin(),end=vec.end();beg!=end;){
        size_t const remaining_size=end-begin;
        size_t const this_chunk_size=std::min(remaining_size,chunk_size);
        results.push_back(
            spawn_async(
            process_chunk,begin,begin+this_chunk_size));
        begin+=this_chunk_size;
    }
    return std::experimental::when_all(
        results.begin(),results.end()).then(
        [](std::future<std::vector<
             std::experimental::future<ChunkResult>>> ready_results)
        {
            std::vector<std::experimental::future<ChunkResult>>
                all_results=ready_results .get();
            std::vector<ChunkResult> v;
            v.reserve(all_results.size());
            for(auto& f: all_results)
            {
                v.push_back(f.get());
            }
            return gather_results(v);
        });
}
```

### Waiting for the first future in a set  

```C++
template< class Sequence >
struct when_any_result {
    std::size_t index;
    Sequence futures;
};

template< class InputIt >
auto when_any( InputIt first, InputIt last )
    -> future<when_any_result<std::vector<typename std::iterator_traits<InputIt>::value_type>>>;

template< class... Futures >
auto when_any( Futures&&... futures )
    -> future<when_any_result<std::tuple<std::decay_t<Futures>...>>>;
```



```C++
#include <atomic>
#include <experimental/future>
#include <memory>
#include <vector>
struct MyData {};
struct FinalResult {};

bool matches_find_criteria(MyData const &);
FinalResult process_found_value(MyData const &);

std::experimental::future<FinalResult>
find_and_process_value(std::vector<MyData> &data) {
    unsigned const concurrency = std::thread::hardware_concurrency();
    unsigned const num_tasks = (concurrency > 0) ? concurrency : 2;
    std::vector<std::experimental::future<MyData *>> results;
    auto const chunk_size = (data.size() + num_tasks - 1) / num_tasks;
    auto chunk_begin = data.begin();
    std::shared_ptr<std::atomic<bool>> done_flag =
        std::make_shared<std::atomic<bool>>(false);
    for (unsigned i = 0; i < num_tasks; ++i) {
        auto chunk_end =
            (i < (num_tasks - 1)) ? chunk_begin + chunk_size : data.end();
        results.push_back(std::experimental::async([=] {
            for (auto entry = chunk_begin; !*done_flag && (entry != chunk_end);
                 ++entry) {
                if (matches_find_criteria(*entry)) {
                    *done_flag = true;
                    return &*entry;
                }
            }
            return (MyData *)nullptr;
        }));
        chunk_begin = chunk_end;
    }
    std::shared_ptr<std::experimental::promise<FinalResult>> final_result =
        std::make_shared<std::experimental::promise<FinalResult>>();
    struct DoneCheck {
        std::shared_ptr<std::experimental::promise<FinalResult>> final_result;

        DoneCheck(
            std::shared_ptr<std::experimental::promise<FinalResult>>
                final_result_)
            : final_result(std::move(final_result_)) {}

        void operator()(
            std::experimental::future<std::experimental::when_any_result<
                std::vector<std::experimental::future<MyData *>>>>
                results_param) {
            auto results = results_param.get();
            MyData *const ready_result = results.futures[results.index].get();
            if (ready_result)
                final_result->set_value(process_found_value(*ready_result));
            else {
                results.futures.erase(results.futures.begin() + results.index);
                if (!results.futures.empty()) {
                    std::experimental::when_any(
                        results.futures.begin(), results.futures.end())
                        .then(std::move(*this));
                } else {
                    final_result->set_exception(
                        std::make_exception_ptr(
                            std::runtime_error(“Not found”)));
                }
            }
        }
    };

    std::experimental::when_any(results.begin(), results.end())
        .then(DoneCheck(final_result));
    return final_result->get_future();
}
```

### Latches and barriers in the Concurrency TS 

In the cases waiting for is for a set of threads to reach a particular point in the code, or to have processed a certain number of data items between them, you might be better served using a ***latch*** or a ***barrier*** rather than a ***future***

***Latches*** and ***barriers*** are thread coordination mechanisms that allow any number of threads to block until an expected number of threads arrive. A ***latch*** cannot be reused, while a ***barrier*** can be used repeatedly.

A ***latch*** is a synchronization object that becomes ready when its counter is decremented to zero, thus a lightweight facility for waiting for a series of events to occur. 

A ***barrier*** is a reusable synchronization component used for internal synchronization between a set of threads.  

`std::experimental::latch`
`std::experimental::barrier`
`std::experimental::flex_barrier`

A ***barrier*** *phase* consists of the following steps:

1. The **expected count** is decremented by each call to [`arrive`](https://en.cppreference.com/w/cpp/thread/barrier/arrive) or [`arrive_and_drop`](https://en.cppreference.com/w/cpp/thread/barrier/arrive_and_drop).
2. When the **expected count** reaches zero, the ***phase completion step*** is run, meaning that the [`*completion*`](https://en.cppreference.com/w/cpp/thread/barrier#Data_members) is invoked. The end of the completion step [strongly happens-before](https://en.cppreference.com/w/cpp/atomic/memory_order#Strongly_happens-before) all calls that were unblocked by the completion step return.
3. When the ***completion step*** finishes, the **expected count** is reset to the value specified at construction less the number of calls to [`arrive_and_drop`](https://en.cppreference.com/w/cpp/thread/barrier/arrive_and_drop) since, and the next ***barrier phase*** begins.

# chapter 5. The C++ memory model and operations on atomic types 
## Memory model basics 

two aspects to the memory model: 

1. the basic structural aspects, which relate to how things are laid out in memory
2. the concurrency aspects.  

### structural aspects

In C++ an object has size, alignment requirement, storage duration, lifetime, type, value.

An object is stored in one or more **memory locations**. Each **memory location** is either an object (or sub-object) of a **scalar type** or a sequence of adjacent **bit fields**

#### Bit-field

The type of a bit-field can only be integral or (possibly cv-qualified) enumeration type, an unnamed bit-field cannot be declared with a cv-qualified type.

Multiple adjacent bit-fields are usually packed together (although this behavior is **implementation-defined**):

The **special unnamed bit-field of size zero** can be forced to **break up padding**. It specifies that the next bit-field begins at the beginning of its allocation unit:

```C++
struct S
{
    // will usually occupy 2 bytes:
    unsigned char b1 : 3; // 1st 3 bits (in 1st byte) are b1
    unsigned char    : 2; // next 2 bits (in 1st byte) are blocked out as unused
    unsigned char b2 : 6; // 6 bits for b2 - doesn't fit into the 1st byte => starts a 2nd
    unsigned char b3 : 2; // 2 bits for b3 - next (and final) bits in the 2nd byte
};

{
    // will usually occupy 2 bytes:
    // 3 bits: value of b1
    // 5 bits: unused
    // 2 bits: value of b2
    // 6 bits: unused
    unsigned char b1 : 3;
    unsigned char :0; // start a new byte
    unsigned char b2 : 2;
};
```

### concurrency aspects

In order to avoid the **race condition**, there has to be an **enforced ordering** between the accesses in the two threads by **mutexes** or **atomic operations**.

using **atomic operations** to access the memory location avoids the **undefined behavior** but doesn’t prevent the race itself 

Every object in a C++ program has a **modification order** composed of all the writes to that object from all threads in the program, starting with the object’s initialization 

All threads must agree on the **modification orders** of each individual object in a program, otherwise a data race and **undefined behavior** occurs. But they don’t necessarily have to agree on the **relative order** of operations on separate objects 

## Atomic operations and types in C++ 

the key use case for **atomic operations** is as a replacement for a mutex for synchronization, so **the standard atomic types** have `is_lock_free()` member function and, since C++17, have a **static constexpr member variable**, `X::is_always_lock_free`, consistent with both the macro `ATOMIC_xxx_LOCK_FREE`

On most popular platforms it’s expected that the atomic variants of all the built-in types are **lock-free**, but not required.

The **standard atomic types `std::atomic<>` class template** are not ***copyable*** or ***assignable*** and the operations are limited to `load()`, `store()` `exchange()`, `compare_exchange_weak()`, and `compare_exchange_strong()` but some specializations  may support the **compound assignment operators** where appropriate. For **atomic types** the assignment operators they support **return values** (of
the corresponding non-atomic type) rather than references. 

There are **type aliases** for all `std::atomic<Integral>` but it's generally simpler to use `std::atomic<T>`  

Each of the operations on the **atomic types** has an **optional memory-ordering argument** of type `std::memory_order` enumeration 

1. **Store operations**, which can have `memory_order_relaxed`, `memory_order_release`, or `memory_order_seq_cst` ordering
2. **Load operations**, which can have `memory_order_relaxed`, `memory_order_consume`, `memory_order_acquire`, or `memory_order_seq_cst` ordering
3. **Read-modify-write operations**, which can have `memory_order_relaxed`, `memory_ order_consume`, `memory_order_acquire`, `memory_order_release`, `memory_order_acq_rel`, or `memory_order_seq_cst` ordering 

### Operations on std::atomic_flag 

`std::atomic_flag` is the only type that doesn’t provide an `is_lock_free()` member function, and operations are required to be **lock-free** 

This type is a simple Boolean flag whose **only available operations** include **initialized to clear**, **queried and set** (with `test_and_set()` member function) or **cleared** (with the `clear()` member function) or destroy.

Objects of the `std::atomic_flag` type must be initialized to a clear state with **ATOMIC_FLAG_INIT**.

**static** `std::atomic_flag` object is guaranteed to be **statically initialized**, which means that there are no
**initialization-order issues** 

The limited feature set makes std::atomic_flag ideally suited to use as a **spinlock mutex** with a **busy-wait** `lock()`

```C++
#include <atomic>
class spinlock_mutex
{
    std::atomic_flag flag;
public:
    spinlock_mutex():
        flag(ATOMIC_FLAG_INIT) // initializes an std::atomic_flag to false
    {}
    void lock()
    {
        // atomically sets the flag to true and obtains its previous valu
        while(flag.test_and_set(std::memory_order_acquire));
    }
    void unlock()
    {
        // atomically sets flag to false
        flag.clear(std::memory_order_release);
    }
};
```

### Operations on `std::atomic<bool>` 

`std::atomic_flag` is so limited that it can’t even be used as a general Boolean flag, because it doesn’t have a simple **nonmodifying query operation** 

```C++
// can construct it from a non-atomic bool
std::atomic<bool> b(true);
// can assign from a non-atomic bool
b=false;
bool x=b.load(std::memory_order_acquire);
b.store(true);
// atomically replaces the value of atomic object and obtains the value held previously
x=b.exchange(false,std::memory_order_acq_rel);
```

`compare_exchange_weak() and compare_exchange_strong() `: atomically compares the value of the atomic object with non-atomic argument and performs atomic exchange and return ***true*** if equal or atomic load and return ***false*** if not 

`compare_exchange_weak()` might **fail spuriously** even if the original value was equal to the expected value, most likely to happen on machines that lack a single **compare-and-exchange instruction** and interrupted by other thread, so typically be used in a loop: 

```C++
bool expected=false;
extern atomic<bool> b; // set somewhere else
while(!b.compare_exchange_weak(expected,true) && !expected);
```

The **compare-exchange functions** can take two **memory-ordering** parameters for the case of success and failure 

### Operations on `std::atomic<T*>`: pointer arithmetic 

`std::atomic<T*>` has all member function as `std::atomic<bool>` in addition to `fetch_add()`, `fetch_sub()` member functions, and the `+=` and `-=` operators, and both **pre- and post-increment and decrement** with `++` and `--` 

### Operations on standard atomic integral types 

The remaining basic atomic types are all the same: they’re all **atomic integral types** and have the same interface 

atomic integral types have a comprehensive set of operations available: `fetch_add()`, `fetch_sub()`, `fetch_and()`, `fetch_or()`, `fetch_xor()`, compound-assignment forms of these operations (`+=`, `-=`, `&=`, `|=`, and `^=`), and **pre- and post-increment and decrement** (`++x`, `x++`, `--x`, and `x--`). 

only **division, multiplication, and shift operators** which can easily be done using `compare_exchange_weak() `in a loop, are missing because atomic integral values are typically used either as **counters** or as **bitmasks**,

### The `std::atomic<>` primary class template 

The primary `std::atomic` template may be instantiated with any [*TriviallyCopyable*](https://en.cppreference.com/w/cpp/named_req/TriviallyCopyable) type, meaning no user-supplied **copy-assignment** or **comparison operators**, permits the compiler to use `memcpy()`

**compare-exchange operations** do **bitwise comparison** as if using `memcmp`, rather than any **comparison operator**. If the type provides **comparison operations** that have different semantics, or the type has **padding bits** that do not participate in normal comparisons, **compare-exchange operation** will fail 

### Free functions for atomic operations 

The free functions are designed to be **C-compatible**, so use **pointers** rather than **references** in all cases 

equivalent nonmember functions with an `atomic_` ***prefix*** for all member function on atomic types 

nonmember functions with an `_explicit` ***suffix*** has additional parameters for the memory-ordering tag 

```C++
std::atomic_store(&atomic_var,new_value);
std::atomic_store_explicit(&atomic_var,new_value,std::memory_order_release);
std::atomic_is_lock_free(&a);
```

## Synchronizing operations and enforcing ordering 

***Sequenced before*** is an asymmetric, transitive, pair-wise relationship between evaluations within the same thread. The **evaluation of expressions** obey [ordering rules](https://en.cppreference.com/w/cpp/language/eval_order) 

<font color=red>Above order and rule is limited to the **same thread**, **inter-thread order** relies on **atomic operations** to synchronize</font>

<font color=red>Different CPU may see different values at the same memory location because of CPU caches and internal buffers, **the final aim of synchronization is not to enforce inter-thread instruction order but enforce all CPU share the same view of memory**</font>

**synchronizes-with relationship** is as follows:

> If an atomic store in thread A is a *release operation*, an atomic load in thread B from the same variable is an *acquire operation*, and the load in thread B reads a value written by the store in thread A, then the store in thread A *synchronizes-with* the load in thread B.

For a **single thread**, if one operation is ***sequenced before*** another, then it also ***happens before*** it, and ***strongly-happens before*** it while ***inter-thread happens-before*** relies on the ***synchronizes-with relationship***  

The difference between ***happens before*** and ***strongly-happens before*** is that operations tagged with ***memory_order_consume*** participate in ***inter-thread-happens-before relationships*** (and thus ***happens-before relationships***), but not in ***strongly-happens-before relationships***.  

### Memory ordering for atomic operations  

The six ordering options represent three models: 

1. **sequentially consistent ordering** (***memory_order_seq_cst***)
2. **acquire-release ordering** (***memory_order_consume***, ***memory_order_acquire***, ***memory_order_release***, ***memory_order_acq_rel***)
3. **relaxed ordering** (***memory_order_relaxed***)  

Unless specify, the memory-ordering option for all operations on atomic types is ***memory_order_seq_cst***, which is the most stringent of the available options.   

distinct **memory-ordering models** can have varying costs on different CPU architectures.   

#### sequentially consistent ordering

If all operations on instances of atomic types are **sequentially consistent**, all threads of a  multi-thread program must see the **same order of operations** as if all these operations were performed in some particular sequence by a single thread.

#### relaxed ordering  

For ***non-sequentially-consistent memory ordering***, threads don’t have to agree on the order of events  not just because the compiler can reorder the instructions. Even if the threads are running the same bit of code, different CPU caches and internal buffers can hold different values for the same memory  

***Relaxed ordering*** only guarantee **atomicity** and **modification order consistency on each individual variable**

1. **Write-write coherence**: If a write A that modifies some atomic M ***happens-before*** evaluation B that modifies M, then A appears earlier than B in the ***modification order*** of M.
2. **Read-read coherence**: if a read A of some atomic M ***happens-before*** a read B on M, and if the value of A comes from a write X on M, then the value of B is either the value stored by X, or the value stored by a side effect Y on M that appears later than X in the ***modification order*** of M.
3. **Read-write coherence**: if a read A of some atomic M ***happens-before*** a write B on M, then the value of A comes from a side-effect (a write) X that appears earlier than B in the ***modification order*** of M.
4. **Write-read coherence**: if a write X on an atomic object M ***happens-before*** a read B of M, then the evaluation B shall take its value from X or from a side effect Y that follows X in the ***modification order*** of M.

#### acquire-release ordering  

**Acquire-release ordering** introduces pairwise synchronization between the thread that does the release and the thread that does the acquire  

Between threads, evaluation A ***inter-thread happens before*** evaluation B if any of the following is true:

1) A ***synchronizes-with*** B (for the **same atomic object**)
2) A is ***dependency-ordered before*** B (see ***memory_order_consume***).
3) A ***synchronizes-with*** some evaluation X, and X is ***sequenced-before*** B (X and B in the **same thread**).
4) A is ***sequenced-before*** some evaluation X (A and X in the same thread), and X ***inter-thread happens-before*** B.
5) A ***inter-thread happens-before*** some evaluation X, and X ***inter-thread happens-before*** B.

#### data dependency with acquire-release ordering and memory_order_consume  

***memory_order_consume*** introduces the data-dependency nuances to the ***inter-thread
happens-before relationship***
<font color=red>C++17 standard explicitly recommends that you do not use it </font>

***Carries dependency***

Within the **same thread**, evaluation A that is *sequenced-before* evaluation B may also carry a dependency into B (that is, B depends on A), if any of the following is true:

1) The value of A is used as an operand of B, **except**
      a) if B is a call to [std::kill_dependency](https://en.cppreference.com/w/cpp/atomic/kill_dependency),

      b) if A is the left operand of the built-in `&&`, `||`, `?:`, or `,` operators.

2) A writes to a scalar object M, B reads from M.
3) A ***carries dependency*** into another evaluation X, and X ***carries dependency*** into B.

***Dependency-ordered before***

Between threads, evaluation A is ***dependency-ordered before*** evaluation B if any of the following is true:

1) A performs a ***release operation*** (tagged with `memory_order_release`, `memory_order_acq_rel`, or `memory_order_seq_cst`) on some atomic M, and, in a different thread, B performs a ***consume operation*** on the same atomic M, and B reads a value written by A.
2) A is ***dependency-ordered before*** X and X ***carries a dependency*** into B.

<font color=red>***memory_order_consume*** is special case of ***acquire-release ordering***, and only limit the evaluations which have dependency</font>

```C++
struct X
{
    int i;
    std::string s;
};

std::atomic<X*> p;
std::atomic<int> a;

void create_x()
{
    X* x=new X;
    x->i=42;
    x->s="hello";
    a.store(99,std::memory_order_relaxed);
    p.store(x,std::memory_order_release);
}

void use_x()
{
    X* x;
    while(!(x=p.load(std::memory_order_consume)))
        std::this_thread::sleep_for(std::chrono::microseconds(1));
    assert(x->i==42); // guaranteed not to fire
    assert(x->s=="hello"); // guaranteed not to fire
    assert(a.load(std::memory_order_relaxed)==99); // may or may not fire
}
int main()
{
    std::thread t1(create_x);
    std::thread t2(use_x);
    t1.join();
    t2.join();
}
```

### Release sequences and synchronizes-with  

the ***release sequence rules*** assures *multiple* threads can synchronise their loads on a single store. 

```C++
#include <atomic>
#include <thread>
#include <vector>

std::vector<int> queue_data;
std::atomic<int> count;

void wait_for_more_items() {}
void process(int data){}

void populate_queue()
{
    unsigned const number_of_items=20;
    queue_data.clear();
    for(unsigned i=0;i<number_of_items;++i)
    {
        queue_data.push_back(i);
    }
    
    count.store(number_of_items,std::memory_order_release);
}

void consume_queue_items()
{
    while(true)
    {
        int item_index;
        // two thread fetch_sub both sync with count.store, so both see initialized queue_data rather than broken queue_data
        if((item_index=count.fetch_sub(1,std::memory_order_acquire))<=0)
        {
            wait_for_more_items();
            continue;
        }
        process(queue_data[item_index-1]);
    }
}

int main()
{
    std::thread a(populate_queue);
    std::thread b(consume_queue_items);
    std::thread c(consume_queue_items);
    a.join();
    b.join();
    c.join();
}
```

### fences (memory barriers)

***Fences*** (also commonly called ***memory barriers***) are typically used to enforce memory-ordering constraints on `memory_order_relaxed` operations without modifying any data

it’s important to note that the **synchronization point** is the ***fence*** itself.  

```C++
extern "C" void atomic_thread_fence( std::memory_order order ) noexcept;
```

1. **Fence-atomic synchronization**
    1. A **release fence** F is ***sequenced-before*** an **atomic store** X (with any memory order) in thread A
    2. An **atomic acquire operation** Y in thread B reads the value written by X
    3. Then F ***synchronizes-with*** Y
1. **Atomic-fence synchronization**
    1. An **atomic read** Y (with any memory order) is ***sequenced-before*** an **acquire fence** F in thread B
    2. Y reads the value written by an **atomic release operation** X in thread A
    3. Then X ***synchronizes-with*** F
1. **Fence-fence synchronization**
    1. A **release fence** FA is ***sequenced-before*** an **atomic write** X (with any memory order) in thread A
    2. An **atomic read** Y (with any memory order) is ***sequenced-before*** a **acquire fence** FB in thread B
    3. Y reads the value written by X
    4. Then FA ***synchronizes-with*** FB

### Ordering non-atomic operations  

The real benefit of using **atomic operations** to enforce an ordering is that they can enforce an ordering on **non-atomic operations** and avoid the **undefined behavior** of a **data race**,   

This is also the basis for the higher-level synchronization facilities in the `C++ Standard Library`, such as ***mutexes*** and ***condition variables***  

# chapter 6. Designing lock-based concurrent data structures  
If a data structure is to be accessed from multiple threads, 

1. either it must be completely immutable  
2. or use a separate mutex and external locking to protect the data,  
3. or data structure itself for concurrent access  

The design of **lock-based concurrent data structures** is all about

1. ***thread-safe***: ensuring that the right mutex is locked when accessing the data 
2. minimize ***serialization***: the lock is held for the minimum amount of time  
3. <font color=red>Attention for **exception-safe**</font>

If you use separate mutexes to protect separate parts of the data structure, there’s now also the possibility of **deadlock** if the operations on the data structure require more than one mutex to be locked.

The ***constructors*** and ***destructors*** cannot be protected by ***mutex***, but this isn’t a problem because the object can be **constructed only once** and **destroyed only once** and the user must ensure that other threads aren’t able to access until it’s fully constructed and have ceased accessing before it’s destroyed  

# chapter 7. Designing lock-free concurrent data structures  

There aren’t any locks with ***lock-free data structures***, but still possibility of ***live locks*** when two
threads each try to change the data structure. ***Live locks*** are typically short-lived because they depend on the exact scheduling of threads and can be avoided by ***wait-free code***

A ***wait-free data structure*** is a ***lock-free data structure*** with the additional property that
every thread accessing the data structure can complete its operation within a bounded number of steps, regardless of the behavior such as ***clashes*** of other threads.

**Primary reason** for using ***lock-free data structures*** is to enable maximum concurrency  A second reason is ***robustness***. If a thread dies while holding a lock, that data structure is broken forever.  

Although ***lock-free and wait-free code*** can increase the potential for concurrency and reduce
the time an individual thread spends waiting, it may well **decrease overall performance** because the **atomic operations** used for ***lock-free code*** can be much slower than **non-atomic operations** and the hardware must synchronize data between threads that access the same **atomic variables**  

guidelines for writing ***lock-free data structures***

1.  start with `std::memory_order_seq_cst` and only relaxed the **memory-ordering constraints** as **optimization** once the basic operations were working.
2.  One of the biggest difficulties with ***lock-free code*** is **managing memory**, the key idea is to use some method to keep track of how many threads are accessing a particular object and only delete each object when it’s no longer referenced from anywhere  
    1. Waiting until no threads are accessing the data structure and deleting all objects that are pending deletion
    2. Using **hazard pointers** to identify that a thread is accessing a particular object
    3. **Reference counting** the objects
    4. Using a **garbage collector**
3.  watch out for the ***ABA problem***  
4.  identify **busy-wait loops** and help the other thread  

# chapter 8. Designing concurrent code  

With multi-threaded code, not only must you think about the usual factors, such as encapsulation, coupling, and cohesion, but you also need to consider which data to share, how to synchronize accesses to that data, which threads need to wait for which other threads to complete certain operations, and so on.  

## Techniques for dividing work between threads  

1. Dividing data between threads before processing begins  
2. Dividing data recursively such as Quicksort algorithm  
3. Dividing work by task type such as ***pineline*** where data is dynamically generated or is coming from external input

## Factors affecting the performance of concurrent code  

### Oversubscription and excessive task switching  

In multithreaded systems, it’s typical to have more threads than processors, but threads often spend time waiting for external I/O to complete, blocked on mutexes, waiting for condition variables, and
so forth, so this isn’t a problem.  

However too many threads ready to run rather than ***suspending*** will waste processor time switching between the threads, called ***oversubscription***.  

C++11 Standard Thread Library provides `std::thread::hardware_concurrency()`  

### Data contention and cache ping-pong  

If two threads are executing concurrently on different processors and one of the threads modifies shared data, this change then has to propagate to the cache on the other core, which takes time. This may cause the second processor to stop in its tracks and wait for the change to propagate through the
memory hardware. 

In terms of CPU instructions, this can be a phenomenally slow operation, equivalent to many hundreds of individual instructions, although the exact timing depends primarily on the physical structure of the hardware. 

The shared data passed back and forth between the caches of different processors many times, called ***cache ping-pong***, will cause a processor has to wait for a cache transfer, and can’t do any work in the meantime. This situation is called ***high contention***  

The effects of ***contention*** with ***mutexes*** is to serialize threads at the **operating system level**, different from the effects of ***contention*** with **atomic operations** at the ***processor level***  

### False sharing / True sharing (data proximity)

Processor caches don’t generally deal in individual memory locations; instead, they deal in blocks of memory called ***cache lines***  

If data owned by different threads lies in the same **cache line**, called ***false sharing***, the cache hardware still has to play cache ping-pong even though each thread only accesses its own data

The C++17 standard defines `std::hardware_destructive_interference_size` in the header `<new>`,  specifies the **maximum number of consecutive bytes** that may be subject to ***false sharing*** for the current compilation target  

When the processor switches threads, it’s more likely to have to reload the ***cache lines*** if each thread uses data spread across multiple cache lines than if each thread’s data is close together in the same ***cache line***  

So the data accessed by a single thread should share ***cache lines*** as possible, called ***data proximity***. The C++17 standard specifies the constant `std::hardware_constructive_interference_size`, which is the **maximum number of consecutive bytes** guaranteed to be on the same ***cache line***    

## Designing data structures for multithreaded performance  
The key things to bear in mind when designing your data structures for multi-threaded performance are ***contention***, ***false sharing***, and ***data proximity*** which can be improved by altering the data layout or changing which data elements are assigned to which thread.   

## Additional considerations when designing for concurrency  

1. exception safety

2. scalability (the performance increase as more processing cores are added to the system)
   
    ***Amdahl’s law*** where P is performance gain of N processors and fs is the “serial” fraction of program,
    $$
    P = \frac{1}{f_s + \frac{1-f_s}{N}}
    $$
    
3. Hiding latency with multiple threads by non-blocking call such as asynchronous I/O 

# chapter 9. Advanced thread management 
## Thread pools 

several key design issues when building a **thread pool**, such as how many threads to use, the most efficient way to allocate tasks to threads, and whether or not you can wait for a task to complete 

A simple implementation of this thread pool is as follows

```C++
#include <thread>
#include <vector>
#include <atomic>
struct join_threads
{
    join_threads(std::vector<std::thread>&)
    {}
};

class thread_pool
{
    std::atomic_bool done;
    thread_safe_queue<std::function<void()> > work_queue;
    std::vector<std::thread> threads;
    join_threads joiner;

    void worker_thread()
    {
        while(!done)
        {
            std::function<void()> task;
            if(work_queue.try_pop(task))
            {
                task();
            }
            else
            {
                std::this_thread::yield();
            }
        }
    }
public:
    thread_pool():
        done(false),joiner(threads)
    {
        unsigned const thread_count=std::thread::hardware_concurrency();
        try
        {
            for(unsigned i=0;i<thread_count;++i)
            {
                threads.push_back(
                    std::thread(&thread_pool::worker_thread,this));
            }
        }
        catch(...)
        {
            done=true;
            throw;
        }
    }

    ~thread_pool()
    {
        done=true;
    }

    template<typename FunctionType>
    void submit(FunctionType f)
    {
        work_queue.push(std::function<void()>(f));
    }
};
```

To wait for task, pass `std::packaged_task<>` instances to work_queue, and use a ***type-erasing*** function wrapper instead of `std::function<>` because `std::function` requires that the stored function objects are ***copy-constructible*** and `std::packaged_task<>`  is move-only types  

```C++
#include <deque>
#include <future>
#include <memory>
#include <functional>
#include <iostream>
#include <iostream>

class function_wrapper
{
    struct impl_base {
        virtual void call()=0;
        virtual ~impl_base() {}
    };
    std::unique_ptr<impl_base> impl;
    template<typename F>
    struct impl_type: impl_base
    {
        F f;
        impl_type(F&& f_): f(std::move(f_)) {}
        void call() { f(); }
    };
public:
    template<typename F>
    function_wrapper(F&& f):
        impl(new impl_type<F>(std::move(f)))
    {}

    void call() { impl->call(); }

    function_wrapper(function_wrapper&& other):
        impl(std::move(other.impl))
    {}

    function_wrapper& operator=(function_wrapper&& other)
    {
        impl=std::move(other.impl);
        return *this;
    }

    function_wrapper(const function_wrapper&)=delete;
    function_wrapper(function_wrapper&)=delete;
    function_wrapper& operator=(const function_wrapper&)=delete;
};

class thread_pool
{
public:
    std::deque<function_wrapper> work_queue;

    template<typename FunctionType>
    std::future<typename std::result_of<FunctionType()>::type>
    submit(FunctionType f)
    {
        typedef typename std::result_of<FunctionType()>::type result_type;

        std::packaged_task<result_type()> task(std::move(f));
        std::future<result_type> res(task.get_future());
        work_queue.push_back(std::move(task));
        return res;
    }
    // rest as before
};
```

## Interrupting threads 

In many situations it’s desirable to signal to a long-running threa to stop because the pool is
now being destroyed, or because the work being done by the thread has been explicitly canceled by the user 

The C++11 Standard doesn’t provide this mechanism 

You have to insert interruption points in working threads, but interruption points do not work for threads which are blocked waiting for something 

For threads waiting for something like ***condition variable***, ***mutex locks***, and ***futures***, you have to **poll and wait for timeout**

# chapter 10. Parallel algorithms 

The C++17 standard added the concept of ***parallel algorithms*** with the addition of a new first parameter, which specifies the ***execution policy*** to the C++ Standard Library

## Execution policies 

The standard specifies three **execution policy types** and corresponding **global execution policy objects**

1. `std::execution::sequenced_policy` / `std::execution::seq `
2. `std::execution::parallel_policy` / `std::execution::par`
3. `std::execution::parallel_unsequenced_policy`  / `std::execution::par_unseq `

You cannot construct **execution policy** objects yourself, except by copying from the three global ones, because they might have special **initialization requirements** 

You cannot define your own ***execution policies*** 

The **execution policy** affects several aspects of algorithm behavior: 

1. The **algorithm’s complexity**: many parallel algorithms will perform more of the **core operations** of the algorithm (whether ***swaps***, ***comparisons***, or applications of a supplied function object), with the intention to provide an overall performance improvement
2. The behavior when an exception is thrown: All the standard-supplied ***execution policies*** will call `std::terminate` if there are **any uncaught exceptions** except that `std::bad_alloc` will be thrown 
3. Where, how, and when the steps of the algorithm are executed: the only aspect that differs between the ***standard execution policies***

### std::execution::sequenced_policy

The sequenced policy forces the implementation to perform all operations on the thread that called the function. And all operations must be performed in some definite but unspecified order, so they are not interleaved. The precise order may be different between different invocations of the function 

### std::execution::parallel_policy 

This is ***permission***, not a ***requirement***—the library may still execute the code on a single thread if it wishes 

All operations must be performed in some definite but unspecified order, and not interleaved. The precise order may be different between different invocations of the function 

This imposes **additional requirements** on the iterators, values, and callable objects used with the algorithm: 

1. they must not cause data races if invoked in parallel
2. must not rely on being run on the same thread as any other operation, or indeed rely on not being run on the same thread as any other operation. 

### std::execution::parallel_unsequenced_policy 

The algorithm steps may be performed on unspecified threads of execution, unordered and unsequenced
with respect to one another. This means that operations may now be interleaved with each other on a single thread, such that a second operation is started on the same thread before the first has finished, and may be migrated between threads 

The parallel unsequenced policy provides the greatest scope for parallelizing the algorithm in exchange for imposing the strictest requirements: the iterators, values, and callable objects used with the algorithm must not use any form of synchronization or call any function that synchronizes with another, or any function such that some other code synchronizes with it such as mutexes or atomic variables and must not modify any shared state 

## The parallel algorithms from the C++ Standard Library 

The C++ Standard Library defines five categories of iterators:

1. ***Input Iterators*** are single-pass iterators for retrieving values.
2. ***Output Iterators*** are single-pass iterators for writing values.
3. ***Forward Iterators*** are multipass iterators for one-way iteration through persistent
   data.
4. ***Bidirectional Iterators*** are multipass iterators like ***Forward Iterators***, but they can also
   be made to go backward to access previous elements.
5. ***Random Access Iterators*** are multipass iterators that can go forward and backward
   like ***Bidirectional Iterators***, but they can go forward and backward in steps larger than a single element, and you can use the array index operator to directly access elements at an offset

The algorithms overload with an execution policy may denote stronger constraints on template parameter types 

```C++
template<class InputIterator, class OutputIterator>
OutputIterator copy(
				InputIterator first, InputIterator last, OutputIterator result);

template<class ExecutionPolicy,
class ForwardIterator1, class ForwardIterator2>
ForwardIterator2 copy(
					ExecutionPolicy&& policy,
					ForwardIterator1 first, ForwardIterator1 last,
					ForwardIterator2 result);
```

# chapter 11. Testing and debugging multithreaded applications 
Testing and debugging concurrent code is hard. 

## Types of concurrency-related bugs 

Typically, these concurrency-related bugs fall into two categories:

1. Unwanted blocking
    1. ***Deadlock***
    2. ***Livelock***: livelock, such as a spin lock, will eventually resolve because of the random scheduling, but there will be a long delay with a high CPU usage during that delay
    3. ***Blocking on I/O or other external input***
2. Race conditions 
    1. ***Data races***: because of unsynchronized concurrent access to a shared memory location
    2. ***Broken invariants***
    3. ***Lifetime issues***

## Techniques for locating concurrency-related bugs 

