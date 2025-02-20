[TOC]

# chapter 1. Introduction to function programming

**Functional programming** is a style of programming that emphasizes the **evaluation of expressions**, rather than **execution of commands**. <font color=red>The expressions in these languages are formed by using functions to combine basic values.</font>

***imperative programming*** vs ***declarative programming***  

syntactic sugar such as the ***auto*** keyword and **lambdas**, and new features such as ***ranges***, ***concepts***, and ***coroutines*** make C++ programming in the functional style much easier 

## Pure functions
a **pure function** is any function that doesn’t have observable (at a higher level) side effects  

**Avoiding mutable state** improves the correctness of code and removes the need for mutexes in multit-hreaded code.


## Thinking functionally
Thinking functionally means thinking about the input data and the transformations you need to perform to get the desired output


## Benefits of functional programming  

1. Code brevity and readability: code is much shorter 
2. Concurrency and synchronization because pure functions have no shared mutable state 
3. Continuous optimization due to improvement in programming language, compiler implementation, or the implementation of the library   

# chapter 2. Getting started with functional programming  

## Higher-order functions  

The main feature of all **functional programming (FP) languages** is that functions can be treated like ordinary values  

Functions that take other functions as arguments or that return new functions are called **higher-order functions**

**higher-order functions**, such as STL algorithms are usually implemented by **loop, recursion or fold**

Since C++17, many STL algorithms let you specify that their execution should be parallelized by execution policy `std::execution::par`

## folding (or reduction)  

The `std::accumulate` algorithm is an implementation of a general concept called ***folding (or reduction)***. This is a **higher-order function** that abstracts the process of iterating over recursive structures  

***folding*** is nothing more than a nicer way to write tail-recursive functions that iterate over a collection  

C++ doesn’t provide a separate algorithm for **right folding**, but you could pass **reverse iterators** (***crbegin*** and ***crend***) to `std::accumulate` to achieve the same effect.  
$$
\text{left folding:}\  ((((r0\nabla t1)\nabla t2)\nabla t3) ...\nabla tn)\\
\text{right folding:}\ (t1 \nabla... (tn–2 \nabla(tn–1\nabla (tn\nabla r0 ))))
$$

## example using STL

Although the **standard algorithms** provide a way to write code in a more **functional manner**, they aren’t designed to, so not so efficient

```C++
std::vector<person_t> females;
std::copy_if(people.cbegin(), people.cend(),
    std::back_inserter(females),
    is_female);
std::vector<std::string> names(females.size());
std::transform(females.cbegin(), females.cend(),
            names.begin(),
            name); // Transformation function
```

# chapter 3. Function objects

**Automatic return type deduction** is mainly useful when writing function templates in which the return type depends on the types of the arguments and `decltype(auto)` is useful for generic functions that forward the result of another function without modifying it. But <font color=red>all of **return statement** must return the same type and no any conversion</font>

Classes with the overloaded call operator are often called ***functors***.
This term is highly problematic because it’s already used for [something different in ***category theory***](#factor)

```C++
class older_than {
  public:
    older_than(int limit)
    	: m_limit(limit)
    {
    }
    bool operator()(const person_t& person) const
    {
    	return person.age() > m_limit;
    }
  private:
    int m_limit;
};
// more general
class older_than {
  public:
    older_than(int limit)
    	: m_limit(limit)
    {
    }
    template <typename T>
    bool operator()(T&& object) const
    {
    	return std::forward<T>(object).age()> m_limit;
    }
  private:
    int m_limit;
};
```

## Lambdas and closures

Lambdas are useful **syntactic sugar** allowing you to create function objects ***inline***  

When a lambda is created, the compiler creates a class with a call operator and gives that class an internal name. That name isn’t accessible to the programmer except that declare the variable by ***auto***, and different compilers give it different names  

call operator of lambdas is **constant by default**, you can declare the lambda as `mutable` but should avoid

<font color=red>In ***capture-list*** of lambdas, you can define arbitrary member variables by specifying the variable name and its initial value separately. The type of the variable is automatically deduced from the specified value. </font>

```C++
request.on_completed(
        [ session = std::move(session),
          time = current_time()
        ]
        (response_t response)
        {
            std::cout
                << "Got response: " << response
                << " for session: " << session
                << " the request took: "
                << (current_time() - time)
                << "milliseconds";
        }
        );
```

**Generic lambdas** using ***auto***, are closures with **call operator template**

```C++
auto predicate = [limit = 42](auto&& object) {
    return object.age() > limit;
};
// all the arguments of the same type, but deduced automatically,
[] (auto first, decltype(first) second) { … }
```

## Boost.Phoenix  

Boost library defines **magic argument placeholders**, and operators that allow you to compose them , which is most useful for writing simple ones but will **slow the compilation time significantly**  

```C++
using namespace boost::phoenix::arg_names;
std::vector<int> numbers{21, 5, 62, 42, 53};
std::partition(numbers.begin(), numbers.end(),
				arg1 <= 42); // arg1 <= 42 return a unary function object
// numbers now contain {21, 5, 42, 62, 53}
// <= 42 > 42
```

## Wrapping function objects with `std::function`

class template `std::function` can wrap any type of function object including **class member variables** and **class member functions**.  

`std::function` uses a technique known as **type erasure** based on virtual member function calls which introduces **noticeable performance penalties**. In addition `std::function` can invoke a **non-constant callable** although its call operator is marked as **const**. This can lead to all kinds of problems in multi-threaded code  

```C++
std::string str{"A small pond"};
std::function<bool(std::string)> f;
f = &std::string::empty;
std::cout << f(str);
```

# chapter 3. Creating new functions from the old ones

## partial function application

**partial function application**: creating a new function object from an existing one by fixing one or more of its arguments to a specific value

### use template

cannot work with **pointers to member functions and member variables** except using `std::invoke`

```C++
template <typename Function, typename SecondArgType>
class partial_application_bind2nd_impl {
public:
    partial_application_bind2nd_impl(Function function,
    								SecondArgType second_arg)
        : m_function(function)
        , m_value(second_arg)
    {
    }

    template <typename FirstArgType>
    auto operator()(FirstArgType&& first_arg) const
    			-> decltype(m_function(
    						std::forward<FirstArgType>(first_arg),
    						m_second_arg))
    {
        return m_function(
        			std::forward<FirstArgType>(first_arg),
        			m_second_arg);
    }

private:
    Function m_function;
    SecondArgType m_second_arg;
};

// Before C++17 create a function template whose job is only to make compiler deduce the types automatically
template <typename Function, typename SecondArgType>
partial_application_bind2nd_impl<Function, SecondArgType>
bind2nd(Function&& function, SecondArgType&& second_arg)
{
    return partial_application_bind2nd_impl<Function, SecondArgType>(
    						std::forward<Function>(function),
    						std::forward<SecondArgType>(second_arg));
}
```

### Using `std::bind` to bind values to specific function arguments 

In C++11, function `std::bind1st` and `std::bind2nd` were deprecated (removed in C++17) and introduces `std::bind` instead

`std::bind` introduces the concept of ***placeholders*** similarly to Boost.Phoenix library

`std::bind` can bind arguments of any callable 

Member functions are the same as ordinary ones, except they have an implicit first argument this that points to the instance of the class the member function was called on 

By default, `std::bind` stores the **copies of the bound values** in the function object it returns unless using `std::ref`

```C++
auto is_greater_than_42 =
	std::bind(std::greater<double>(), _1, 42);
auto is_less_than_42 =
	std::bind(std::greater<double>(), 42, _1);
is_less_than_42(6); // returns true
is_greater_than_42(6); // returns false

void print_person(const person_t& person,
                std::ostream& out,
                person_t::output_format_t format);

std::for_each(people.cbegin(), people.cend(),
                std::bind(print_person,
                        _1,
                        std::ref(std::cout),
                        person_t::name_only
                ));

class person_t {
	void print(std::ostream& out, output_format_t format) const {}
};

std::for_each(people.cbegin(), people.cend(),
                std::bind(&person_t::print,
                        _1,
                        std::ref(std::cout),
                        person_t::name_only
                ));
```

### Using **lambdas** as an alternative for `std::bind`

**lambdas** are a core-language feature which can be optimized easily by compiler while `std::bind` stores the copies of all values, references, functions and information about which placeholders are use in the function object, making it slower than an ordinary lambda if the compiler doesn’t manage to optimize it properly. 

## Currying 

**Currying** is particularly useful in generic contexts when you can be given a function with any number of arguments. 

convert function to its **curried version** by nesting enough **lambda expressions** to capture all the arguments needed 

```C++
void print_person(const person_t& person,
                std::ostream& out,
                person_t::output_format_t format);

auto print_person_cd(const person_t& person)
{
    return [&](std::ostream& out) {
        return [&](person_t::output_format_t format) {
            print_person(person, out, format);
        };
    };
}
```

# chapter 5. Purity: Avoiding mutable state

one of the main reasons for the existence of bugs is that it’s hard to manage all the states a program can be in. In the **object-oriented world**, we tend to approach this issue by encapsulating parts of the program state into objects. 

an expression is **referentially transparent** if the program wouldn’t behave any differently if we replaced the entire expression with just its return value, that is **no observable side effects**. In a more relaxed manner, **no observable side effects you care about**

In **pure functional programming**, instead of changing an object, you create a copy of object with some properties changed to new value 

***this*** is a **constant pointer** to an instance of the object the member function belongs to that can never be null (although, in reality, some compilers allow calling **nonvirtual member functions** on **null pointers**). 

***this*** remains pointer type because it was introduced when C++ didn’t have references and change to reference would break **backward compatibility** with the previous language versions

```C++
// reduce the performance overhead
class person_t {
public:
	person_t with_name(const std::string& name) const & {}
    person_t with_name(const std::string& name) && {}  // called on temporaries
}
```

Mutable state is safe as long as it isn’t shared between multiple system components at the same time. 

# chapter 6. Lazy evaluation 

## Laziness in C++ 

C++ doesn’t support **lazy evaluation** but can simulate this behavior by creating a template class called `lazy_val` that can store a computation and cache the result of that computation after executed (called **memoization**)

```C++
#include <functional>
#include <string>
#include <iostream>
#include <mutex>
#include <optional>

template <typename F>
class lazy_val {
private:
    F m_computation;

    // In the book, we are using a value and a Boolean flag to denote
    // whether we have already calculated the value or not. In this
    // implementation, we are using std::optional which is like a container
    // that can either be empty or it can contain a single value --
    // exactly what we tried to simulate with the value/Boolean flag pair.
    // Optional values will be coveref in more detail in chapter 9.
    mutable std::optional<decltype(m_computation())> m_value;
    mutable std::mutex m_value_lock;

public:
    lazy_val(F function)
        : m_computation(function)
    {
    }

    lazy_val(lazy_val &&other)
        : m_computation(std::move(other.m_computation))
    {
    }

    operator decltype(m_computation()) () const
    {
        std::lock_guard<std::mutex> lock(m_value_lock);

        if (!m_value) {
            m_value = std::invoke(m_computation);
        }

        return *m_value;
    }

};

template <typename F>
inline lazy_val<F> make_lazy_val(F&& computation)
{
	return lazy_val<F>(std::forward<F>(computation));
}

int main(int argc, char *argv[])
{
    int number = 6;

    auto val = make_lazy_val( [=] {
        std::cout << "Calculating the answer..." << std::endl;
        std::cout << "while the number is: " << number << std::endl;
        return 42;
    });

    number = 2;

    std::cout << "Lazy value defined" << std::endl;

    std::cout << val << std::endl;
}
```

use `std::call_once` function: instead of a mutex, which is a general low-level synchronization primitive 

```C++
template <typename F>
class lazy_val {
private:
    F m_computation;

    mutable std::optional<decltype(m_computation())> m_value;
    mutable std::once_flag m_value_flag;

    operator decltype(m_computation()) () const
    {
        std::call_once(m_value_flag, [this] {
       		m_value = std::invoke(m_computation);
        });
        return *m_value;
    }
};
```

## Laziness to optimize **dynamic programming**

**lazy quicksort algorithm**: get the top k items of the collection of n, complexity  `O(n + k log k) `

Cache all previously calculated results to optimize ***dynamic programming***

### Generalized memoization 

```C++
template <typename Result, typename... Args>
auto make_memoized(Result (*f)(Args...))
{
    std::map<std::tuple<Args...>, Result> cache;

    return [f, cache](Args... args) mutable -> Result
    {
        const auto args_tuple =
            std::make_tuple(args...);
        const auto cached = cache.find(args_tuple);

        if (cached == cache.end()) {
            auto result = f(args...);
            cache[args_tuple] = result;
            return result;
        } else {
            return cached->second;
        }
    };
}

unsigned int fib(unsigned int n)
{
    std::cout << "Calculating " << n << "!\n";
    return n == 0 ? 0
         : n == 1 ? 1
         : fib(n - 1) + fib(n - 2);
}

int main(int argc, char* argv[])
{
    auto fibmemo = make_memoized(fib);

    std::cout << "15! = " << fibmemo(15) << std::endl;  // no optimization for first call
    std::cout << "15! = " << fibmemo(15) << std::endl;  // use caches
    std::cout << "16! = " << fibmemo(16) << std::endl;  // no any optimizatio
    return 0;
}
```

if want to **memoize every recursive call**, should pass the wrapped one into origin function

```C++
template <typename F>
unsigned int fib(F&& fibmemo, unsigned int n)
{
    return n == 0 ? 0
            : n == 1 ? 1
            : fibmemo(n - 1) + fibmemo(n - 2);
}
```

complete code

```C++
#include <iostream>
#include <map>
#include <mutex>
#include <tuple>
#include <type_traits>
#include <unordered_map>

namespace detail {

class null_param {};

template <class Sig, class F>
class memoize_helper;

template <class Result, class... Args, class F>
class memoize_helper<Result(Args...), F> {
private:
    using function_type = F;
    using args_tuple_type
        = std::tuple<std::decay_t<Args>...>;

    function_type f;

    mutable std::map<args_tuple_type, Result> m_cache;
    mutable std::recursive_mutex m_cache_mutex;

public:
    // The constructors need to initialize the wrapped function.
    // You could made copy-constructor copy the cached values as well,
    // but that’s not necessary.

    template <typename Function>
    memoize_helper(Function&& f, null_param)
        : f(f)
    {
    }

    template <class... InnerArgs>
    Result operator()(InnerArgs&&... args) const
    {
        std::unique_lock<std::recursive_mutex>
                lock{m_cache_mutex};

        // Searches for the cached value
        const auto args_tuple =
            std::make_tuple(args...);
        const auto cached = m_cache.find(args_tuple);

        //  If the cached value is found, returns it without calling f
        if (cached != m_cache.end()) {
            return cached->second;

        } else {
            // If the cached value isn’t found, calls f and stores
            // the result. Passes *this as the first argument: the
            // function to be used for the recursive call.
            auto&& result = f(
                    *this,
                    std::forward<InnerArgs>(args)...);
            m_cache[args_tuple] = result;
            return result;
        }
    }
};

} // namespace detail

using detail::memoize_helper;

template <class Sig, class F>
memoize_helper<Sig, std::decay_t<F>>
make_memoized_r(F&& f)
{
    return {std::forward<F>(f), detail::null_param{}};
}


int main(int argc, char* argv[])
{
    auto fibmemo = make_memoized_r<
                unsigned int(unsigned int)>(
        [](auto& fib, unsigned int n) {
            std::cout << "Calculating " << n << "!\n";
            return n == 0 ? 0
                 : n == 1 ? 1
                 : fib(n - 1) + fib(n - 2);
        });

    std::cout << "15! = " << fibmemo(15) << std::endl;       // <1>
    std::cout << "15! = " << fibmemo(15) << std::endl;       // <2>

    return 0;
}
```

## Expression templates

<font color=red>**laziness requires purity **</font>

**Pure functions** give you the same result whenever called with the same arguments. And that’s why you can delay executing them without any consequences 

# chapter 7. Ranges

The main issue of iterators is that STL algorithms take iterators to the beginning and end of a collection as separate arguments instead of taking the collection itself and passing pairs of iterators is also error prone 

Indeed, It doesn’t really need an end iterator, but just something, called ***sentinel***, to test whether you’re at the end.

range libraries usually provide a special **pipe syntax** that overloads the **| operator**, inspired by the UNIX shell pipe operator 

## Views and actions for ranges 

***View*** transformations such as `filter` and `transform` created a new view of existing data, evaluated lazily 

***Action*** transformations such as `sorting`, do changing the original collection.

A ***view*** transformation creates a **lazy view** over the original data useful for items don’t need to be processed more than once, whereas an ***action*** works on an existing collection and performs its transformation useful for items accessed often 

## delimited ranges and infinite ranges with sentinels 

A ***delimited range*** is one whose end you don’t know in advance—but you have a predicate function that can tell you when you’ve reached the end 

The **range-based for loop**, as defined in C++11 and C++14, requires both the `begin` and `end` to have the same type, which was removed in C++17. So in C++17, **sentinel-based ranges** can be used with the **range-based for loop**.

can easily create **infinite ranges with sentinels**, making code more generic when designing code

# chapter 8. Functional data structure

creates a clear separation between the **pure parts** of the program and parts with the **mutable state** 

<font color=red>The main optimization of **immutable data structures**, which are also called ***persistent***, is **data sharing** </font>
<font color=red>All immutable data structures have downsides even if as magical as ***bitmapped vector tries***</font>
Immutable data structures have always been an active area of research

## Immutable linked lists 

For simple case of **immutable single-linked list**, it's efficient only for add and remove from the beginning of a list, and suitable for implementing stack-like structures 

Most functional programming languages rely on **garbage collection** to free up the memory that’s no longer used. In C++, you can achieve the same by using **smart pointers** with the problem that all **destructors** are invoked recursively and might cause **overflowing the stack**, solved by providing a **custom destructor**. 

```C++
template <typename T>
class list {
public:
…
private:
    struct node {
        T value;
        std::shared_ptr<node> tail;
    };
    std::shared_ptr<node> m_head;
};
```

## Immutable vector-like data structures 

`std::vector` is not suitable for immutable structure because it needs to copy all data every time modified. One of the popular alternatives is the ***bitmapped vector trie*** (***prefix tree***) 

***bitmapped vector trie*** is a tree structure whose **leaf node** is **copy-on-write vector** with **maximum size** ***m*** and **non-leaf nodes** hold no data but pointers to the nodes in the lower level. Two levels can contain at most `m × m × m` items.

Copy for ***bitmapped vector trie*** just nedd to copy all **non-leaf nodes**

Element lookup in ***bitmapped vector tries*** just like an array of bits, the complexity is logarithmic with logarithm base ***m***, so usually choose ***m*** to be a power of 2 

**Appending elements** / **Removing elements from the end**  / **Updating elements** is just lookup plus **copy-on-write operation **

Other operations like concatenating、prepending or inserting at a position have the same complexities as `std::vector`

# chapter 9. Algebraic data types and pattern matching

## Algebraic data types 

**A product of two types** is a new type that contains instances of both, realized by **aggregation**, or `std::pair` and `std::tuple`, which store value with no names and should be used locally and not in public API

**A sum of two types** is a new type that holds instances of any one but not both at the same time, 

**Enums** are a special kind of sum type, **sum types** is a generalization of an enum 

**open sum types**: realized by **inheritance**, using **dynamic dispatch**, and the **visitor pattern** with **runtime performance penalties** but can be **easily extended**. 

**closed sum types**: sum types that hold exactly the types you specify and nothing else, realized by `std::variant ` a type-safe implementation of **unions** introduced in C++17 

## Domain modeling with algebraic data types 

The main idea when implementing program state through **algebraic data types** is to minimize the number of states and make illegal states impossible to represent, requiring some thinking and longer code

The idea of having a type that contains either a value or an error has been popular in the functional programming 

### pattern matching 
The main issue when implementing programs with algebraic data types is that every time you need a value, you have to check and then extract it from its wrapper type 

***switch-statement*** works only on **integer-based types**, but you might want a combination of a **type check**, a **value check**, and a **custom predicate**, which is what **pattern matching** do in most functional programming language 

C++ provides ways to do **pattern matching** for template metaprogramming and for normal programs, you can use `std::visit` function with overloaded function object that will separate implementations for different types;  

```C++
template <typename... Ts>
struct overloaded : Ts... { using Ts::operator()...; };

template <typename... Ts> overloaded(Ts...) -> overloaded<Ts...>;

void point_for(player which_player)
{
    std::visit(
        overloaded {
            [&](const normal_scoring& state) {
            // Increment the score, or switch the state
            },
            [&](const forty_scoring& state) {
            // Player wins, or we switch to deuce
            },
            [&](const deuce& state) {
            // We switch to advantage state
            },
            [&](const advantage& state) {
            // Player wins, or we go back to deuce
            }
        },
        m_state);
}
```

# chapter 10. Monads

<a id="functor"></a>

**functors** in FP is a concept from an abstract branch of mathematics called ***category theory***, quite different from that means a class with the call operator in C++

A **class template F** is a **functor** if there is such function `transform: (F<T1>, f:{T1->T2}) -> F<T2>`

One of the basic functors is the `std::optional` type 

```C++
template <typename T1, typename F>
auto transform(const std::optional<T1>& opt, F f)
    		-> decltype(std::make_optional(f(opt.value())))
{
    if (opt) {
    	return std::make_optional(f(opt.value()));
    } else {
    	return {};
    }
}
```

A **monad `M<T>`** is a **functor** that has an additional function `join: M<M<T>> → M<T> `, to deal with `f:{T1->M<T2>}` and `transform: (M<T1>, f)->M<M<F2>>`, that is, flatten out nested monadic values 

***ranges*** are **functors** and also **monads**  

# chapter 11. Template metaprogramming

Templates is a **Turing-complete programming language** executed at compile time and also a **pure functional language**, in which all variables are immutable, and no mutable state in any form. 

Pattern matching during compilation 

## Making curried functions 

currying allows you to treat multi-argument functions as unary functions 

```C++
template <typename Function, typename... CapturedArgs>
class curried {
private:
    using CapturedArgsTuple = std::tuple<
    							std::decay_t<CapturedArgs>...>;
    template <typename... Args>
    static auto capture_by_copy(Args&&... args)
    {
        return std::tuple<std::decay_t<Args>...>(
        			std::forward<Args>(args)...);
    }
public:
    curried(Function, CapturedArgs... args)
            : m_function(function)
            , m_captured(capture_by_copy(std::move(args)...))
    {
    }
    curried(Function, std::tuple<CapturedArgs...> args)
            : m_function(function)
            , m_captured(std::move(args))
    {
    }
    …
private:
    Function m_function;
    std::tuple<CapturedArgs...> m_captured;
};
```

## Calling all callables 

Due to limitation of the C++ core language, some callables, such as **pointers to member functions and member variables**, can’t be called as normal function although every member function is just an ordinary function, where the first argument is the implicit ***this*** pointer.

`std::invoke` was added to the standard library as a remedy for this limitation of the language

## DSL building blocks 

**domain-specific language (DSL)** is a **mini-language** used only in small domain, and doesn’t have to be generic, it significantly simplify the main program logic.

The main problem is that implementing all the necessary structures to represent **abstract syntax tree (AST)  nodes** can be tedious work and requires a lot of boilerplate code, making ***DSLs*** in C++ not popular 

***Ranges*** are also ***DSLs***, in a sense; with an ***AST*** for defining **range transformations** using the **pipe syntax**. 

# chapter 12. Functional design for concurrent systems

design software as a set of isolated, separate components.   

## The actor model: Thinking in components  

We should design classes to have a set of **actions or tasks** they can perform, and then add the data necessary to implement those actions instead of thinking first about what data an object contains

In the **actor model**, actors are completely isolated entities that share nothing but can send messages to one another.   

The C++ [Actor Framework](http://actor-framework.org) has an impressive set of features. The actors are lightweight concurrent processes (much lighter-weight than threads), and network-transparent, meaning you can distribute actors over multiple separate machines without the need to change code

# chapter 13. Testing and debugging  

By using the higher-level abstractions covered in this book, you can move many common software bugs to compilation time

## Unit testing and pure functions  

***unit tests***, at the lowest level of testing, isolate small parts of a program and test them individually to assure their correctness. Testing **pure functions** is a bit easier because every **pure function** is already an isolated part of the program  

## Automatically generating tests  

Although unit tests are useful (and necessary), they are written by hand and error-prone. You can use test cases automatically generated using random-number generator in following manners

1. generating test cases by the inverse of tested functions
2. checking a set of properties for tested function 
3. comparing the result with other "standard" function
