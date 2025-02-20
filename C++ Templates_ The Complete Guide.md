[TOC]


# Chapter 1. Function Templates 

template instantiation Two-Phase Translation 

## Type Deduction for Default Arguments 

```C++
template<typename T>
void f(T = "");
f(1); // OK: deduced T to be int, so that it calls f<int>(1)
f(); // ERROR: cannot deduce T

template<typename T = std::string>
void f(T = "");
f(); // OK
```

### Deducing the Return Type 

Before C++14 

```C++
// the return type might be a reference type, if T is a reference
template<typename T1, typename T2>
auto max (T1 a, T2 b) -> decltype(b<a?a:b)
{
	return b < a ? a : b;
}

/*
std::decay<T>::type is
1. If T is array of U or reference to it, the member typedef type is U*
2. if T is a function type F or reference to one, the member typedef type is std::add_pointer<F>::type.
3. Otherwise is std::remove_cv<std::remove_reference<T>::type>::type.
*/
template<typename T1, typename T2>
auto max (T1 a, T2 b) -> typename std::decay<decltype(true?a:b)>::type
{
	return b < a ? a : b;
}
```

Since C++14

```C++
// an initialization of type auto always decays
template<typename T1, typename T2>
auto max (T1 a, T2 b)
{
	return b < a ? a : b;
}
```

### Return Type as Common Type 

`std::common_type<>` also decays 

```C++
typename std::common_type<T1,T2>::type // since C++11
std::common_type_t<T1,T2> // equivalent since C++14
```

## Overloading Function Templates 

```C++
// maximum of two int values:
int max (int a, int b)
{
	return b < a ? a : b;
}
// maximum of two values of any type:
template<typename T>
T max (T a, T b)
{
	return b < a ? a : b;
}
int main()
{
    ::max(7, 42); // calls the nontemplate for two ints
    ::max(7.0, 42.0); // calls max<double> (by argument deduction)
    ::max<>(7, 42); // calls max<int> (by argument deduction)
    ::max<double>(7, 42); // calls max<double> (no argument deduction)
    ::max(’a’, 42.7); // calls the nontemplate for two ints
}
```

template with return type argument

```C++
template<typename T1, typename T2>
auto max (T1 a, T2 b)
{
	return b < a ? a : b;
}
template<typename RT, typename T1, typename T2>
RT max (T1 a, T2 b)
{
	return b < a ? a : b;
}

auto a = ::max(4, 7.2); // uses first template
auto b = ::max<long double>(7.2, 4); // uses second template
auto c = ::max<int>(4, 7.2); // ERROR: both function templates match
```

notice return by reference

```C++
#include <cstring>
// maximum of two values of any type (call-by-reference)
template<typename T>
T const& max (T const& a, T const& b)
{
	return b < a ? a : b;
}
// maximum of two C-strings (call-by-value)
char const* max (char const* a, char const* b)
{
	return std::strcmp(b,a) < 0 ? a : b;
}
// maximum of three values of any type (call-by-reference)
template<typename T>
T const& max (T const& a, T const& b, T const& c)
{
	return max (max(a,b), c); // error if max(a,b) uses call-by-value
}
int main ()
{
    /* int const& max (int const& a, int const& b, int const& c) calls
    	int const& max (int const& a, int const& b) twice
    */
    auto m1 = ::max(7, 42, 68); // OK
    char const* s1 = "frederic";
    char const* s2 = "anica";
    char const* s3 = "lucas";
    /*
    char const* & max(char const* &, char const* &, char const* &) calls
    char const* max (char const* a, char const* b) twice
    */
    auto m2 = ::max(s1, s2, s3); // run-time ERROR, return refenece to temporary
}
```



## Why Not **inline** 

From a strict language definition perspective, ***inline*** only means that a definition of a function can appear multiple times in a program  

Unlike ordinary noninline functions, we can define noninline function templates in a header file and include this header file in multiple translation units. <font color=red>except for full specializations of templates for specific types, which must be defined in header files and ***inline***.</font>

Nowadays, compilers usually are better at deciding whether inline without the hint of inline keyword 



# Chapter 2. Class Templates 

1. until C++17 you must always specify the template arguments explicitly. 
2. C++17 introduced class argument template deduction, which allows skipping template arguments if they can be derived from the constructor.  

<font color=red>before C++11 you had to put whitespace between the two closing template brackets </font>

```C++
Stack<Stack<int> > intStackStack; // OK with all C++ versions
Stack<Stack<int>> intStackStack; // ERROR before C++11

C<42, sizeof(int) > 4> c; // ERROR: first > ends the template argument list
C<42, (sizeof(int) > 4)> c; // OK
```

## concept 

concept is often used to denote a set of constraints that is repeatedly required in a template library 

## Type Aliases 

```C++
// Alias Declarations
typedef Stack<int> IntStack; // typedef
using IntStack = Stack<int>; // alias declaration
// Alias Templates
template<typename T>
using DequeStack = Stack<T, std::deque<T>>;
// Alias Templates for Member Types
template<typename T>
using MyTypeIterator = typename MyType<T>::iterator;
```

## Class Template Argument Deduction 

<font color=red>unlike for function templates, class template arguments may not be deduced only partially (by explicitly specifying only some of the template arguments) </font>

1. when passing arguments of a template type T by reference, the parameter doesn’t decay 
2. when passing arguments of a template type T by value, the parameter decays 

# Chapter 3. Nontype Template Parameters 

## Restrictions
**nontype template parameters** can only be 

1.  **constant integral values** (including **enumerations**)
2.  ·std::nullptr_t·
3.  **pointers/lvalue references** to objects/functions which must have **static** lifetime

<font color=red>**nontype template parameter** declared as **reference or pointer** can be a **constant expression** with **external linkage** in all C++ versions, **internal linkage** since C++11, or **any linkage** since C++17. </font><a id="Nontype_Template_Parameters_Linkage"></a>

```C++
template<char const* name>
class MyClass {
	...
};
MyClass<"hello"> x; // ERROR: string literal "hello" not allowed

extern char const s03[] = "hi"; // external linkage
char const s11[] = "hi"; // internal linkage
int main()
{
    Message<s03> m03; // OK (all versions)
    Message<s11> m11; // OK since C++11
    static char const s17[] = "hi"; // no linkage
    Message<s17> m17; // OK since C++17
}
```
## use **auto** since C++17
```C++
#include <iostream>
template<auto T> // take value of any possible nontype parameter (since C++17)
class Message {
public:
void print() {
	std::cout << T << ’\n’;
	}
};
```

# Chapter 4. Variadic Templates 
```C++
template<typename T, typename... Types>
void print (T firstArg, Types... args)
{
    std::cout << firstArg << ’\n’;
    if (sizeof...(args) > 0) { // error if sizeof...(args)==0
    	print(args...); // print() for no arguments declared is needed for compilation 								though sizeof... is constexpr
    }
}
```

## Variadic Expressions 

```C++
template<typename... T>
void printDoubled (T const&... args)
{
	print(args + args...);
}

printDoubled(7.5, std::string("hello"), std::complex<float>(4,2));
print(7.5 + 7.5,
    std::string("hello") + std::string("hello"),
    std::complex<float>(4,2) + std::complex<float>(4,2); // equivalent

template<typename... T>
void addOne (T const&... args)
{
    print (args + 1...); // ERROR: 1... is a literal with too many decimal points
    print (args + 1 ...); // OK
    print ((args + 1)...); // OK
}
```


### Fold Expressions Since C++17

Since C++17, there is a feature to compute the result of using a **binary operator** over all the arguments of a **parameter pack** <font color=red>(an expression, such as function call,  that contains an unexpanded [parameter pack](https://en.cppreference.com/w/cpp/language/parameter_pack))</font> (with an optional initial value). 

| Fold Expression         | Evaluation                                     |
| ----------------------- | ---------------------------------------------- |
| ( ... op pack )         | ((( pack1 op pack2 ) op pack3 ) ... op packN ) |
| ( pack op ... )         | ( pack1 op ( ... ( packN-1 op packN )))        |
| ( init op ... op pack ) | ((( init op pack1 ) op pack2 ) ... op packN )  |
| ( pack op ... op init ) | ( pack1 op ( ... ( packN op init )))           |

```C++
template<typename T>
class AddSpace
{
    private:
    	T const& ref; // refer to argument passed in constructor
    public:
    	AddSpace(T const& r): ref(r) { }
    	friend std::ostream& operator<< (std::ostream& os, AddSpace<T> s) {
    		return os << s.ref << ’ ’; // output passed argument and a space
    		}
};
template<typename... Args>
void print (Args... args) {
	( std::cout << ... << AddSpace(args) ) << ’\n’;
}
```

same rules apply to variadic function template parameters as for ordinary parameters 

```C++
// args are copies with decayed types:
template<typename... Args> void foo (Args... args);
// args are nondecayed references to passed objects:
template<typename... Args> void bar (Args const&... args);
```

## Variadic Deduction Guides 

```C++
namespace std {
    template<typename T, typename... U> array(T, U...)
    	-> array<enable_if_t<(is_same_v<T, U> && ...), T>,
    		(1 + sizeof...(U))>;
}
```

## Variadic Base Classes and using 

```C++
#include <string>
#include <unordered_set>
class Customer
{
    private:
    	std::string name;
    public:
        Customer(std::string const& n) : name(n) { }
        std::string getName() const { return name; }
};
struct CustomerEq {
    bool operator() (Customer const& c1, Customer const& c2) const {
        return c1.getName() == c2.getName();
    }
};
struct CustomerHash {
    std::size_t operator() (Customer const& c) const {
        return std::hash<std::string>()(c.getName());
    }
};
// define class that combines operator() for variadic base classes:
template<typename... Bases>
struct Overloader : Bases...
{
	using Bases::operator()...; // OK since C++17
};
int main()
{
    // combine hasher and equality for customers in one type:
    using CustomerOP = Overloader<CustomerHash,CustomerEq>;
    std::unordered_set<Customer,CustomerHash,CustomerEq> coll1;
    std::unordered_set<Customer,CustomerOP,CustomerOP> coll2;
    ...
}
```



# Chapter 5. Tricky Basics 

## Zero Initialization 

```C++
template<typename T>
class MyClass {
    private:
    	T x;
    public:
        MyClass() : x{} { // ensures that x is initialized even for built-in types
        }
}
```

## Templates for Raw Arrays and String Literals  

```C++
#include <iostream>
template<typename T>
struct MyClass; // primary template

template<typename T, std::size_t SZ>
struct MyClass<T[SZ]> // partial specialization for arrays of known bounds
{
	static void print() { std::cout << "print() for T[" << SZ << "]\n"; }
};
template<typename T, std::size_t SZ>
struct MyClass<T(&)[SZ]> // partial spec. for references to arrays of known bounds
{
	static void print() { std::cout << "print() for T(&)[" << SZ << "]\n"; }
};
template<typename T>
struct MyClass<T[]> // partial specialization for arrays of unknown bounds
{
	static void print() { std::cout << "print() for T[]\n"; }
};
template<typename T>
struct MyClass<T(&)[]> // partial spec. for references to arrays of unknown bounds
{
	static void print() { std::cout << "print() for T(&)[]\n"; }
};
template<typename T>
struct MyClass<T*> // partial specialization for pointers
{
	static void print() { std::cout << "print() for T*\n"; }
};

#include "arrays.hpp"
template<typename T1, typename T2, typename T3>
void foo(int a1[7], int a2[], // pointers by language rules
        int (&a3)[42], // reference to array of known bound
        int (&x0)[], // reference to array of unknown bound
        T1 x1, // passing by value decays
        T2& x2, T3&& x3) // passing by reference
{
    MyClass<decltype(a1)>::print(); // uses MyClass<T*>
    MyClass<decltype(a2)>::print(); // uses MyClass<T*>
    MyClass<decltype(a3)>::print(); // uses MyClass<T(&)[SZ]>
    MyClass<decltype(x0)>::print(); // uses MyClass<T(&)[]>
    MyClass<decltype(x1)>::print(); // uses MyClass<T*>
    MyClass<decltype(x2)>::print(); // uses MyClass<T(&)[]>
    MyClass<decltype(x3)>::print(); // uses MyClass<T(&)[]>
}
int main()
{
    int a[42];
    MyClass<decltype(a)>::print(); // uses MyClass<T[SZ]>
    extern int x[]; // forward declare array
    MyClass<decltype(x)>::print(); // uses MyClass<T[]>
    foo(a, a, a, x, x, x, x);
}
int x[] = {0, 8, 15}; // define forward-declared array
```

## `.template` Construct  

The `.template` notation (and similar notations such as `->template` and `::template`) should be
used **only inside templates** and only if they follow something that depends on a template parameter.  

```C++
template<unsigned long N>
void printBitset (std::bitset<N> const& bs) {
    std::cout << bs.template to_string<char, std::char_traits<char>,
   								std::allocator<char>>();
}
```



## Variable Templates  Since C++14

<font color=red>as for all templates, **variable template** declaration may not occur inside functions or block scope. </font>

```C++
template<typename T = long double>
constexpr T pi = T{3.1415926535897932385};

// To use a variable template, you have to specify its type.   
std::cout << pi<> << ’\n’; // outputs a long double
std::cout << pi<float> << ’\n’; // outputs a float
std::cout << pi << ’\n’; // ERROR
```

the initialized variables are in global scope  

```C++
// header.hpp:
template<typename T> T val{}; // zero initialized value
// translation unit 1:
#include "header.hpp"
int main()
{
    val<long> = 42;
    print();
}
// translation unit 2:
#include "header.hpp"
void print()
{
	std::cout << val<long> << ’\n’; // OK: prints 42
}
```

Type Traits Suffix _v

```C++
std::is_const_v<T> // since C++17 
std::is_const<T>::value // since C++11

namespace std {
	template<typename T> constexpr bool is_const_v = is_const<T>::value;
}
```

## Template Template Parameters  

template parameter itself can be a class template  

Since C++11, we can also substitute Cont with the name of an alias template, but it wasn’t until C++17 that permits the use of the keyword ***typename*** instead of ***class***

```C++
template<typename T,
	template<class> class Cont = std::deque>
class Stack { // OK
	...
};

template<typename T,
	template<typename> typename Cont = std::deque>
class Stack { // ERROR before C++17
    ...
};
```

### Template Template Argument Matching  

prior to C++17 a **template template argument** had to be a template with parameters that **exactly
match** the parameters of the template template parameter it substitutes, with exceptions of **variadic template template parameters**  

C++17 relaxed the matching rule to just require that the **template template parameter** be at least as specialized as the corresponding **template template argument** 

```C++
/* ERROR prior C++17
defaultvalue std::deque is not compatible with the template template parameter Cont.
*/
template<typename T,
	template<class> class Cont = std::deque>
class Stack {
	...
};

/* exactly match std::deque including the second default template arguments of template template arguments
*/
template<typename T,
	template<typename Elem,
    	typename Alloc = std::allocator<Elem>>
    	class Cont = std::deque>
class Stack {
...
};

template<typename T1, typename T2,
	template<typename... > class Cont> // Cont expects any number of
class Rel { // type parameters
	...
};

Rel<int, double, std::deque> rel; // OK: std::list has two template parameters
// but can be used with one argumen
```

# Chapter 6. Move Semantics and `enable_if<>`  

## Perfect Forwarding  

`T&&` for a template parameter T behaves different from X&& for a specific type X.  
`T&&` for a template parameter T declares a **forwarding reference** (also called **universal reference**)  

## Disabling Special Member Functions   

```C++
class C {
    public:
    template<typename T>
    C (T const&) {
    	std::cout << "tmpl copy constructor\n";
    }
    ...
};

C x;
C y{x}; // still uses the predefined copy constructor (not the member template)
```

a tricky solution is to declare below to prevent copy constructor from being implicitly declared  

```C++
C(C const volatile&) = delete;
```

## Disable Templates with enable_if<>  

`std::enable_if<>` is a **type trait** that evaluates a given **compile-time expression** passed as its (first) template argument and behaves as follows:

1. If the expression yields ***true***, its type member type yields second template argument type or ***void*** if no second template argument is passed.
2. If the expression yields ***false***, the member type is not defined. Due to a template feature called **SFINAE (substitution failure is not an error)**, this has the effect that the function template with the `enable_if` expression is ignored.  

```C++
#include <utility>
#include <string>
#include <iostream>
class Person
{
    private:
        std::string name;
    public:
        template<typename STR>
        explicit Person(STR&& n) : name(std::forward<STR>(n)) {}
        // copy and move constructor:
        Person (Person const& p) : name(p.name) {}
    	Person (Person&& p) : name(std::move(p.name)) {}
};

std::string s = "sname";
Person p1(s); // init with string obj
Person p3(p1); // ERROR 
// because template<typename STR> Person(STR&& n) is a better match than Person (Person const& p)

template<typename STR,
    typename = std::enable_if_t<
    	std::is_convertible_v<STR, std::string>>>
Person(STR&& n);

Person p3(p1); // OK 
```



# Chapter 7. By Value or by Reference  

## Passing by Value (Decay)  

Calling a copy constructor can become expensive. However, compilers might optimize away copy operations copying objects  

1. When directly calling the function template for **prvalues**, compilers usually optimize passing the argument so that no copying constructor is called at all. Since C++17, this optimization is required. Before C++17, a compiler must at least have to try to use **move semantics** if it doesn’t optimize the copying away
2. When directly calling the function template for **xvalues**, force to call the **move constructor**  

```C++
template<typename T>
void printV (T arg) {
...
}

std::string returnString();
std::string s = "hi";
printV(s); // copy constructor
printV(std::string("hi")); // copying usually optimized away (if not, move constructor)
printV(returnString()); // copying usually optimized away (if not, move constructor)
printV(std::move(s)); // move constructor
```

## Passing by Reference (never decay)

```C++
template<typename T>
void outR (T& arg) {
	...
}

std::string returnString();
std::string s = "hi";
outR(s); // OK: T deduced as std::string, arg is std::string&
outR(std::string("hi")); // ERROR: not allowed to pass a temporary (prvalue)
outR(returnString()); // ERROR: not allowed to pass a temporary (prvalue)
outR(std::move(s)); // ERROR: not allowed to pass an xvalue

const std:string returnConstString();
std::string const c = "hi";
outR(c); // OK: T deduced as std::string const
outR(returnConstString()); // OK: same if returnConstString() returns const string
outR(std::move(c)); // OK: T deduced as std::string const
outR("hi"); // OK: T deduced as char const[3]
```

## `std::ref()` and `std::cref()`

`std::ref()` and `std::cref()`wraps the passed argument s by an object that **acts like a reference**. In fact, it creates an object of type `std::reference_wrapper<>` referring to the original argument . The wrapper supports implicit type conversion back to the original type, yielding the original object.

`std::ref()` and `std::cref()` usually work fine only if you pass objects through generic code  

## Avoid Return Reference Type

```C++
template<typename T>
T retR(T&& p) // p is a forwarding reference
{
	return T{...}; // OOPS: returns by reference when called for lvalues
}

template<typename T>
T retV(T p) // Note: T might become a reference
{
	return T{...}; // OOPS: returns a reference if T is a reference
}
int x;
retV<int&>(x); // retT() instantiated for T as int&
```

Use the type trait `std::remove_reference<>` or declare the return type to be **auto** which always decays 



# Chapter 8. Compile-Time Programming

In fact, C++ has multiple features to support compile-time programming:

1. Since before C++98, templates have provided the ability to compute at compile time, including
   using loops and execution path selection, but requires nonintuitive syntax  
2. With **partial specialization** we can choose at compile time between different class template implementations depending on specific constraints or requirements.
3. With the **SFINAE principle**, we can allow selection between different function template implementations for different types or different constraints  
4. In C++11 and C++14, compile-time computing became increasingly better supported with the **constexpr feature** using “intuitive” execution path selection and, since C++14, most statement kinds (including for loops, switch statements, etc.).
5. C++17 introduced a **“compile-time if”** to discard statements depending on compile-time conditions or constraints. It works even outside of templates  

## before C++11

```C++
template<unsigned p, unsigned d> // p: number to check, d: current divisor
struct DoIsPrime {
	static constexpr bool value = (p%d != 0) && DoIsPrime<p,d-1>::value;
};
template<unsigned p> // end recursion if divisor is 2
struct DoIsPrime<p,2> {
	static constexpr bool value = (p%2 != 0);
};
template<unsigned p> // primary template
struct IsPrime {
	// start recursion with divisor from p/2:
	static constexpr bool value = DoIsPrime<p,p/2>::value;
};
// special cases (to avoid endless recursion with template instantiation):
template<>
struct IsPrime<0> { static constexpr bool value = false; };
template<>
struct IsPrime<1> { static constexpr bool value = false; };
template<>
struct IsPrime<2> { static constexpr bool value = true; };
template<>
struct IsPrime<3> { static constexpr bool value = true; };
```

## ***constexpr*** function  

can evalute at compile time, but not necessary evalute at compile

```C++
constexpr bool
doIsPrime (unsigned p, unsigned d) // p: number to check, d: current divisor
{
    return d!=2 ? (p%d!=0) && doIsPrime(p,d-1) // check this and smaller divisors
    : (p%2!=0); // end recursion if divisor is 2
}
constexpr bool isPrime (unsigned p)
{
    return p < 4 ? !(p<2) // handle special cases
    : doIsPrime(p,p/2); // start recursion with divisor from p/2
}
```

 C++11 introduced **constexpr function** with the limitation of **having only one statement** and C++14 removes it

```C++
constexpr bool isPrime (unsigned int p)
{
for (unsigned int d=2; d<=p/2; ++d) {
    if (p % d == 0) {
    	return false; // found divisor without remainder
    	}
    }
    return p > 1; // no divisor without remainder found
}
```

## Execution Path Selection  

### class templates

use **partial specialization** to select **class templates** at compile time between different implementation  

### function templates

Because function templates do not support **partial specialization**, use other mechanisms including

1. Use classes with **static** functions,
2. Use `std::enable_if` (use **partial specialization** and **SFINAE** feature)
3. Use the **SFINAE** feature
4. Use the **compile-time if** feature, available since C++17

#### SFINAE (pronounced like sfee-nay),  “substitution failure is not an error"

In substitution of the constructs appearing directly in the declaration of the function (but not its body), which is distinct from instantiation process, ignore template substitution that produces constructs that make no sense  

```C++
template<typename T>
typename T::size_type len (T const& t)
{
	return t.size();
}
std::allocator<int> x;
std::cout << len(x) << ’\n’; // ERROR: len() selected, but x has no size()

template<typename T>
auto len (T const& t) -> decltype( (void)(t.size()), T::size_type() )
{
	return t.size();
}
/*
decltype( (void)(t.size()), T::size_type() )
expression t.size() must be valid, otherwise SFINAE
The cast of the expression to void is to avoid the possibility of a user-defined comma operator overloaded for the type of the expressions
*/
```

#### **compile-time if**

C++17 additionally introduces a **compile-time if** statement that allows is to enable or disable
specific statements based on compile-time conditions, which can be used in any function, not only in templates.   

```C++
template<typename T, typename... Types>
void print (T const& firstArg, Types const&... args)
{
    std::cout << firstArg << ’\n’;
    if constexpr(sizeof...(args) > 0) {
    	print(args...); // code only available if sizeof...(args)>0 (since C++17)
	}
}
```



# Chapter 9. Using Templates in Practice  

**inclusion model**: include the definitions of a template in the header file.

Only **full specializations of function templates** need ***inline*** when defined in header files outside
classes or structures.

To take advantage of **precompiled headers** ***(PCH)*** (often for widely used and stable headers), be sure to keep the same order for #include directives    

# Chapter 10. Basic Template Terminology 

`template-id`: the combination of the template name, followed by the arguments in angle brackets

 ## “Class Template” or “Template Class”?  

**“class type”** includes unions, but **“class”** does not.  

`class template` states that the class is a template, a parameterized description of a family of classes.  

`template class` has been used as a synonym for `class template` or to refer to classes generated from templates or to refer to classes with a name that is a template-id.  

## Substitution, Instantiation, and Specialization  

`template instantiation`: substitute concrete arguments for the template parameters.
`specialization` : entity resulting from an instantiation or an incomplete instantiation.
`explicit specialization` (as opposed to an `instantiated or generated specialization`).
When talking about (explicit or partial) specializations, the general template is also called the `primary template`.    

## Incomplete Types  

1. A class type that has been declared but not yet defined.
2. <font color=red>An array type with an **unspecified bound**.**</font>
3. An array type with an incomplete element type.
4. ***void***
5. An **enumeration type** as long as the underlying type or the enumeration values are not defined.
6. Any type above to which ***const*** and/or ***volatile*** are applied  

## Template Arguments versus Template Parameters  

<font color=red>parameters are initialized by arguments</font>

# Chapter 11. Generic Libraries  

`callable type` is either a function object type or a pointer to member  

`function object type` includes:

1. Pointer-to-function types
2. Class types with an overloaded `operator()` (sometimes called **functors**), including lambdas
3. Class types with a **conversion function** yielding a pointer-to-function or reference-to-function  

## `std::invoke()`

A common application of `std::invoke()` is to wrap single function calls  

```C++
#include <utility> // for std::invoke()
#include <functional> // for std::forward()
#include <type_traits> // for std::is_same<> and invoke_result<>
template<typename Callable, typename... Args>
decltype(auto) call(Callable&& op, Args&&... args)
{
    /*
    If the callable has return type void, the initialization of ret as 	       		decltype(auto) is not allowed, because void is an incomplete type
    */
    if constexpr(std::is_same_v<std::invoke_result_t<Callable, Args...>,
    	void>) {
            // return type is void:
            std::invoke(std::forward<Callable>(op),
            std::forward<Args>(args)...);
            ...
            return;
        }
    else {
            // return type is not void:
            decltype(auto) ret{std::invoke(std::forward<Callable>(op),
            std::forward<Args>(args)...)};
            ...
            return ret;
        }
}
```

## `std::addressof()`

`std::addressof<>()` **function template** yields the actual address of an object or function. It works even if the object type has an **overloaded operator &**  

## `std::declval()`

<font color=red>The function doesn’t have a definition and therefore cannot be called</font> (and doesn’t create an object). Hence, it can only be used in unevaluated operands (such as those of **decltype** and **sizeof** constructs)  

```C++
#include <utility>
template<typename T1, typename T2,
		// avoid to call a (default) constructor for T1 and T2 
		typename RT = std::decay_t<decltype(true ? std::declval<T1>()
											: std::declval<T2>())>> 
RT max (T1 a, T2 b)
{
	return b < a ? a : b;
}
```

## Perfect Forwarding Temporaries  

```C++
template<typename T>
void f (T&& t) // t is forwarding reference
{
	g(std::forward<T>(t)); // perfectly forward passed argument t to g()
}

/* if the data perfectly forwarded in generic code is not a parameter */
template<typename T>
void foo(T x)
{
    auto&& val = get(x);
    ...
    // perfectly forward the return value of get() to set():
    set(std::forward<decltype(val)>(val));
}
```

## References as Template Parameters  

using reference types for nontype template parameters is tricky and can be dangerous.   

To disable references, a simple **static assertion** is enough:  	

```C++
template<typename T>
class optional
{
    static_assert(!std::is_reference<T>::value,
    "Invalid instantiation of optional<T> for references");
    ...
};
```

# Chapter 12. Fundamentals in Depth 

## Parameterized Declarations 

C++ currently supports four fundamental kinds of templates, which <font color=red>can appear in namespace scope or class scope but not in function scope or local class scope</font>

1. class templates/nested class templates
2. function templates/member function templates,
3. variable templates/static data member templates
4. alias template/member alias templates 

```C++
template<typename T>            // a namespace scope class template
class Data {
  public:
    static constexpr bool copyable = true; // not a variable template, called temploid
    //...
};

template<typename T>            // a namespace scope function template
void log (T x) {
    //...
}

template<typename T>            // a namespace scope variable template (since C++14)
T zero = 0;

template<typename T>            // a namespace scope variable template (since C++14)
bool dataCopyable = Data<T>::copyable;

template<typename T>            // a namespace scope alias template
using DataList = Data<T*>;

class Collection {
  public:
    template<typename T>        // an in-class member class template definition
    class Node {
        //...
    };

    template<typename T>        // an in-class (and therefore implicitly inline)
    T* alloc() {                // member function template definition
        //...
    }

    template<typename T>        // a member variable template (since C++14)
     static T zero = 0;

    template<typename T>        // a member alias template
     using NodePtr = Node<T>*;
};

```

C++17 introduced **deduction guides** whose syntax was chosen to be reminiscent of function templates, but not templates (e.g., they are not instantiated)

<font color=red>**constructor template** disables the implicit declaration of the **default constructor** (because it is only implicitly declared if no other constructor is declared) </font>

### Union Templates 

<font color=red>**Union templates** are considered a kind of **class template**</font>

```C++
template<typename T>
union AllocChunk {
    T object;
    unsigned char bytes[sizeof(T)];
};
```

### Function templates can have Default Call Arguments 

### No Virtual Member Function Templates 

the usual implementation of the virtual function call mechanism uses a fixed-size table with one entry per virtual function. Supporting virtual member function templates would require support for a whole new kind of mechanism in C++ compilers and linkers 

### Linkage of Templates 

<font color=red>unlike class types, class templates cannot share a name with a different kind of entity:</font>

```C++
int C;
class C; // OK: class names and nonclass names are in a different “space”

int X;
template<typename T>
class X; // ERROR: conflict with variable X

struct S;
template<typename T>
class S; // ERROR: conflict with struct S
```

<font color=red>Template names cannot have C linkage.</font> Nonstandard linkages (such as Jave linkage) may have an implementation-dependent meaning 

Templates usually have **external linkage** except that 

1. namespace scope function templates with the **static** specifier, 
2. templates that are direct or indirect members of an **unnamed namespace** (which have internal linkage)
3. member templates of **unnamed classes** (which have no linkage).  

<font color=red>The linkage of an instance of a template is that of the template. </font>

### Primary Templates 

**Nonprimary templates** occur when declaring partial specializations of **class templates** or **variable templates** 

## Template Parameters 

three basic kinds of template parameters, any of which can be used in template parameter pack  

1. Type parameters <font color=red>act much like a type alias</font>
2. Nontype parameters
3. Template template parameters

a template parameter name can be referred to in a subsequent parameter declaration (but not before):

```C++
template<typename T, T Root>
class Structure;
```

### Nontype Parameters  

the type must be one of the following:  

1. An integer type or an enumeration type
2. A pointer type <font color=red>(function and array types can implicitly decay to pointer type)</font>
3. A pointer-to-member type
4. An **lvalue reference type** to objects/functions <font color=red>rvalue references are not permitted  </font>
5. `std::nullptr_t`
6. A type containing **auto** or **decltype(auto)** (since C++17 only)

declaration of a nontype template parameter can also start with the keyword **typename**/**class**

```C++
template<typename T, // a type parameter
		typename T::Allocator* Allocator> // a nontype parameter
class List;

template<class X*> // a nontype parameter of pointer type
class Y;
```

<font color=red>**nonreference nontype parameters** are always **prvalues**, so their address cannot be taken, and they cannot be assigned to.</font>

### Template Template Parameters  

Template template parameters are placeholders for **class or alias templates**. They <font color=red>cannot be declared with the keywords ***struct*** and ***union***</font>

```C++
template<template<typename X> class C> // OK
void f(C<int>* p);

template<template<typename X> typename C> // OK since C++17
void f(C<int>* p);

template<template<typename X> struct C> // ERROR: struct not valid here
void f(C<int>* p);
template<template<typename X> union C> // ERROR: union not valid here
void f(C<int>* p);
```

<font color=red>the names of the template parameter of the template template parameter can be used only in the declaration of other parameters of that template template parameter. </font>

```C++
template<template<typename T, T*> class Buf> // OK
class Lexer {
    static T* storage; // ERROR: a template template parameter cannot be used here
    ...
};
```

### Template Parameter Packs
Since C++11, any kind of template parameter can be turned into a **template parameter pack** by
introducing an `ellipsis (...)`  prior to the template parameter name or, if the template parameter is unnamed, where the template parameter name would occur:  

```C++
template<typename... Types> // declares a template parameter pack named Types
class Tuple;

template<typename... >
class Tuple;

template<typename T, unsigned... Dimensions>
class MultiArray; // OK: declares a nontype template parameter pack

template<typename T, template<typename,typename>... Containers>
void testContainers(); // OK: declares a template template parameter pack
```

<font color=red>**Primary class templates, variable templates, and alias templates** may have **at most one template parameter pack**, which must be the last template parameter. While **partial specializations of class and variable templates can have multiple parameter pack ** </font>

<font color=red>**Function templates** can have multiple template parameter packs, as long as each template parameter subsequent to a template parameter pack either has a default value or can be deduced  </font>

```C++
template<typename... Types, typename Last>
class LastType; // ERROR: template parameter pack is not the last template 	 parameter
template<typename... TestTypes, typename T>
void runTests(T value); // OK: template parameter pack is followed
						// by a deducible template parameter

template<unsigned...> struct Tensor;
template<unsigned... Dims1, unsigned... Dims2>
	auto compose(Tensor<Dims1...>, Tensor<Dims2...>); // OK: the tensor dimensions can be deduced

template<typename...> Typelist;
template<typename X, typename Y> struct Zip;
template<typename... Xs, typename... Ys>
	struct Zip<Typelist<Xs...>, Typelist<Ys...>>;
	// OK: partial specialization uses deduction to determine
	// the Xs and Ys substitutions
```

<font color=red>a type parameter pack cannot be expanded in its own parameter clause but can be expanded in nested templates </font>

```C++
template<typename... Ts, Ts... vals> struct StaticValues {};
// ERROR: Ts cannot be expanded in its own parameter list

template<typename... Ts> struct ArgList {
	template<Ts... vals> struct Vals {};
};
```

### Default Template Arguments  

<font color=red>A number of contexts do not permit **default template arguments** </font>

1. Partial specializations  
2. Parameter packs
3. The out-of-class definition of a member of a class template  
4. A friend class template declaration  
5. A friend function template declaration unless it is a definition and no declaration of it appears
   anywhere else in the translation unit

```C++
template<typename T>
class C;
template<typename T = int>
class C<T*>; // ERROR

template<typename... Ts = int> struct X; // ERROR

template<typename T> struct X
{
	T f();
};
template<typename T = int> T X<T>::f() {} // ERROR

struct S {
	template<typename = void> friend struct F; // ERROR
};

struct S {
    template<typename = void> friend void f(); // ERROR: not a definition
    template<typename = void> friend void g() {} // OK so far
};
template<typename> void g(); // ERROR: g() was given a default template argument
							// when defined; no other declaration may exist here
```

A template parameter for a **class template, variable template, or alias template** can have a default only if default arguments were also supplied for the subsequent parameters, The subsequent default values could have been declared in a previous declaration of that template.  

default template arguments for template parameters of **function templates** do not require subsequent template parameters to have a default template argument  

```C++
template<typename T1, typename T2, typename T3,
	typename T4 = char, typename T5 = char>
class Quintuple; // OK

template<typename T1, typename T2, typename T3 = char,
	typename T4, typename T5>
class Quintuple; // OK: T4 and T5 already have defaults

template<typename T1 = char, typename T2, typename T3,
	typename T4, typename T5>
class Quintuple; // ERROR: T1 cannot have a default argument
// because T2 doesn’t have a default

template<typename R = void, typename T>
R* addressof(T& value); // OK: if not explicitly specified, R will be void
```

Default template arguments cannot be repeated:  

```C++
template<typename T = void>
class Value;

template<typename T = void>
class Value; // ERROR: repeated default argument
```

## Template Arguments  

When instantiating, the template arguments can be determined by

1. Explicit template arguments. <font color=red>A template name followed by explicit template arguments enclosed in angle brackets is called a ***template-id***. </font>
2. Injected class name: within the scope of a class template X, the name of class template (X) can be equivalent to the **template-id**
3. Default template arguments
4. Argument deduction  

<font color=red>If all the template arguments can be deduced, no angle brackets is needed after template name while the angle brackets must be provided even if all template parameters have a default value</font>

<font color=red>Nontype template arguments can be **implicitly converted without narrowing**. But prior to C++17, when matching an argument to nontype template parameter that is a pointer or reference, **user-defined conversions and derived-to-base conversions are not considered**  </font>

```C++
template<typename T, T nontypeParam>
class C;
    struct Base {
    int i;
} base;

struct Derived : public Base {
} derived;

C<Base*, &derived>* err1; // ERROR: derived-to-base conversions are not considered
C<int&, base.i>* err2; // ERROR: fields of variables aren’t considered to be variables

int a[10];
C<int*, &a[0]>* err3; // ERROR: addresses of array elements aren’t acceptable either
```

[<font color=red>Linkage restrictions</font>](#Nontype_Template_Parameters_Linkage)

## Equivalence  

1. For type arguments, type aliases don’t matter: 
2. For integer nontype arguments, the value of the argument is compared    

```C++
template<typename T, int I>
class Mix;

using Int = int;
Mix<int, 3*3>* p1;
Mix<Int, 4+5>* p2; // p2 has the same type as p1
```

It is an error for templates to be declared in ways that differ only from functionally equivalent expressions that are not actually <font color=red>but need not be diagnosed by your compiler</font>

```C++
template<int N> struct I {};
template<int M, int N> void f(I<M+N>); // #1
template<int N, int M> void f(I<N+M>); // #2, equivalent to #1
template<int M, int N> void f(I<N+M>); // #3 ERROR
```

<font color=red>A function generated from a function template is **never equivalent** to an ordinary function even
though they may have the same type and the same name.   </font> So

1. A function generated from a **member function template** never overrides a **virtual function**.

2. A constructor generated from a **constructor template** is never a **copy or move constructor**  

   However, a **constructor template** can be a **default constructor**.  

1. An assignment generated from an **assignment template** is never a **copy-assignment or move-assignment operator** 

## Variadic Templates  

### Pack Expansions

A **pack expansion** is a construct that expands an argument pack into separate arguments according to ***pattern***

**pack expansions** can be used essentially <font color=red>anywhere in the language where the grammar provides a **comma-separated list** (some of these are included merely for the sake of completeness and useless)</font>, including:  

1. In the list of base classes.
2. In the list of base class initializers in a constructor.
3. In a list of call arguments (the pattern is the argument expression).
4. In a list of initializers (e.g., in a braced initializer list).
5. In the template parameter list of a class, function, or alias template.
6. In the list of exceptions that can be thrown by a function (deprecated in C++11 and C++14, and
   disallowed in C++17).
7. Within an attribute, if the attribute itself supports pack expansions (although no such attribute is defined by the C++ standard).  
8. When specifying the alignment of a declaration.
9. When specifying the capture list of a lambda.
10. In the parameter list of a function type.
11. In using declarations (since C++17).  

```C++
template<typename... Bases>
struct Overloader : Bases...
{
	using Bases::operator()...; // OK since C++17, expansion in using declarations
};
```

```C++
template<typename... Mixins>
class Point : public Mixins... { // base class pack expansion
	double x, y, z;
public:
	Point() : Mixins()... { } // base class initializer pack expansion
	template<typename Visitor>
	void visitMixins(Visitor visitor) {
		visitor(static_cast<Mixins&>(*this)...); // call argument pack expansion
	}
};

struct Color { char red, green, blue; };
struct Label { std::string name; };
Point<Color, Label> p; // inherits from both Color and Label
```

```C++
template<typename... Ts>
struct Values {
    template<Ts... Vs>
    struct Holder {
    };
};

int i;
Values<char, int, int*>::Holder<’a’, 17, &i> valueHolder;
```

### Function Parameter Packs  

A **function parameter pack** is a function parameter that matches zero or more function call arguments  

**Template parameter packs** and **function parameter packs** are referred to as **parameter packs**  

```C++
template<typename... Mixins>
class Point : public Mixins...
{
	double x, y, z;
public:
// default constructor, visitor function, etc. elided
	Point(Mixins... mixins) // mixins is a function parameter pack
		: Mixins(mixins)... { } // initialize each base with the supplied mixin value
};
```

ambiguity between an unnamed function parameter pack and a C-style “vararg” parameter.  

```c++
template<typename T> void c_style(int, T...); // an unnamed parameter of type T followed by a C-style vararg parameter.
template<typename... T> void pack(int, T...);
```

### Multiple and Nested Pack Expansions  

When a **pack expansion** containing **multiple parameter packs**, all of the parameter packs must have the same length.   

```C++
template<typename F, typename... Types>
void forwardCopy(F f, Types const&... values) {
	f(Types(values)...);
}
// three-argument forwardCopy would look like
template<typename F, typename T1, typename T2, typename T3>
void forwardCopy(F f, T1 const& v1, T2 const& v2, T3 const& v3) {
	f(T1(v1), T2(v2), T3(v3));
}
```

When pack expansions are nested, each occurrence of a parameter pack is “expanded” by its **nearest enclosing pack expansion (and only that pack expansion)**.  

```C++
template<typename... OuterTypes>
class Nested {
    template<typename... InnerTypes>
	void f(InnerTypes const&... innerValues) {
		g(OuterTypes(InnerTypes(innerValues)...)...);
	}
};
// such as 
template<typename O1, typename O2>
class Nested {
    template<typename I1, typename I2, typename I3>
    void f(I1 const& iv1, I2 const& iv2, I3 const& iv3) {
        g(O1(I1(iv1), I2(iv2), I3(iv3)),
            O2(I1(iv1), I2(iv2), I3(iv3)),
            O3(I1(iv1), I2(iv2), I3(iv3)));
    }
};
```

### Zero-Length Pack Expansions  

the substitution of an argument pack of any size does not affect how the **pack expansion** (or its **enclosing variadic template**) is parsed, so <font color=red>the syntactic interpretation often fails in the presence of **zero-length argument packs** which behaves as if not (semantically) as if not present. </font>

### Fold Expressions  

***fold expressions*** applies to **all binary operators** except `.`, `->`, and `[]`.  

```C++
(pack op ... op value) // binary right fold
(value op ... op pack) // binary left fold

(pack op ... ) // unary right fold
(... op pack) // unary left fold
```

<font color=red>the parentheses are required in ***fold expressions*** </font>

an empty expansion of a **unary fold** is generally an error, with **three exceptions**:
1. An empty expansion of a unary fold of && produces the value ***true***,.
2. An empty expansion of a unary fold of || produces the value ***false***.
3. An empty expansion of a unary fold of the comma operator (,) produces a **void expression**.  

<font color=red>recommend using **binary fold expressions** instead of **unary fold expressions** because it create surprises for the **empty expansion** if you overload one of these special operators  </font>

## Template as Friends  

C++11 also added syntax to make a template parameter a friend:  

```C++
template<typename T>
class Wrap {
    friend T; // ignored if T is not actually a class type
    ...
};
```

<font color=red>Friend class declarations cannot be definitions </font>
<font color=red>Friend function declaration can be a definition and always ***inline*** only if it names an unqualified function name that is not followed by angle brackets. </font>

```C++
template<typename T>
class Tree {
    // OK even if first declaration of class/function
    friend class Factory; 
    friend void func_friend();
    template<typename> friend class Stack;
    template<typename T1, typename T2>
        friend void temp_func_friend(T1, T2);
    //friend class Node<T>; // error if Node isn’t visible
    //friend void combine<int, int>(int, int); // error if template combine isn’t visible
};
```

<font color=red>A **friend template (all instances of a template are friends)** can declare only primary templates and members of primary templates. Any partial specializations and explicit specializations associated with a primary template are **automatically considered friends** too </font>

# Chapter 13. Names in Templates  

C++ (like C) is a **context-sensitive language**: that is, a construct cannot always be understood without knowing its wider context.  

## Name Taxonomy  

two major naming concepts:

1. A name is a **qualified name** if the scope to which it belongs is explicitly denoted using a scope-resolution operator (`::`) or a member access operator (`.` or `->`).

2. A name is a **dependent name** if it depends in some way on a **template parameter**  

| Classification         | Explanation and Notes                                        |
| ---------------------- | ------------------------------------------------------------ |
| Identifier             |                                                              |
| Operator-function-id   | The keyword ***operator*** followed by the symbol for an operator |
| Conversion-function-id | Used to denote a user-defined implicit conversion operator   |
| Literal-operator-id    | Used to denote a user-defined literal operator               |
| Template-id            | The name of a template followed by template arguments enclosed in angle brackets |
| Unqualified-id         | The generalization of an identifier                          |
| Qualified-id           | qualified with the name of a class, enum, or namespace, or just with the global scope resolution operator. |
| Qualified name         | names that undergo qualified lookup                          |
| Unqualified name       | names undergo what the standard calls unqualified lookup.    |
| Name                   | Either a qualified or an unqualified name                    |
| Dependent name         | name that depends in some way on a template parameter        |
| Non-dependent name     | name that is not a dependent name                            |

## Looking Up Names  

1. **Qualified names** are looked up in the scope implied by the qualifying construct. If that scope is a class, then base classes may also be searched  
2. **Unqualified names**: **ordinary lookup** in addition to sometimes **argument-dependent lookup (ADL)  **

### ordinary lookup

1. look up in successively more enclosing scopes
2. in member function definitions, the scope of the class and its **base classes** is searched before any other enclosing scopes

### Argument-Dependent Lookup  

Argument-dependent lookup (ADL) is the set of rules for looking up the **unqualified function names in function-call expressions**

**ADL** does not happen if ordinary lookup finds

1. the name of a member function,
2. the name of a variable,
3. the name of a type, 
4. the name of a block-scope function declaration.
5. <font color=red>ADL is also inhibited if the name of the function to be called is enclosed in parentheses (require ***unqualified-id***) </font>

ADL looks up the name in **associated namespaces and associated classes** of the types of the call arguments, as if the name had been qualified with each of these namespaces in turn, <font color=red>except that **using directives** are ignored  </font>

the definition of the set of **associated namespaces and associated classes** for a given type is 

1. For **built-in types**, this is the empty set.
2. For **pointer and array types**, the set of **associated namespaces and classes** is that of the underlying type.
3. For **enumeration types**, the **associated namespace** is the namespace in which the enumeration is declared.
4. For **class members**, the **enclosing class** is the associated class.  
5. For **class types (including union types)**, the set of **associated classes** is the type itself, the enclosing class, and any direct and indirect base classes. The set of **associated namespaces** is the namespaces in which the **associated classes** are declared. If the class is a class template instance, then the types of the template type arguments and the classes and namespaces in which the template template arguments are declared are also included.
6. For **function types**, the sets of **associated namespaces and classes** comprise the namespaces and classes associated with all the parameter types and those associated with the return type.
7. For pointer-to-member-of-class-X types, the sets of **associated namespaces and classes** include those associated with X in addition to those associated with the type of the member. (If it is a pointer-to-member-function type, then the parameter and return types can contribute too.)  

#### Argument-Dependent Lookup of Friend Declarations  

The ability of argument-dependent lookup to find friend declarations and definition is sometimes referred to as **friend name injection**, a misleading name, because it is the name of a pre-standard C++ feature that did in fact “inject” the names of friend declarations into the enclosing scope, making them visible to normal name lookup  

```C++
template<typename T>
class C1 {
    friend void f();
    friend void f(C1<T> const&);
};
void g (C1<int>* p)
{
    f(); // Error. f() is not visible
    f(*p); // OK. f(C<int> const&) is visible by ADL
}
```

### Injected Class Names  

<font color=red>The name of a class is injected inside the scope of that class itself and is therefore accessible as an unqualified name in that scope. (However, it is not accessible as a **qualified name** because this is the notation used to denote the constructors </font>

```C++
template<template<typename> class TT> class X {
};
template<typename T> class C {
    C* a; // OK: same as “C<T>* a;”
    C<void>& b; // OK
    X<C> c; // OK: C without a template argument list denotes the template C
    X<::C> d; // OK: ::C is not the injected class name and therefore always
    // denotes the template
};

template<int I, typename... T> class V {
    V* a; // OK: same as “V<I, T...>* a;”
    V<0, void> b; // OK
};
```

Within a class template, the injected class name or any type alias declarations of any enclosing class or class template is said to refer to a **current instantiation**. 
Types that depend on a template parameter but do not refer to a **current instantiation** are said to refer to an **unknown specialization**

```C++
template<typename T> class Node {
    using Type = T;
    Node* next; // Node refers to a current instantiation
    Node<Type>* previous; // Node<Type> refers to a current instantiation
    Node<T*>* parent; // Node<T*> refers to an unknown specialization
};

template<typename T> class C {
    using Type = T;
    struct I {
        C* c; // C refers to a current instantiation
        C<Type>* c2; // C<Type> refers to a current instantiation
        I* i; // I refers to a current instantiation
    };
    struct J {
        I* i; // I refers to an unknown specialization,
        		// because I does not enclose J
        J* j; // J refers to a current instantiation
    };
};
```

## Parsing Templates  

Two fundamental activities of compilers for most programming languages are tokenization—also
called scanning or lexing—and parsing  

### Context Sensitivity in Nontemplates 

tokenizing is easier than parsing. Fortunately, parsing is a subject for which a solid theory has been developed, and many useful languages are not hard to parse using this theory. However, the theory works best for **context-free languages**

To handle **context sensitivity**, a C++ compiler will couple a **symbol table** to the tokenizer and parser: 

1. When a declaration is parsed, it is entered in the symbol table. 
2. When the tokenizer finds an identifier, it looks it up and annotates the resulting token if it finds a type. 

```C++
x* // parse as declaration if x is a type otherwise parse as multiplication
X<1>(0) //  treat the < as an angle bracket only if the name is known to be that of a template; otherwise, the < is treated as an ordinary less-than operator
```

problem with ***tokenizer***

```C++
// a right-shift token >> are never treated as two separate tokens by the tokenizer, as a consequence of the maximum munch tokenization principle
List<List<int>> a; // Error before C++11 (List<List<int) >> a

// digraph <: as an alternative for the source character [
template<typename T> struct G {};
struct S;
G<::S> gs; // valid since C++11, but an error before that
// C++11 not treat characters <: as a digraph token equivalent to [ unless the characters <:: is immediately followed by : or >
#define F(X) X ## :
int a[] = { 1, 2, 3 }, i = 1;
int n = a F(<::)i]; // valid in C++98/C++03, but not in C++11
					// int n = a [ :: i]; in C++98/C++03
					// int n = a < :: : i]; in C++11
```



```C++
Trap<T>::x * y; // #2 declaration or multiplication?
typename Trap<T>::x * y; 

p.Deep<N>::f(); // parsed as ((p.Deep)<N)>f()
p.template Deep<N>::f();

namespace N {
    class X {
        ...
    };
    template<int I> void select(X*);
}
void g (N::X* xp)
{
	select<3>(xp); // ERROR: no ADL!  
    			// parsed as (select<3)>(xp) because compiler cannot decide that xp is a function call argument until it has decided that <3> is a template argument list, and cannot decide that <3> is a template argument list until it has found select() to be a template
}
```

### Dependent Names and Expressions 

<a id="Dependent_Names"></a>

Inside the definition of a template the meaning of some constructs may differ from one instantiation to another. In particular, **types and expressions** may depend on **types of type template parameters and values of non-type template parameters**.

dependent expression can be **type-dependent** or **value-dependent**

 ## Inheritance and Class Templates  

When an **unqualified name** is looked up in the templated derivation, the **non-dependent bases** are considered before the **list of template parameters**

```c++
template<typename X>
class Base {
    public:
        int basefield;
        using T = int;
}

class D1: public Base<Base<void>> { // not a template case really
    public:
        void f() { basefield = 3; } // usual access to inherited member
};
template<typename T>
class D2 : public Base<double> { // nondependent base
    public:
        void f() { basefield = 7; } // usual access to inherited member
        T strange; // T is Base<double>::T, not the template parameter!
};
```

standard C++ says that **non-dependent names** are looked up as soon as they are encountered but not looked up in **dependent base classes** to avoid **specialized base classes**

```C++
template<typename T>
class DD : public Base<T> { // dependent base
    public:
    	void f() { basefield = 0; }
};

template<> // explicit specialization
class Base<bool> {
    public:
    	enum { basefield = 42 };
};
void g (DD<bool>& d)
{
	d.f(); // DD<bool>::f() tries to modify constant basefield
}
```

To correct the code, it suffices to make the name **basefield dependent** because <font color=red>dependent names can be looked up only at the time of instantiation</font>, but it inhibits the **virtual call mechanics**

```C++
// Variation 1:
template<typename T>
class DD1 : public Base<T> {
    public:
    	void f() { this->basefield = 0; } // lookup delayed at the time of instantiatio
};

// Variation 2:
template<typename T>
class DD2 : public Base<T> {
    public:
    	void f() { Base<T>::basefield = 0; }
};
```

# Chapter 14. Instantiation  

## On-Demand Instantiation  

This **on-demand instantiation** feature, also called **implicit or automatic instantiation**, sets C++ templates apart from similar facilities in other early compiled languages (like Ada or Eiffel; some of these languages require explicit instantiation directives, whereas others use run-time dispatch mechanisms to avoid the instantiation process altogether).

The need to access a member of a class template is not always very explicitly visible.

In the following code, the compiler is allowed (but not required) to perform instantiation of `C<double>` if it can resolve the call without it   

```C++
template<typename T>
class C {
    public:
    	C(int); // a constructor that can be called with a single parameter
}; // may be used for implicit conversions
void candidate(C<double>); // #1
void candidate(int) { } // #2
int main()
{
	candidate(42);
}
```

  ## Lazy Instantiation  

A compiler should be “lazy” when instantiating templates, i.e. only as much as is really needed.

### Partial and Full Instantiation  

<font color=red>For **variable/class/function template**, if no need, compiler should not perform a complete instantiation and only permitted to substitute the declaration. This is called ***partial instantiation***.  </font>

**alias templates** do not have **partial instantiation**

```C++
template<typename T> T f (T p) { return 2*p; }
decltype(f(2)) x = 2; // body of f() not instantiated
    
template<typename T> class Q {
	using Type = typename T::Type;//  int::Type doesn’t make sense
};
Q<int>* p = 0; // OK: the body of Q<int> is not substituted

template<typename T> T v = T::default_value();
decltype(v<int>) s; // OK: initializer of v<int> not instantiated
```

### Instantiated Components 

When a **class template** is implicitly (fully) instantiated, <font color=red>each declaration of its members is instantiated as well, but the corresponding definitions are not unless used, with some exceptions:</font>

1. if the class template contains an **anonymous union**, the members of that union’s definition are also instantiated 
2. The definitions of **virtual member functions** may or may not be instantiated as a result of instantiating a class template. Many implementations will do

In addition to **non-data members of class templates**, **default function call arguments**, **exception specifications** and **default member initializers** are not instantiated unless they are needed 

Because only declaration is instantiated, <font color=red>the error detected in initialization process will not be reported, but the error in declaration will be detected as normal, except that `operator->` which requires to return a pointer type or another class type to which `operator->` applies, will check the requirement of declaration only if that operator is actually selected by overload resolution. </font>

```C++
template<typename T>
class Safe {
};
template<int N>
class Danger {
	int arr[N]; // OK here, although would fail for N<=0
};
template<typename T, int N>
class Tricky {
    public:
        void noBodyHere(Safe<T> = 3); // OK until usage of default value results in an error
        void inclass() {
        	Danger<N> noBoomYet; // OK until inclass() is used with N<=0
        }
        struct Nested {
        	Danger<N> pfew; // OK until Nested is used with N<=0
        };
        union { // due anonymous union:
        	Danger<N> anonymous; // OK until Tricky is instantiated with N<=0
        	int align;
        };
        void unsafe(T (*p)[N]); // OK until Tricky is instantiated with N<=0
        void error() {
        	Danger<-1> boom; // always ERROR (which not all compilers detect)
        }
};

template<typename T>
class C {
public:
	T operator-> (); // declaration here triggers no error, even for C<omt>
};

class B {
  int operator->() {} // triggers error only when selected by overload resolution
};
```

## The C++ Instantiation Model 

### Two-Phase Lookup 

two-phase lookup: first phase is the parsing of a template, and the second phase is instantiation 

how to determine [dependent names](#Dependent_Names)

| point of parsing                                            | point of instantiation (POI)<br/>with the template parameters replaced with the template arguments for that specific instantiation |
| ----------------------------------------------------------- | ------------------------------------------------------------ |
| non-dependent names ordinary lookup and, if applicable, ADL | dependent qualified names are looked up                      |
| unqualified dependent names ordinary lookup                 | unqualified dependent names perform ADL                      |

<font color=red>the second lookup performed at the POI is **only an ADL**</font>

```C++
template<typename T>
void f1(T x)
{
	g1(x); // #1
}
void g1(int)
{ }
int main()
{
	f1(7); // ERROR: g1 not found! no ADL triggered for built-in type int
}
// #2 POI for f1<int>(int)

class MyInt {
	public:
	MyInt(int i);
};
MyInt operator- (MyInt const&);
bool operator> (MyInt const&, MyInt const&);
using Int = MyInt;

template<typename T>
void f(T i)
{
    if (i>0) {
    	g(-i);
    }
}

void g(Int)
{
	f<Int>(42); // ADL can find g
}
// POI for f<Int>(Int)
```

### POI

C++ defines the **POI** for a reference to a **function template specialization** to be **immediately after** the nearest namespace scope declaration or definition that contains that reference. 

**POI** for **variable templates** is handled similarly to that of **function templates** 

the **POI** for a reference to a **generated class instance** is defined to be the point **immediately before** the nearest namespace scope declaration or definition that contains the reference to that instance.  

<font color=red>A **translation unit** often contains multiple POIs for the **same instance**. </font>
<font color=red>For **class template** instances, **only the first POI in each translation unit** is retained, and the subsequent ones are ignored </font>
<font color=red>For instances of **function and variable templates**, **all POIs are retained**.</font>

The ODR requires that the instantiations occurring at any of the retained POIs be equivalent, but a C++ compiler does not need to verify and diagnose violations of this rule. This allows a C++ compiler to pick just one nonclass POI to perform the actual instantiation without worrying that another POI might result in a different instantiation.

In practice, most compilers delay the actual instantiation of most **function templates** to the end
of the **translation unit**. Some instantiations cannot be delayed, including 

1. cases where instantiation is needed to determine a **deduced return type** 
2. and cases where the function is ***constexpr*** and must be evaluated to produce a constant
   result. 
3. Some compilers instantiate **inline functions** when they’re first used to potentially ***inline*** the
   call right away 

## Implementation Schemes For The Inclusion Model 

The source model for template definitions which are simply added to header files that are #included into the translation unit is called the **inclusion model** 

One challenge for an implementation with the **automatic scheme** is to deal with the possibility of having <font color=red>multiple POIs across different translation units</font> for all linkable entities produced by template instantiation:
instantiated function templates and member function templates, as well as instantiated static data members and instantiated variable templates. 

three broad classes of solutions that have been used by C++ implementers 

### Greedy Instantiation 

the most commonly used technique 

**linker allows duplicate definitions of linkable entities**, which is also typically used to handle duplicate **spilled inlined functions** and **virtual function dispatch tables**. 

drawbacks:

1. The compiler may be wasting time on generating and optimizing N instantiations, of which only
   one will be kept 
2. Linkers typically do not check that two instantiations are identical 
3. The sum of all the object files could potentially be much larger because of the duplicated code

### Queried Instantiation 

**queried instantiation** did not survive in the marketplace 

The generated specializations are stored with information in the a database shared by the compilations of all translation units. Whenever a point of instantiation for a linkable entity is encountered:

1. If no specialization is available: instantiation occurs, and the resulting specialization enter the database. 
2. If a specialization is available but is out of date because source code changes: instantiation occurs, but the resulting specialization replaces the one previously stored in the database 
3. An up-to-date specialization is available in the database. Nothing needs to be done 

implementation challenges: 

1. It is not trivial to maintain correctly the dependencies of the database contents with respect to the state of the source code, making the 2nd case falls into 3rd
2. concurrency control in the database to support concurrent compilation

problems to the programmer:

1. traditional compilation model inherited from most C compilers no longer applies 
2. any tool that operates on object files may need to be made aware of the contents of the database. 

### Iterated Instantiation 

The first compiler to support C++ templates — Cfront 3.0 — has inflexible constraint that Cfront had to be very portable from platform to platform, means 

1. used the C language as a common target representation across all target platforms 
2. used the local target linker, implying the linker was not aware of templates 

In fact, Cfront emitted template instantiations as ordinary C functions, and therefore it had to avoid duplicate instantiations. Although the Cfront source model was different from the standard inclusion model, its instantiation strategy can be adapted to fit the inclusion model.

The Cfront iteration can be described as follows:
1. Compile the sources without instantiating any required linkable specializations.
2. Link the object files using a prelinker.
3. The prelinker invokes the linker and parses its error messages to determine whether any are the result of missing instantiations. If so, the prelinker invokes the compiler on sources that contain the needed template definitions, with options to generate the missing instantiations.
4. Repeat step 3 if any definitions are generated. 

drawbacks:

1. Time overhead by the prelinker and every required recompilation and relinking 
2. Diagnostics (errors, warnings) are delayed until link time 
3. Cfront in particular used a central repository which has to support concurrent compilations 

## Explicit Instantiation 

**explicit instantiation directive**: keyword ***template*** followed by a declaration of the specialization to be instantiated. 

Members of class templates can also be explicitly instantiated 

```C++
template<typename T>
class S {
    public:
    	void f() {
    	}
};
template void S<int>::f();
template class S<void>;
```

### Manual Instantiation 

place the template definition into a **third source file**, conventionally with the extension `.tpp` and not included except in the translation unit where it is explicitly instantiated. 

can save the time for duplicate instantiation and parsing template definitions but requires manually updating the list of explicit instantiation definitions each time a new specialization is required,  

### Explicit Instantiation Declarations 

use **explicit instantiation declaration**, which is an **explicit instantiation directive** prefixed by the keyword ***extern*** to suppresses most automatic instantiation except:

1. **Inline functions** can still be instantiated for the purpose of expanding (but no separate object code is generated).
2. Variables with deduced ***auto*** or ***decltype(auto)*** types and functions with **deduced return types** can still be instantiated to determine their types.
3. Variables whose values are usable as **constant-expressions** can still be instantiated so their values can be evaluated.
4. Variables of reference types can still be instantiated so the entity they reference can be resolved.
5. Class templates and alias templates can still be instantiated to check the resulting types 

**explicit instantiation declarations** are more like an optimization with not much benifit because
some redundant automatic instantiation is likely to occur and the template definitions are still parsed as part of the header. 

## Compile-Time if Statements 

## In the Standard Library 

it is common for standard library implementations to introduce **explicit instantiation declarations** for commonly used template instance such as various stream classes and `std::basic_string<char> `

# Chapter 15. Template Argument Deduction 

## The Deduction Process 

<font color=red>We describe the basic deduction process in terms of matching a type ***A*** (derived from the **call argument type**) to a parameterized type ***P*** (derived from the **call parameter declaration**). And then attempt to conclude the correct substitution of template parameters from ***P***.</font>

1. If the call parameter is declared with a **reference** declarator, P is taken to be the type referenced, and A is the type of the argument. 
2. Otherwise, P is the declared parameter type, and A is obtained from the type of the argument by **decaying** array and function types to pointer types, ignoring top-level const and volatile qualifiers and reference

## Deduced Contexts 

**Complex type declarations** are built from more elementary constructs (pointer, reference, array, and function declarators; pointer-to-member declarators; template-ids; and so forth), but not

1. **Qualified type names**. For example, a type name like `Q<T>::X` will never be used to deduce a
   template parameter T.

2. Nontype expressions that are not just a nontype parameter. Such as T in 

   `int(&)[sizeof(S<T>)]`

3. The expression of a **decltype-specifier**

```C++
template<int N>
class X {
    public:
        using I = int;
        void f(int) {
    }
};
template<int N>
void fppm(void (X<N>::*p)(typename X<N>::I));

int main()
{
	fppm(&X<33>::f); //  X<N>::I is not deduced context but X<N>::*p is
}
```

## Special Deduction Situations 

### initializer list

When argument of a function call is an initializer list, that argument doesn’t have a specific type, so in general no deduction will be performed from that given pair (A, P) except that parameter type ***P***, after removing references and top-level const and volatile qualifiers, is equivalent to `std::initializer_list<P0>`  

### Literal Operator Templates 

**template literal operator** is only supported for **numeric literals** that are valid even without the suffix 

```C++
template<char...> int operator "" _B7();

auto b = 01.3_B7; // OK: deduces <’0’, ’1’, ’.’, ’3’>
auto c = 0xFF00_B7; // OK: deduces <’0’, ’x’, ’F’, ’F’, ’0’, ’0’>
auto d = 0815_B7; // ERROR: 8 is no valid octal literal
auto e = hello_B7; // ERROR: identifier hello_B7 is not defined
auto f = "hello"_B7; // ERROR: literal operator _B7 does not match
```

### Forwarding References 

Only when composing types through the substitution of template parameters, type aliases, or
decltype constructs, **reference to a reference** is permitted.  

```C++
using RI = int&;
R const& rr = r; // OK: rr has type int&

using RCI = int const&;
RCI volatile&& r = 42; // OK: r has type int const&
using RRI = int&&;
RRI const&& rr = 42; // OK: rr has type int&&
```

When a function parameter is a **forwarding reference** (an rvalue reference to a template parameter of that function template, **template argument deduction** considers not just the type of the function call argument but also whether that argument is an lvalue or an rvalue 

```C++
// Perfect Forwarding
#include <utility>
template<typename... Ts> void forwardToG(Ts&&... xs)
{
	g(std::forward<Ts>(xs)...); // forward all xs to g()
}
```

#### forwarding return value of a function call 

```C++
template<typename... Ts>
auto forwardToG(Ts&&... xs) -> decltype(g(std::forward<Ts>(xs)...))
{
	return g(std::forward<Ts>(xs)...); // forward all xs to g()
}
// since C++14
template<typename... Ts>
decltype(auto) forwardToG(Ts&&... xs)
{
	return g(std::forward<Ts>(xs)...); // forward all xs to g()
}
```

#### perfect forwarding is not, in fact, “perfect” 

it does not distinguish whether an lvalue is a **bit-field** lvalue, nor does it capture <font color=red>whether the expression has a specific constant value</font>

```C++
void g(int*);
void g(...);

template<typename T> void forwardToG(T&& x)
{
	g(std::forward<T>(x)); // forward x to g()
}
void foo()
{
    g(0); // calls g(int*)
    forwardToG(0); // eventually calls g(...)
}
// This is yet another reason to use nullptr (introduced in C++11) instead of null pointer constants:
g(nullptr); // calls g(int*)
forwardToG(nullptr); // eventually calls g(int*)
```

#### prevent forwarding references 

<font color=red>forwarding reference applies only when the form `template-parameter &&`, is part of a function template </font>

```C++
template<typename T>
class X
{
    public:
        X(X&&); // X is not a template parameter
        X(T&&); // this constructor is not a function template
        template<typename U> X(X<U>&&); // X<U> is not a template parameter
        template<typename U> X(U, T&&); // T is a template parameter from
        							// an outer template
};
```



```C++
void int_lvalues(int&); // accepts lvalues of type int
	template<typename T> void lvalues(T&); // accepts lvalues of any type
void int_rvalues(int&&); // accepts rvalues of type int
	template<typename T> void anything(T&&); // SURPRISE: accepts lvalues and
										// rvalues of any type

template<typename T>
    typename std::enable_if<!std::is_lvalue_reference<T>::value>::type
    rvalues(T&&); // accepts rvalues of any type
```

## SFINAE (Substitution Failure Is Not An Error) 

SFINAE was originally intended to eliminate surprising errors due to unintended matches with function template overloading 

### Immediate Context 

**SFINAE** protects against attempts to form invalid types or expressions that occur within the **immediate context** of the **function template substitution** including errors due to ambiguities or access control violations. An error is a real error if not occur in the immediate context of the function template substitution.

Anything that happens during the instantiation is **immediate context** of that **function template substitution** except:

1. the definition of a class template (i.e., its “body” and list of **base classes**),
2. the definition of a function template (“body” and, in the case of a constructor, its constructor initializers),
3. the initializer of a variable template,
4. a default argument,
5. a default member initializer, 
6. an exception specification 
7. any of above which is implicitly triggered by the substitution process 

```C++
template<typename T>
class Array {
    public:
    	using iterator = T*;
};
template<typename T>
	void f(Array<T>::iterator first, Array<T>::iterator last);
template<typename T>
	void f(T*, T*);

int main()
{
	f<int&>(0, 0); // ERROR: trigger instantiation of Array<int&>, within which fails
} 

template<typename T> auto f(T p) {
	return p->m;
}
int f(...);
template<typename T> auto g(T p) -> decltype(f(p));

int main()
{
	g(42); // ERROR: trigger instantiation of f<int>
}
```

## Limits Conversions of Deduction 

Allowable Argument Conversions 

1. If reference declarator, the substituted ***P*** type may be **more const/volatile-qualified** than the ***A*** type.
2. If the ***A*** type is a pointer or pointer-to-member type, it may be convertible to the substituted ***P***
   type by a qualification conversion (add const and/or volatile qualifiers).
3. Unless for a conversion operator template, the substituted ***P*** type may be a base class type of the ***A*** type or a pointer to a base class type of the class type for which ***A*** is a pointer type.

Prior to C++17, template argument deduction applied exclusively to function and member function templates.  

**Default function call arguments** and **exception specifications** do not participate in **template argument deduction**

```C++
template<typename T>
void f (T x = 42)
{ }
int main()
{
    f<int>(); // OK: T = int
    f(); // ERROR: cannot deduce T from default call argument
}
```

## Explicit Function Template Arguments 

Once a template argument is explicitly specified, its corresponding parameter is no longer subject to deduction. Explicitly specified template arguments are substituted using SFINAE principles 

It is possible to explicitly specify some template arguments while having others be deduced. even for **variadic template**

It is occasionally useful to specify an empty template argument list to ensure the selected function is
a template instance while still using deduction to determine the template arguments: 

```C++
int f(int); // #1
template<typename T> T f(T); // #2

int main() {
    auto x = f(42); // calls #1
    auto y = f<>(42); // calls #2
}

void f();
template<typename> void f();
namespace N {
    class C {
        friend int f(); // OK: a new N::f() is “invisibly” declared
        friend int g<>(); // ERROR: template must be visible at that point
        friend int f<>(); // ERROR: return type conflict 
    };
}
```

## Deduction from Initializers and Expressions 

### The ***auto*** Type Specifier 

the old use of ***auto*** as a “storage class specifier” is no longer permitted in C++11 and later 

The ***auto*** type specifier can be used to deduce the type of a variable from initializer. In such cases, ***auto*** is called a **placeholder type** (another placeholder type, `decltype(auto)`)

The **auto** deduction proceeds as if the variable were a function parameter and its initializer the corresponding function argument.  

the ***auto*** specifier can be combined to make a variable const, a pointer, a member pointer, reference and so on, but ***auto*** has to be the **“main” type specifier** of the declaration. 

```C++
template<typename T> struct X { T const m; };

auto const N = 400u; // OK: constant of type unsigned int
auto* gp = (void*)nullptr; // OK: gp has type void*
auto const S::*pm = &X<int>::m; // OK: pm has type int const X<int>::*
X<auto> xa = X<int>(); // ERROR: auto in template argument
int const auto::*pm2 = &X<int>::m; // ERROR: auto is part of the “declarator”

int x;
auto&& rr = 42; // OK: rvalue reference binds to an rvalue (auto = int)
auto&& lr = x; // Also OK: auto = int& and reference collapsing makes
				// lr an lvalue reference
```

#### Deduced Return Types 

C++14 allows **deducible auto placeholder type** can be function return types 

```C++
auto f() { return 42; }
auto lm = [] (int x) { return f(x); };
```

Functions can be declared separately from their definition 

```C++
auto f(); // forward declaration
auto f() { return 42; }

int known();
auto known() { return 42; } // ERROR: incompatible return type

struct S {
	auto f(); // the definition will follow the class definition
};
auto S::f() { return 42; }
```

#### Deducible Nontype Parameters 

C++17 added the ability to declare **nontype template parameters** whose actual types are deduced from the corresponding template argument and <font color=red>the **general constraints** on the type of **nontype template parameters** remain in effect.  </font>

```C++
template<auto V> struct S;
S<42>* ps;
S<3.14>* pd; // ERROR: floating-point nontype argument

template<auto V> struct Value {
	using ArgType = decltype(V);
};

template<auto... VS> struct Values {
};
Values<1, ’x’, nullptr> triplet;// each nontype parameter element of the pack can be deduced to a distinct type
```

#### Special Situations 

initializer list as initializer and function return 

```C++
auto oops { 0, 8, 15 }; // std::initializer_list<int> before C++17, ERROR in C++17
auto val { 2 }; // OK: val has type int in C++17

auto subtleError() {
	return { 1, 2, 3 }; // ERROR
}
```

When **multiple variable declarations** share the same ***auto*** or deduced return types with **multiple return-statement deduction** is performed independently for each but the deduced type must be the same

```C++
char c;
auto *cp = &c, d = c; // OK
auto e = c, f = c+1; // ERROR: deduction mismatch char vs. int

auto f(bool b) {
    if (b) {
        return 42.0; // deduces return type double
    } else {
        return 0; // ERROR: deduction conflict
    }
}
```

If the returned expression calls the function recursively, the program is invalid unless a prior deduction already determined the return type 

```C++
auto f(int n)
{
    if (n > 1) {
    	return n*f(n-1); // ERROR: type of f(n-1) unknown
    } else {
    	return 1;
    }
}

auto f(int n)
{
    if (n <= 1) {
    	return 1; // return type is deduced to be int
    } else {
    	return n*f(n-1); // OK: type of f(n-1) is int and so is type of n*f(n-1)
    }
}
```



### decltype 

1. If e is the name of an entity (such as a variable, function, enumerator, or data member) or a class member access, `decltype(e)` yields the **declared type** of that entity or the denoted class member
2. If e is any other expression, `decltype(e)` produces a type that reflects the **type and value category of that expression** as follows:
   – If e is an lvalue of type T, decltype(e) produces T&.
   – If e is an xvalue of type T, decltype(e) produces T&&.
   – If e is a prvalue of type T, decltype(e) produces T. 

perfectly return

```C++
??? f();
decltype(f()) g() // since C++14, decltype(auto) g() 
{
	return f();
}
```

### **decltype(auto)**

C++14 adds a placeholder type: **decltype(auto)**. The type of a variable, return type, or template
argument is determined from the type of the **associated expression (initializer, return value, or template argument)** by applying the `decltype` construct directly to the expression. So parentheses may be significant (since they are significant for the `decltype` construct 

```C++
int x;
decltype(auto) z = x; // object of type int
decltype(auto) r = (x); // reference of type int&

int g();
decltype(auto) f() {
	int r = g();
	return (r); // run-time ERROR: returns reference to temporary
}
```

Unlike ***auto***, **decltype(auto)** does not allow specifiers or declarator operators that modify its type 

```C++
decltype(auto)* p = (void*)nullptr; // invalid
int const N = 100;
decltype(auto) const NN = N*N; // invalid
```

Since C++17, **decltype(auto)** can also be used for **deducible nontype parameters**

```C++
template<decltype(auto) Val> class S
{
	...
};
constexpr int c = 42;
extern int v = 42;
S<c> sc; // #1 produces S<42>
S<v> sv; // ERROR nontype parameter expect constant argument
S<(v)> sv; // #2 produces S<(int&)v>
```

subtle distinction

```C++
template<typename T, typename U>
auto addA(T t, U u) -> decltype(t+u)
{
	return t + u;
}
void addA(...);

template<typename T, typename U>
auto addB(T t, U u) -> decltype(auto)
{
	return t + u;
}

void addB(...);

struct X {
};

using AddResultA = decltype(addA(X(), X())); // OK: AddResultA is void
using AddResultB = decltype(addB(X(), X())); // ERROR: addB() template must be fully instantiated to determine its return type
```

### Structured Bindings 

Syntactically, a **structured binding** must always have an **auto type** optionally extended by ***const***
and/or ***volatile*** qualifiers and/or **&** and/or **&&** declarator operators (but **not a * pointer declarator** or some other declarator construct). It is followed by a **bracketed list** containing **at least one identifier** (reminiscent of the “capture” list of lambdas) 

Three different kinds of entities can initialize a structured binding: 

1. **class type**, where **all the non-static data members are public**. In that case, the number of bracketed identifiers must equal the number of members 

2. **arrays **

3. `std::tuple-like` classes to have their elements decomposed through a template-based protocol using `get<>()`. If the expression `std::tuple_size<E>::value` is a valid integral constant expression, it must equal the number of bracketed identifiers. This performs an actual initialization of the bracketed initializers which can go wrong

   ```C++
   std::tuple_element<i, E>::type& ni = e.get<i>();
   std::tuple_element<i, E>::type& ni = get<i>(e);
   // if deduced to have a reference type
   std::tuple_element<i, E>::type&& ni = e.get<i>();
   std::tuple_element<i, E>::type&& ni = get<i>(e);
   ```
   
   

```C++
struct MaybeInt { bool valid; int value; };
MaybeInt g();
auto const&& [b, N] = g(); // binds b and N to the members of the result of g()

auto f() -> int(&)[2]; // f() returns reference to int array
auto [ x, y ] = f(); // #1
auto& [ r, s ] = f(); // #2

#include <utility>
enum M {};

template<> class std::tuple_size<M> {
	public:
		static unsigned const value = 2; // map M to a pair of values
};
template<> class std::tuple_element<0, M> {
	public:
		using type = int; // the first value will have type int
};
template<> class std::tuple_element<1, M> {
	public:
	using type = double; // the second value will have type double
};
template<int> auto get(M);
template<> auto get<0>(M) { return 42; }
template<> auto get<1>(M) { return 7.0; }

auto [i, d] = M(); // as if: int&& i = 42; double&& d = 7.0;
```

### Generic Lambdas 

C++14 introduced the notion of “generic” lambdas, where one or more of the parameter types use ***auto***t o deduce the type rather than writing it specifically 

For a **generic lambda**, the **function call operator** becomes a **member function template**

```c++
[] (auto i) {
	return i < 0;
}

class SomeCompilerSpecificNameZ
{
    public:
        SomeCompilerSpecificNameZ(); // only callable by compiler
        template<typename T>
        auto operator() (T i) const
        {
        	return i < 0;
        }
};
```

## Alias Templates 

**Alias templates** are “transparent” with respect to deduction. This is possible because **alias templates cannot be specialized**, which guarantees that **alias templates** can be generically expanded according to its definition

```C++
template<typename T> using A = T;
template<> using A<int> = void; // ERROR, but suppose it were possible...
```

## Class Template Argument Deduction 

C++17 introduces deducing the template parameters of a class type from the arguments specified in an initializer of a variable declaration or a functional-notation type conversion, <font color=red>realized by **implicit deduction guide** generated for every constructor and constructor template of the primary class template</font>

<font color=red>All parameters must be determined by the deduction process or from default arguments. It
is not possible to explicitly specify a few arguments and deduce others </font>

```C++
template<typename T1, typename T2, typename T3 = T2>
class C
{
    public:
        // constructor for 0, 1, 2, or 3 arguments:
        C (T1 x = T1{}, T2 y = T2{}, T3 z = T3{});
};

C c1(22, 44.3, "hi"); // OK in C++17: T1 is int, T2 is double, T3 is char const*
C c2(22, 44.3); // OK in C++17: T1 is int, T2 and T3 are double
C c3("hi", "guy"); // OK in C++17: T1, T2, and T3 are char const*
C c4; // ERROR: T1 and T2 are undefined
C c5("hi"); // ERROR: T2 is undefined

C<string> c10("hi","my", 42); // ERROR: only T1 explicitly specified, T2 not deduced
C<> c11(22, 44.3, 42); // ERROR: neither T1 nor T2 explicitly specified
C<string,string> c12("hi","my"); // OK: T1 and T2 are deduced, T3 has default
```



```C++
template<typename ... Ts> struct Tuple {
    Tuple(Ts...);
    Tuple(Tuple<Ts...> const&);
};
// the implicit deduction guides
template<typename... Ts> Tuple(Ts...) -> Tuple<Ts...>;
template<typename... Ts> Tuple(Tuple<Ts...> const&) -> Tuple<Ts...>;

auto x = Tuple{1,2}; // Tuple<int, int>
Tuple a = x; // Tuple<int, int>
Tuple b(x); // Tuple<int, int>
Tuple c{x, x}; // Tuple<Tuple<int, int>, Tuple<int, int>>
Tuple d{x}; // Tuple<int, int>
auto e = Tuple{x}; // Tuple<int, int>
```

### deduction guide with keyword ***explicit***

The deduction is considered only for **direct-initialization cases**, not for **copy-initialization** cases 

```C++
template<typename T, typename U> struct Z {
    Z(T const&);
    Z(T&&);
};
template<typename T> Z(T const&) -> Z<T, T&>; // #1
template<typename T> explicit Z(T&&) -> Z<T, T>; // #2
Z z1 = 1; // only considers #1 ; same as: Z<int, int&> z1 = 1;
Z z2{2}; // prefers #2 ; same as: Z<int, int> z2{2};
```

### Deduction guides is independent of constructors 

```C++
template<typename T> struct X {
	...
};
template<typename T> struct Y {
    Y(X<T> const&);
    Y(X<T>&&);
};
template<typename T> Y(X<T>) -> Y<T>; // cover the cases of both constructors
```

### controversy caused by implicit deduction guide since C++17

a **braced initializer** with one element deduces differently from a braced initializer with multiple elements 

```C++
template<typename T>
class S {
    private:
    	T a;
    public:
        S(T b) : a(b) {
        }
};

template<typename T> S(T) -> S<T>; // implicit deduction guide

S x{12}; // x has type S<int>
S y{x}; // S<int> not S<S<int>>

std::vector v{1, 2, 3}; // vector<int>, not surprising
std::vector w2{v, v}; // vector<vector<int>>
std::vector w1{v}; // vector<int>!
```



```C++
template<typename T>
struct ValueArg {
	using Type = T;
};
template<typename T>
class S {
    private:
    	T a;
    public:
    	using ArgType = typename ValueArg<T>::Type;
        S(ArgType b) : a(b) { } // disable implicit deduction guides
    // because ValueArg<T>:: is not a deduced context in 
    // template<typename> S(typename ValueArg<T>::Type) -> S<T>;
};
```

In order to maintain backward compatibility, class template argument deduction is disabled if the name of the template is an **injected class name**

```C++
template<typename T> struct X {
    template<typename Iter> X(Iter b, Iter e);
    template<typename Iter> auto f(Iter b, Iter e) {
    	return X(b, e); // injected class name X<T>, not deduction guides
    }
};
```

disable the deduction rule for **forwarding reference T&&**, because

```C++
template<typename T> struct Y {
    Y(T const&);
    Y(T&&);
};
void g(std::string s) {
	Y y = s;
}

template<typename T> Y(T const&) -> Y<T>; // deduced guide Y<std::string>
template<typename T> Y(T&&) -> Y<T>; // deduced guide Y<std::string&> likely result error
```

# Chapter 16. Specialization and Overloading 

two C++ language mechanisms that allow pragmatic deviations from pure generic-ness: 

1. **template specialization**: only the primary template needs to be looked up The specializations
   are considered only afterward to determine which implementation should be used 
2. **overloading of function templates**: all overloaded function templates must be brought into an overload set by looking them up 

## Overloading Function Templates 

### Functions Signatures 

Two functions can coexist in a program if they have distinct signatures, function signatures contains:

1. The **unqualified name** of the **function** (or the name of the **function template** from which it was generated)
2. The **class or namespace scope** of that name and, if the name has **internal linkage**, the **translation unit** in which the name is declared
3. The ***const***, ***volatile***, or ***const volatile*** qualification of **member function**
4. The **& or &&** qualification of **member function**
5. The types of the function parameters (before template parameters are substituted if the function is generated from a function template)
6. **Return type**, if the function is generated from a function template
7. The **template parameters and the template arguments**, if the function is generated from a function template 

<font color=red>The following has different signatures but an error occurs because of **overload ambiguity**</font>

```C++
// rule 7
template<typename T1, typename T2>
void f1(T1, T2);
template<typename T1, typename T2>
void f1(T2, T1);
// rule 6
template<typename T>
long f2(T);
template<typename T>
char f2(T);
```

### Partial Ordering of Overloaded Function Templates 

Overload resolution is decided by partial ordering rules:

1. Ignore function call parameters that are not used, Thai is, **ellipsis parameters** and parameters that uses **default argument**.
2. For left function call parameters, the order is determined by coverage. If neither covers the other, there is no ordering  

```C++
template<typename T>
void t(T*, T const* = nullptr, ...);
template<typename T>
void t(T const*, T*, T* = nullptr);

void example(int* p)
{
	t(p, p);
}
// (A1*, A1 const*) and (A2 const*, A2*) 
// neither can cover the other so the call is ambiguous.
```

## Explicit Specialization 

**Alias template**s are the only form of template that cannot be specialized, either by a **full specialization** or a **partial specialization**. This restriction is necessary to make the use of **template aliases** transparent to the template argument deduction process.

**class templates** and **variable templates** cannot be overloaded, but **explicit specialization**

 ### Full Specialization

A **full specialization declaration** is identical to a normal **class/function declaration** and not a **template declaration**, <font color=red>ATTENTION **ODR**</font>). So out-of-class member definition of a full class template specialization cannot prefix `template<>`

```C++
template<typename T>
class S;

template<> class S<char**> {
    public:
    	void print() const;
};
// the following definition cannot be preceded by template<>
void S<char**>::print() const
{
	std::cout << "pointer to pointer to char\n";
}
```

A **full function template specialization** cannot include **default argument values** because <font color=red>call is entirely resolved based on the **function template**</font>

```C++
template<typename T>
int f(T, T x = 42)
{
	return x;
}
template<> int f(int, int)
{
	return 0;
}

f(1); // f<int>(1, 42)
```

<font color=red>ordinary **static data members** and **member functions** of **class templates** can be fully specialized but the **whole class template** cannot be fully specialized to this type anymore</font>

```C++
template<typename T>
class Outer { // #1
public:
     static int code; // #4
     void print() const { // #5
         std::cout << "generic";
     }
};

template<typename T>
int Outer<T>::code = 6; // #6

template<>
int Outer<void>::code = 12;
template<>
void Outer<void>::print() const
{
     std::cout << "Outer<void>";
}

template<> class Outer<void>{}; // error: redefinition of ‘class Outer<void>’
```

For **member functions** and **static data members**, **outside-of-class declaration must be definition**, but for **specializing members of class templates**, **outside-of-class declaration** may not be definition. See [ODR](#ODR)

```C++
template<>
int Outer<void>::code; // not definition to be initialized with a default constructor.
template<>
void Outer<void>::print() const;
```

<font color=red>nested class template cannot be fully specialized if outer class template is not fully specialized</font>

```C++
template<typename T>
class Outer { // #1
    template<typename U>
    class Inner { // #2
        private:
        	static int count; // #3
        };
};

template<>
    template<>
    class Outer<char>::Inner<wchar_t> {
        public:
        	enum { count = 1 };
    };

template<>
	template<typename X>
	long Outer<wchar_t>::Inner<X>::count;

// the following is not valid C++:
// template<> cannot follow a template parameter list
template<typename X>
	template<> class Outer<X>::Inner<void>; // ERROR
```

### Partial Specialization 

The parameter list of the **partial specialization** cannot have **default arguments**; because <font color=red> resolution is based on primary class template</font>, default arguments of primary class template are used instead. 

The **nontype arguments of the partial specialization** should be either **non-dependent values** or **plain nontype template parameters**, no more complex.

```C++
template<typename T, int I = 3>
class S; // primary template
template<typename T>
class S<int, T>; // ERROR: parameter kind mismatch
template<typename T = int>
class S<T, 10>; // ERROR: no default arguments
template<int I>
class S<int, I*2>; // ERROR: no nontype expressions
```

<font color=red>The standard does not describe how **partially specialize variable templates** are declared or what they mean.  </font>

Why function templates can not be partially specialized is mostly historical 

# Chapter 17 Future Directions  

## Generalized Nontype Template Parameters  

In standard C++, two instances of template are the same type if and only if they have the same arguments. C++ forbids string literal as a template nontype argument because two identical string literals appearing in different source locations are not required to have the same address.

Similarly, the “equality” of two class-type values is not a trivial matter for **literal class type**  

## Partial Specialization of Function Templates  

```C++
template<typename T>
void add (T& x, int i); // a primary template
template<typename T1, typename T2>
void add (T1 a, T2 b); // another (overloaded) primary template
template<typename T>
void add<T*> (T*&, int); // Which primary template does this specialize?
```

## Named Template Arguments  

similar to the “designated initializer” syntax introduced in the 1999 C standard:  

```C++
template<typename T,
Move: typename M = defaultMove<T>,
Copy: typename C = defaultCopy<T>,
Swap: typename S = defaultSwap<T>,
Init: typename I = defaultInit<T>,
Kill: typename K = defaultKill<T>>
class Mutator {
...
};
void test(MatrixList ml)
{
	mySort (ml, Mutator <Matrix, .Swap = matrixSwap>);
}
```

## Overloaded Class Templates  

## Deduction for Nonfinal Pack Expansions  

Template argument deduction for pack expansions only works when the pack expansion occurs at
the end of the parameter or argument list.

one cannot easily extract the last element of the list due to the restrictions

## Regularization of ***void***

irregular type complicates the implementation of generic code

```C++
auto&& r = f(); // error if f() returns void
decltype(auto) r = f(); // error if f() returns void
```

## Type Checking for Templates  

One solution to this problem is to describe the template’s requirements as part of the signature of the template itself using a ***concept***

## Reflective Metaprogramming  

In the context of programming, **reflection** refers to the ability to programmatically inspect features of the program (e.g., answering questions such as `Is a type an integer?` or `What nonstatic data members does a class type contain?`). **Metaprogramming** is the craft of `“programming the program”` . **Reflective metaprogramming** is the craft of automatically synthesizing code that adapts itself to existing properties (often, types) of a program    

## Pack Facilities  

dealing with parameter packs often requires recursive template instantiation techniques  

## Modules  

# Chapter 18 The Polymorphic Power of Templates  

Historically, C++ started with supporting polymorphism only through the use of **inheritance combined with virtual functions**. **Container types** were a primary motivation for the introduction of **templates** into the C++ programming language. And instead of implementing each operation as member functions for every container, the designers of STL identified an **abstract concept of iterators** that can be provided for any kind of linear collection  

Strictly speaking, in C++ parlance, ***dynamic polymorphism*** and ***static polymorphism*** are shortcuts for ***bounded dynamic polymorphism*** and ***unbounded static polymorphism***

***Bridge pattern*** (divide implementation and interface) implemented using inheritance or using templates  

In the context of C++, ***generic programming*** is sometimes defined as programming with **templates**
(whereas ***object-oriented programming*** is thought of as programming with **virtual functions**)  

# Chapter 19 Implementing Traits  

***policy class*** is a class that implements one or more policies for an algorithm through an agreed-upon interface 

***traits class*** is a class aggregating useful types and constants 

The major advantage of ***policy classes*** over **function template** as template parameters is that it
is easier to carry **state information** 

## Type Functions 

Traditional C and C++ functions are called **value functions**

**type functions**: functions that takes some type as arguments and produce a type or a constant as a result such as built-in  `sizeof` and **class templates**

**Explicit specializations** should be taken care for **void**

```C++
template<typename T>
struct AddLValueReferenceT {
  using Type = T&;
};

template<>
struct AddLValueReferenceT<void> {
  using Type = void;
};
template<>
struct AddLValueReferenceT<void const> {
  using Type = void const;
};
template<>
struct AddLValueReferenceT<void volatile> {
  using Type = void volatile;
};
template<>
struct AddLValueReferenceT<void const volatile> {
  using Type = void const volatile;
};

// simpify if no partial specialization
template<typename T>
using AddLValueReferenceT = T&;
template<typename T>
using AddRValueReferenceT = T&&;
```

using ***metafunction forwarding*** to inherit the Type member

```C++
template<typename T>
struct DecayT : RemoveCVT<T> {
};

template<typename T>
struct DecayT<T[]> {
	using Type = T*;
};
template<typename T, std::size_t N>
struct DecayT<T[N]> {
	using Type = T*;
};

template<typename R, typename... Args>
struct DecayT<R(Args...)> {
	using Type = R (*)(Args...);
};
// matches any function type that uses C-style varargs
template<typename R, typename... Args>
struct DecayT<R(Args..., ...)> {
	using Type = R (*)(Args..., ...);
};
```

### Predicate Traits 

a special form of type traits, ***type predicates*** (type functions yielding a Boolean value). 

```C++
template<typename T1, typename T2>
struct IsSameT {
	static constexpr bool value = false;
};
template<typename T>
struct IsSameT<T, T> {
	static constexpr bool value = true;
};
```

we yield not only a Boolean value member but also a **Boolean type base class** to realize ***tag dispatching technique***

```C++
template<bool val>
struct BoolConstant {
	using Type = BoolConstant<val>;
	static constexpr bool value = val;
};
using TrueType = BoolConstant<true>;
using FalseType = BoolConstant<false>;

template<typename T1, typename T2>
struct IsSameT : FalseType
{
};
template<typename T>
struct IsSameT<T, T> : TrueType
{
};

template<typename T>
void fooImpl(T, TrueType)
{
	std::cout << "fooImpl(T,true) for int called\n";
}
template<typename T>
void fooImpl(T, FalseType)
{
	std::cout << "fooImpl(T,false) for other type called\n";
}
template<typename T>
void foo(T t)
{
	fooImpl(t, IsSameT<T,int>{}); // choose impl. depending on whether T is int
}
int main()
{
	foo(42); // calls fooImpl(42, TrueType)
	foo(7.7); // calls fooImpl(42, FalseType)
}
```

### Result Type Traits 

***result type traits*** are very useful when writing operator templates, such as a function template that allows us to add two Array containers of convertible types 

```C++
template<typename T1, typename T2>
Array<RemoveCV<RemoveReference<PlusResult<T1, T2>>>> // avoid Array of reference type
operator+ (Array<T1> const&, Array<T2> const&);

template<typename T1, typename T2>
struct PlusResultT {
	using Type = decltype(T1() + T2());
};
template<typename T1, typename T2>
using PlusResult = typename PlusResultT<T1, T2>::Type;
```

eliminate the value-initialization requirement by `declval()`

```C++
template<typename T1, typename T2>
struct PlusResultT {
	using Type = decltype(std::declval<T1>() + std::declval<T2>());
};

namespace std {
	template<typename T>
	add_rvalue_reference_t<T> declval() noexcept; // noexcept used in the context of the noexcept operator
    // add_rvalue_reference_t allows to work with odd function return type, such as abstract class type
}
```

## SFINAE-Based Traits 

The two main approaches for **SFINAE-based traits**

1. SFINAE out **functions overloads**
2. SFINAE out **partial specializations**

### SFINAE out functions overloads

```c++
#include <type_traits>

template<typename T>
struct IsDefaultConstructibleHelper {
  private:
    // test() trying substitute call of a default constructor for T passed as U:
    template<typename U, typename = decltype(U())>
      static std::true_type test(void*);
    // test() fallback:
    template<typename>
      static std::false_type test(...);
  public:
    using Type = decltype(test<T>(nullptr));
};

template<typename T>
struct IsDefaultConstructibleT : IsDefaultConstructibleHelper<T>::Type {
};

template<typename T>
struct IsDefaultConstructibleT {
  private:
    // ERROR: test() uses T directly, not in immediate context
    template<typename, typename = decltype(T())>
      static char test(void*);
    // test() fallback:
    template<typename>
      static long test(...);
  public:
    static constexpr bool value
        = IsSameT<decltype(test<T>(nullptr)), char>::value;
};
```

### SFINAE out partial specializations 

```C++
#include "issame.hpp"
#include <type_traits>  // defines true_type and false_type

// helper to ignore any number of template parameters: 
// type trait std::void_t<> in C++17
template<typename...> using VoidT = void;

// primary template:
template<typename, typename = VoidT<>>
struct IsDefaultConstructibleT : std::false_type
{
};

// partial specialization (may be SFINAE'd away):
template<typename T>
struct IsDefaultConstructibleT<T, VoidT<decltype(T())>> : std::true_type
{
};

```

Starting with C++14, the C++ standardization committee has recommended that compilers and standard libraries indicate which parts of the standard they have implemented by defining agreed-upon ***feature macros***

```C++
#include <type_traits>
#ifndef __cpp_lib_void_t
namespace std {
	template<typename...> using void_t = void;
}
#endif
```

### Using Generic Lambdas for SFINAE 

Since C++17

```C++
#include <utility>

// helper: checking validity of \T\TI{f}(\TI{args}...)} for F \TI{f} and Args... \TI{args:
template<typename F, typename... Args,
         typename = decltype(std::declval<F>()(std::declval<Args&&>()...))>
std::true_type isValidImpl(void*);

// fallback if helper SFINAE'd out:
template<typename F, typename... Args>
std::false_type isValidImpl(...);

// isValid is a traits factory
// define a lambda that takes a lambda f and returns whether calling f with args is valid
inline constexpr
auto isValid = [](auto f) {
                 return [](auto&&... args) {
                          return decltype(isValidImpl<decltype(f),
                                                      decltype(args)&&...
                                                     >(nullptr)){};
                        };
               };

// helper template to represent a type as a value
template<typename T>
struct TypeT {
    using Type = T;
};

// helper to wrap a type as a value
template<typename T>
constexpr auto type = TypeT<T>{};

// helper to unwrap a wrapped type in unevaluated contexts
template<typename T>
T valueT(TypeT<T>);  // no definition needed

constexpr auto isDefaultConstructible
	= isValid([](auto x) -> decltype((void)decltype(valueT(x))()) {});

isDefaultConstructible(type<int>) // true (int is default-constructible)
isDefaultConstructible(type<int&>) // false (references are not default-constructible)
```

### SFINAE-Friendly Traits 

As a general design principle, a trait template should never fail at instantiation time if given reasonable template arguments as input, the general approach is to check by **SFINAE** in the **immediate context** 

```C++
#include <utility>
template<typename T1, typename T2>
struct PlusResultT {
	using Type = decltype(std::declval<T1>() + std::declval<T2>());
};
template<typename T1, typename T2>
using PlusResult = typename PlusResultT<T1, T2>::Type;

// make the PlusResultT SFINAE-friendly
// primary template:
template<typename, typename, typename = std::void_t<>>
struct HasPlusT : std::false_type
{
};
// partial specialization (may be SFINAE’d away):
template<typename T1, typename T2>
struct HasPlusT<T1, T2, std::void_t<decltype(std::declval<T1>()
		+ std::declval<T2>())>> : std::true_type
{
};
template<typename T1, typename T2, bool = HasPlusT<T1, T2>::value>
struct PlusResultT { // primary template, used when HasPlusT yields true
	using Type = decltype(std::declval<T1>() + std::declval<T2>());
};
template<typename T1, typename T2>
	struct PlusResultT<T1, T2, false> { // partial specialization, used otherwise
};
```

## Traits Convenience 

**constexpr variable templates** offer a way to reduce this verbosity of value

**alias templates** offer a way to reduce verbosity of type, but

1. Alias templates cannot be specialized but some traits are meant to be specialized by users 
2. The use of an alias template will always instantiate the type (e.g., the underlying class template
   specialization), which makes it harder to avoid instantiating traits that don’t make sense for a given
   type 

## Type Classification 

The C++ standard library uses a more fine-grained approach than only to check whether a type is a fundamental type. It first defines primary type categories, where each type matches exactly one type category 

### Identifying Function Types 

```C++
template<typename T>
struct IsFunctionT : std::false_type {        // primary template: no function
};

template<typename R, typename... Params>
struct IsFunctionT<R (Params...)> : std::true_type {      // functions
    using Type = R;
    using ParamsT = Typelist<Params...>;
    static constexpr bool variadic = false;
};

template<typename R, typename... Params>
struct IsFunctionT<R (Params..., ...)> : std::true_type { //  C-style variadic functions
    using Type = R;
    using ParamsT = Typelist<Params...>;
    static constexpr bool variadic = true;
};
```

`IsFunctionT` does not handle all function types because <font color=red>function types can have **const** and **volatile** **qualifiers** as well as **lvalue (&)** and **rvalue (&&)** reference **qualifier** though they are only meaningfully used for **non-static member functions** </font>

### Determining Class Types 

No partial specialization patterns that match class types specifically. 

So use an indirect method to identify class types, by some kind of type or expression that is valid for all class types (and not other type). The most convenient property of class types to utilize in this case is that only **class types** (include **union types**) can be used as the **basis of pointer-to-member types**

```C++ 
#include <type_traits>

template<typename T, typename = std::void_t<>>
struct IsClassT : std::false_type {      // primary template: by default no class
};

template<typename T>
struct IsClassT<T, std::void_t<int T::*>>  // classes can have pointer-to-member
 : std::true_type {
};
```

`std::is_class<>` and `std::is_union<>` require special **compiler support** because distinguishing class/struct types from union types cannot currently be done using any standard core language techniques 

## Policy Traits 

The class template `std::char_traits` is used as a **policy traits** parameter by the **string and I/O stream classes** 

Memory allocation for the standard container types is handled using **policy traits classes** `std::allocator_traits`

# Chapter 20. Overloading based on Type Properties

there are a number of techniques that can be used to achieve overloading of function templates based on type properties 

Not all conceptually more specialized algorithm variants can be directly translated into function templates that provide the right partial ordering behavior  

##  Function Template Overloading

**tag dispatching** supports simple dispatching based on **hierarchical tags**, while `enable_if`
supports more advanced dispatching based on arbitrary sets of properties determined by type traits.  

### Tag Dispatching  

The key to effective use of tag dispatching is in the relationship among the tags.  

```C++
namespace std {
    struct input_iterator_tag { };
    struct output_iterator_tag { };
    struct forward_iterator_tag : public input_iterator_tag { };
    struct bidirectional_iterator_tag : public forward_iterator_tag { };
    struct random_access_iterator_tag : public bidirectional_iterator_tag { };
}

template<typename Iterator, typename Distance>
void advanceIterImpl(Iterator& x, Distance n, std::input_iterator_tag)
{
    while (n > 0) { // linear time
        ++x;
        --n;
	}
}
template<typename Iterator, typename Distance>
void advanceIterImpl(Iterator& x, Distance n, std::random_access_iterator_tag) {
	x += n; // constant time
}

template<typename Iterator, typename Distance>
void advanceIter(Iterator& x, Distance n)
{
    advanceIterImpl(x, n,
    		typename std::iterator_traits<Iterator>::iterator_category());
}
```

### Enabling/Disabling Function Templates  

`std::enable_if`

C++17’s ***constexpr if*** feature avoids the need for `std::enable_if`in many cases.  

Using **constexpr if** is only possible when the difference in the generic component can be expressed entirely within the body of the function template  

## Class Specialization  

### Enabling/Disabling Class Templates
The way to enable/disable different implementations of class templates is to use enabled/disabled partial specializations of class templates.  

### Tag Dispatching for Class Templates  

# Chapter 21. Templates and Inheritance  
some interesting techniques, including the **Curiously Recurring Template Pattern (CRTP)** and **mixins**, combine **templates** and **inheritance**

  ## The Empty Base Class Optimization (EBCO) 

C++ classes are often “empty,” which means that their internal representation does not require any bits of memory at run time. This is the case typically for classes that contain only **type members**, **non-virtual function members**, and **static data member**

<font color=red>The designers of C++ avoid zero-size classes to prevent one object allocated at the same address as another object or subobject of the same type</font>

***EBCO***: no space needs to be allocated for **empty base class** provided that it does not cause it to be allocated to the same address as another object or subobject of the same type

```C++
#include <iostream>

class Empty {
  using Int = int;  // type alias members don't make a class nonempty
};

class EmptyToo : public Empty {
};

class NonEmpty : public Empty, public EmptyToo {
};

int main()
{
  std::cout << "sizeof(Empty):    " << sizeof(Empty) << '\n'; // 1
  std::cout << "sizeof(EmptyToo): " << sizeof(EmptyToo) << '\n'; // 1
  std::cout << "sizeof(NonEmpty): " << sizeof(NonEmpty) << '\n'; // 2
}

```

EBCO is an important optimization for template libraries because a number of techniques rely on the introduction of base classes simply for the purpose of introducing new type aliases or providing extra functionality without adding new data 

### Members as Base Classes 

<font color=red>**EBCO** has no equivalent for data members because it would create some problems with the representation of **pointers to members**.</font>

```C++
/* avoid waste memory but fails if
1. a nonclass type or a union type
2. T1 and T2 may be the same type
3. class may be final
*/
template<typename T1, typename T2>
class MyClass : private T1, private T2 {
};
```

Mostly serious, above method will modify the interface. A more practical tool is to “merge” the potentially empty type parameter with the other member using the ***EBCO***. However, performance gains (for the clients of their libraries) do justify the **added complexity** 

```C++
template<typename CustomClass>
class Optimizable {
  private:
    CustomClass info; // might be empty
    void* storage;
};

template<typename CustomClass>
class Optimizable {
  private:
	BaseMemberPair<CustomClass, void*> info_and_storage;
};
```

## The Curiously Recurring Template Pattern (CRTP) 

<font color=red>This technique achieves a similar effect to the use of **virtual functions**, without the costs (and some flexibility) of **dynamic polymorphism**, by passing a derived class as a template argument to one of its own base classes.  </font>

```C++
template <class T> 
struct Base
{
    void interface()
    {
        // ...
        static_cast<T*>(this)->implementation();
        // ...
    }

    static void static_func()
    {
        // ...
        T::static_sub_func();
        // ...
    }
};

struct Derived : public Base<Derived>
{
    void implementation();
    static void static_sub_func();
};
```

A simple application of **CRTP** consists of keeping track of how many objects of a certain class type were created 

```C++
#include <cstddef>

template<typename CountedType>
class ObjectCounter {
  private:
    inline static std::size_t count = 0;    // number of existing objects

  protected:
    // default constructor
    ObjectCounter() {
        ++count;
    }

    // copy constructor
    ObjectCounter (ObjectCounter<CountedType> const&) {
        ++count;
    }

    // move constructor
    ObjectCounter (ObjectCounter<CountedType> &&) {
        ++count;
    }

    // destructor
    ~ObjectCounter() {
        --count;
    }

  public:
    // return number of existing objects:
    static std::size_t live() {
        return count;
    }
};
```

### Barton-Nackman trick **(outdated)**

In modern C++, the only advantages to defining a friend function in a class template over simply defining an ordinary function template are syntactic 

```C++
// pollute the namespace
bool operator!= (X const& x1, X const& x2) {
	return !(x1 == x2);
}


// another file
class S {
};

template<typename T>
class Wrapper {
  private:
	T object;
  public:
	Wrapper(T obj) : object(obj) {} // implicit conversion from T to Wrapper<T>
	// only work for Wrapper<T> by ADL
	friend bool operator!= (Wrapper<T> const& x1, Wrapper<T> const& x2) 	{
	  return FALSE;
	}
};
```
### Facade Pattern
The **facade pattern** is particularly useful for creating new types that conform to some **specific interface**. New types need only expose some small number of **core operations** to the facade, and the facade takes care of providing a complete and correct **public interface** using a combination of CRTP and the Barton-Nackman trick.

***IteratorFacadeAccess*** is used to hide the interface

```C++
template<typename Derived, typename Value, typename Category,
         typename Reference = Value&, typename Distance = std::ptrdiff_t>
class IteratorFacade
{
 public:
  using value_type = typename std::remove_const<Value>::type;
  using reference = Reference;
  using pointer = Value*;
  using difference_type = Distance;
  using iterator_category = Category;

  Derived& asDerived() { return *static_cast<Derived*>(this); }
  Derived const& asDerived() const {
    return *static_cast<Derived const*>(this);
  }

  // input iterator interface:
  reference operator*() const {
    return IteratorFacadeAccess::dereference(asDerived();
  }
  Derived& operator++() {
    IteratorFacadeAccess::increment(asDerived();
    return asDerived();
  }
  Derived operator++(int) {
    Derived result(asDerived());
    IteratorFacadeAccess::increment(asDerived());
    return result;
  }
  friend bool operator== (IteratorFacade const& lhs,
    IteratorFacade const& rhs) {
      return lhs.asDerived().equals(rhs.asDerived());
    }
};

// `friend' this class to allow IteratorFacade access to core iterator operations:
class IteratorFacadeAccess
{
  // only IteratorFacade can use these definitions
  template<typename Derived, typename Value, typename Category,
           typename Reference, typename Distance>
    friend class IteratorFacade;

  // required of all iterators:
  template<typename Reference, typename Iterator>
  static Reference dereference(Iterator const& i) {
    return i.dereference();
  }
  //...
  // required of bidirectional iterators:
  template<typename Iterator>
  static void decrement(Iterator& i) {
    return i.decrement();
  }

  // required of random-access iterators:
  template<typename Iterator, typename Distance>
  static void advance(Iterator& i, Distance n) {
    return i.advance(n);
  }
  //...
};

template<typename T>
class ListNode
{
 public:
  T value;
  ListNode<T>* next = nullptr;
  ~ListNode() { delete next; }
};

template<typename T>
class ListNodeIterator
 : public IteratorFacade<ListNodeIterator<T>, T,
                         std::forward_iterator_tag>
{
  ListNode<T>* current = nullptr;
  friend class IteratorFacadeAccess;
 public:
  T& dereference() const {
    return current->value;
  }
  void increment() {
    current = current->next;
  }
  bool equals(ListNodeIterator const& other) const {
    return current == other.current;
  }
  ListNodeIterator(ListNode<T>* current = nullptr) : current(current) { }
};
```

## Mixins 

**Mixins** are useful in cases where a template needs some small level of customization 

```C++
template<typename... Mixins>
class Point : public Mixins...
{
  public:
    double x, y;
    Point() : Mixins()..., x(0.0), y(0.0) { }
    Point(double x, double y) : Mixins()..., x(x), y(y) { }
};
class Color
{
  public:
    unsigned char red = 0, green = 0, blue = 0;
};
using MyPoint = Point<Label, Color>;
```

### Curious Mixins 

**Mixins** can be made more powerful by combining them with the **Curiously Recurring Template Pattern (CRTP)**

```C++
template<template<typename>... Mixins>
class Point : public Mixins<Point>...
{
  spublic:
    double x, y;
    Point() : Mixins<Point>()..., x(0.0), y(0.0) { }
    Point(double x, double y) : Mixins<Point>()..., x(x), y(y) { }
};
```

Mixins also allow us to indirectly parameterize other attributes of the derived class, such as the **virtuality of a member function **

```C++
#include <iostream>

class NotVirtual {
};

class Virtual {
  public:
    virtual void foo() {
    }
};

template<typename... Mixins>
class Base : public Mixins... {
  public:
    // the virtuality of foo() depends on its declaration
    // (if any) in the base classes Mixins...
    void foo() {
       std::cout << "Base::foo()" << '\n';
    }
};

template<typename... Mixins>
class Derived : public Base<Mixins...> {
  public:
    void foo() {
       std::cout << "Derived::foo()" << '\n';
    }
};

int main()
{
    Base<NotVirtual>* p1 = new Derived<NotVirtual>;
    p1->foo();  // calls Base::foo()

    Base<Virtual>* p2 = new Derived<Virtual>;
    p2->foo();  // calls Derived::foo()
}
```

## Named Template Arguments 

specify template parameters by keywords

Note that we never actually instantiate objects of the helper class that contain virtual bases. Hence, virtual bases is not a performance or memory consumption issue. 

```C++
template<typename PolicySetter1 = DefaultPolicyArgs,
        typename PolicySetter2 = DefaultPolicyArgs,
        typename PolicySetter3 = DefaultPolicyArgs,
        typename PolicySetter4 = DefaultPolicyArgs>
class BreadSlicer {
	using Policies = PolicySelector<PolicySetter1, PolicySetter2,
								PolicySetter3, PolicySetter4>;
// use Policies::P1, Policies::P2, ... to refer to the various policies
};

class DefaultPolicies {
  public:
    using P1 = DefaultPolicy1;
    using P2 = DefaultPolicy2;
    using P3 = DefaultPolicy3;
    using P4 = DefaultPolicy4;
};
class DefaultPolicyArgs : virtual public DefaultPolicies {
};

template<typename Policy>
class Policy1_is : virtual public DefaultPolicies {
  public:
    using P1 = Policy; // overriding type alias
};
template<typename Policy>
class Policy2_is : virtual public DefaultPolicies {
  public:
    using P2 = Policy; // overriding type alias
};
template<typename Policy>
class Policy3_is : virtual public DefaultPolicies {
  public:
    using P3 = Policy; // overriding type alias
};
template<typename Policy>
class Policy4_is : virtual public DefaultPolicies {
  public:
    using P4 = Policy; // overriding type alias
};

template<typename Base, int D>
class Discriminator : public Base {
};
template<typename Setter1, typename Setter2,
		typename Setter3, typename Setter4>
class PolicySelector : public Discriminator<Setter1,1>,
                        public Discriminator<Setter2,2>,
                        public Discriminator<Setter3,3>,
                        public Discriminator<Setter4,4> {
};

BreadSlicer<Policy3_is<CustomPolicy>> bc;
```

# Chapter 22. Bridging Static and Dynamic Polymorphism 

The code size will increase if `forUpTo` as function template is more complex

```C++
template<typename F>
void forUpTo(int n, F f)
{
  for (int i = 0; i != n; ++i)
  {
    f(i);  // call passed function f for i
  }
}
```

To limit this increase in code size, turn it into **non-template function** by class template `std::function`,  a general-purpose polymorphic function wrapper.

```C++
void forUpTo(int n, std::function<void(int)> f)
{
  for (int i = 0; i != n; ++i)
  {
    f(i);  // call passed function f for i
  }
}
```

`FunctorBridge` is used for both storage and manipulation of the **stored function object**.  

```C++
template<typename R, typename... Args>
class FunctorBridge
{
  public:
    virtual ~FunctorBridge() {
    }
    virtual FunctorBridge* clone() const = 0;
    virtual R invoke(Args... args) const = 0;
};

template<typename Functor, typename R, typename... Args>
class SpecificFunctorBridge : public FunctorBridge<R, Args...> {
  Functor functor;

 public:
  template<typename FunctorFwd>
  SpecificFunctorBridge(FunctorFwd&& functor)
    : functor(std::forward<FunctorFwd>(functor)) {
  }
  virtual SpecificFunctorBridge* clone() const override {
    return new SpecificFunctorBridge(functor);
  }
  virtual R invoke(Args... args) const override {
    return functor(std::forward<Args>(args)...);
  }
};

```

**type erasure** the extra information about the specific type F is lost due to the derived-to-based conversion. The performance of generated code using type erasure hews more closely to that of dynamic polymorphism,   

`std::decay_t` makes the inferred type F suitable for storage  

```C++
// primary template:
template<typename Signature> 
class FunctionPtr;

// partial specialization:
template<typename R, typename... Args> 
class FunctionPtr<R(Args...)>
{
 private:
  FunctorBridge<R, Args...>* bridge;
 public:
  // constructors:
  FunctionPtr() : bridge(nullptr) {
  }
  // construction from arbitrary function objects:
  template<typename F> FunctionPtr(F&& f): bridge(nullptr)
  {
    using Functor = std::decay_t<F>;
    using Bridge = SpecificFunctorBridge<Functor, R, Args...>;
    bridge = new Bridge(std::forward<F>(f));
  }

  // destructor:
  ~FunctionPtr() { 
    delete bridge; 
  }

  // invocation:
  R operator()(Args... args) const;         // see functionptr-cpinv.hpp
};

```

# Chapter 23. Metaprogramming  

approaches to metaprogramming that are in common use in modern C++

1. **Value metaprogramming** based on **recursive template instantiations** or **constexpr evaluation**  
2. **Type metaprogramming** based on **recursive template instantiation**  
3. **hybrid metaprogramming**, use `std::array` ,`std::tuple`  and `std::variant`  in compile time just as **array types ,(simple) struct types and union types** in run time

**hybrid metaprogramming** that uses `std::tuple` and `std::variant` is sometimes called **heterogeneous metaprogramming**.  

## The Cost of Recursive Instantiation  

Template instantiations are not cheap: Even relatively modest class templates can allocate over a kilobyte of storage for every instance, and that storage cannot be reclaimed until compilation as completed  

`std::condition` can reduce this explosion in the number of instantiations.  

## Computational Completeness  

a template metaprogram can contain, which is sufficient to compute anything that is computable  
• State variables: The template parameters
• Loop constructs: Through recursion
• Execution path selection: By using **conditional expressions** or **specializations**
• Integer arithmetic  

## Enumeration Values versus Static Constants  

a drawback of **static constant members** is that they are ***lvalues*** whose address must be passed to **metaprogram** when used, so the compiler has to instantiate and allocate the definition for the **static member**.   

C++11, however, introduced **constexpr static data members**, have the advantage of type checking (as opposed to **enum type**), and that type can be deduced if declared with the **auto type specifier**. 

C++17 added **inline static data members**, which do solve the address issue raised above, and can be used in conjunction with ***constexpr***.  

# Chapter 24. Typelists  
For **type metaprogramming**, the central data structure is the ***typelist***, which, as its name implies, is a list containing types. Adding an element to an existing ***typelist*** creates a new ***typelist*** without modifying the original  

```C++
template<typename... Elements>
class Typelist
{
};

template<typename Head, typename... Tail>
class FrontT<Typelist<Head, Tail...>>
{
 public:
  using Type = Head;
};
template<typename List>
using Front = typename FrontT<List>::Type;

template<typename Head, typename... Tail>
class PopFrontT<Typelist<Head, Tail...>> {
 public:
  using Type = Typelist<Tail...>;
};
template<typename List>
using PopFront = typename PopFrontT<List>::Type;

template<typename... Elements, typename NewElement>
class PushFrontT<Typelist<Elements...>, NewElement> {
 public:
  using Type = Typelist<NewElement, Elements...>;
};
template<typename List, typename NewElement>
using PushFront = typename PushFrontT<List, NewElement>::Type;
```

With fundamental **typelist operations** `Front`, `PopFront`, and `PushFront`, algorithms—such as searches, transformations, reversals—can be implemented as **template metafunctions** operating on ***typelists*** by **recursive template metaprogram** and optimized with **pack expansions** to avoid recursion and reduce template instantiations  

# Chapter 25. Tuples  

In addition to homogeneous containers and array-like types, C++ (and C) also has a nonhomogeneous containment facility: the ***class*** (or ***struct***)  

The positional interface and the ability to easily construct a ***tuple*** from a ***typelist*** make tuples more suitable than ***structs*** for use with **template metaprogramming techniques**.  

## Base Tuple Design

Tuples contain storage for each of the types in the template argument list.  

```C++
template<typename... Types>
class Tuple;

// recursive case:
template<typename Head, typename... Tail>
class Tuple<Head, Tail...>
{
 private:
  Head head;
  Tuple<Tail...> tail;
 public:
  // constructors:
  Tuple() {
  }
  Tuple(Head const& head, Tuple<Tail...> const& tail)
    : head(head), tail(tail) {
  }
  //...

  Head& getHead() { return head; }
  Head const& getHead() const { return head; }
  Tuple<Tail...>& getTail() { return tail; }
  Tuple<Tail...> const& getTail() const { return tail; }
};

// basis case:
template<>
class Tuple<> {
  // no storage required
};

```

## Tuple Algorithms  

Tuple algorithms are particularly interesting because they require both compile-time and run-time
computation. Like the typelist algorithms, applying an algorithm to a tuple may result in a tuple with a completely different type, which requires compile-time computation  

### Tuples as Typelists  

```C++
// determine whether the tuple is empty:
template<>
struct IsEmpty<Tuple<>> {
  static constexpr bool value = true;
};

// extract front element:
template<typename Head, typename... Tail>
class FrontT<Tuple<Head, Tail...>> {
 public:
  using Type = Head;
};

// remove front element:
template<typename Head, typename... Tail>
class PopFrontT<Tuple<Head, Tail...>> {
 public:
  using Type = Tuple<Tail...>;
};

// add element to the front:
template<typename... Types, typename Element>
class PushFrontT<Tuple<Types...>, Element> {
 public:
  using Type = Tuple<Element, Types...>;
};

// add element to the back:
template<typename... Types, typename Element>
class PushBackT<Tuple<Types...>, Element> {
 public:
  using Type = Tuple<Types..., Element>;
};
```

## Optimizing Tuple  

A tuple is a fundamental heterogeneous container with a large number of potential uses. As such, it
is worthwhile to consider what can be done to optimize the use of tuples both at run time (storage, execution time) and at compile time (number of templates instantiations).  

### Tuples and the **EBCO**  

apply empty base class optimization by inheriting from the tail tuple rather than making it a member.  

But tail is in a base class, so it will be initialized before the member head.  

```C++
// recursive case:
template<typename Head, typename... Tail>
class Tuple<Head, Tail...> : private Tuple<Tail...>
{
  private:
    Head head;
  public:
    Head& getHead() { return head; }
    Head const& getHead() const { return head; }
    Tuple<Tail...>& getTail() { return *this; }
    Tuple<Tail...> const& getTail() const { return *this; }
};
```

### Constant-time get()  

offloaded the hard work of matching up the index to the compiler’s template argument deduction
engine to avoid linear number of template instantiations that can affect compile time  

## Tuple Subscript  

unlike `std::vector`, a tuple’s elements can each have a different type, so a tuple’s `operator[]` must be a template where the result type differs depending on the index of the element.  

# Chapter 26. Discriminated Unions  

**discriminated union**, meaning that a variant knows which of its possible value types is currently
active, providing **better type safety** than the equivalent **C++ union**

inheritance is not permitted for a union, so we opt for a **low-level representation** of the variant storage: a character array large enough to hold any of the types and with suitable alignment for any of the types     

# Chapter 27. Expression Templates  



```C++
int main()
{
	SArray<double> x(1000), y(1000);
	x = 1.2*x + x*y;
}

int main()
{
    SArray<double> x(1000), y(1000);
    ...
    // process x = 1.2*x + x*y
    SArray<double> tmp(x);
    tmp *= y;
    x *= 1.2;
    x += tmp;
}
```

use **expression templates** to avoid temporaries and split loops  

```C++
/* 
1.2*x + x*y;
into an object with the following type:
A_Add<A_Mult<A_Scalar<double>,Array<double>>,
A_Mult<Array<double>,Array<double>>>
*/
int main()
{
    SArray<double> x(1000), y(1000);
    ...
    for (int idx = 0; idx<x.size(); ++idx) {
    	x[idx] = 1.2*x[idx] + x[idx]*y[idx];
    }
}
```

# Chapter 28. Debugging Templates  

constraints imposed on template parameters: ***syntactic constraints*** and ***semantic constraints***  

How to ensure that the templates will function for any template arguments as documented? 
How can a user  find out which of the template parameter requirements it violated when the template does not behave as documented  

## Shallow Instantiation  

inserting unused code with no other purpose than to trigger an error if that code is instantiated with template arguments that do not meet the requirements of deeper levels of templates.  

## Static Assertions  

## Archetypes 

Archetypes are user-defined classes that can be used as template arguments to test a template
definition’s adherence to the constraints it imposes on its corresponding template parameter in the most minimal way  

## Tracers  

A tracer is a user-defined class that can be used as an argument for a template to be tested. Often,
a tracer is also an archetype

## Oracles  

Tracers are relatively simple and effective, but they allow us to trace the execution of templates only
for specific input data and for a specific behavior of its related functionality.  

An extension of tracers is known as oracles (or run-time analysis oracles). They are tracers that are connected to an inference engine—a program that can remember assertions and reasons about them to infer certain conclusions.  

# Appendix A. The One-Definition Rule 

***ODR*** means:

1. define noninline functions or noninline objects **once and only once** across all files
2. define classes, inline functions, and inline variables **at most once per translation unit**, making sure that all definitions for the same entity are identical 

<a id=ODR></a>

A declaration can also be a definition, depending on which entity it introduces and how it introduces it: 

1. **Namespaces and namespace aliases**: The declarations of **namespaces** and their **aliases** are <font color=red>always also definitions</font>
2. **Classes, class templates, functions, function templates, member functions, and member function templates**: The declaration is a definition <font color=red>if and only if</font> the declaration includes a brace enclosed body associated with the name. 
3. **Enumerations**: The declaration is a definition <font color=red>if and only if</font> it includes the brace-enclosed list of enumerators 
4. **Local variables and nonstatic data members**: These entities can <font color=red>always</font> be treated as definitions
5. **Global variables**: <font color=red>If</font> the declaration is not directly preceded by a keyword ***extern*** or <font color=red>if</font> it has an initializer
6. **Static data members**: The declaration is a definition <font color=red>if and only if</font> it appears outside the class or class template or it is declared ***inline*** or ***constexpr*** in the class or class template 
7. **Explicit and partial specializations**: The declaration is a definition <font color=red>if</font> the declaration following
   the `template<>` or `template<...>` is itself a definition; explicit specialization of a **static data member or static data member template** is a definition <font color=red>only if</font> it includes an initializer 

# Appendix B. Value Categories 

Each C++ [expression](https://en.cppreference.com/w/cpp/language/expressions) is characterized by two independent properties: a **type** and a **value category**

## Value Categories Since C++11 

1. A **glvalue** is an expression whose evaluation determines the identity of an object, bit-field, or function (i.e., an entity that has storage).
2. A **prvalue** is an expression whose evaluation initializes an object or a bit-field, or computes the value of the operand of an operator.
3. An **xvalue** is a **glvalue** designating an object or bit-field whose resources can be reused (usually because it is about to “expire”—the “x” in xvalue originally came from “eXpiring value”).
4. An **lvalue** is a **glvalue** that is not an **xvalue**.
5. An **rvalue** is an expression that is either a **prvalue** or an **xvalue**. 

## Checking Value Categories with decltype 

For any expression x, `decltype((x))` <font color=red>(note the double parentheses)</font> yields:
1. `type` if x is a **prvalue**
2. `type&` if x is an **lvalue**
3. `type&&` if x is an **xvalue** 

```C++
if constexpr (std::is_lvalue_reference<decltype((e))>::value) {
	std::cout << "expression is lvalue\n";
}
else if constexpr (std::is_rvalue_reference<decltype((e))>::value) {
	std::cout << "expression is xvalue\n";
}
else {
	std::cout << "expression is prvalue\n";
}
```

## Reference Types 

a reference may limit the value category of an expression it can bind to 

1. a **non-const lvalue reference** can only be initialized with a lvalue expression
2. an **rvalue reference** can only be initialized with a **rvalue (prvalue or xvalue)** expression
3. a **const lvalue reference** can be initialized with expression of any value categories.

use of a reference type as the return type affects the value category of a call to that function 

1. A call to a function whose return type is an **lvalue reference** yields an **lvalue**.
2. A call to a function whose return type is an **rvalue reference** to an object type yields an **xvalue** <font color=red>(**rvalue references to function types** always result in **lvalues**).</font>
3. A call to a function that returns a **non-reference type** yields a **prvalue** 

```C++
int& lvalue();
int&& xvalue();
int prvalue();

std::is_same_v<decltype(lvalue()), int&> // yields true because result is lvalue
std::is_same_v<decltype(xvalue()), int&&> // yields true because result is xvalue
std::is_same_v<decltype(prvalue()), int> // yields true because result is prvalue

int& lref1 = lvalue(); // OK: lvalue reference can bind to an lvalue
int& lref2 = prvalue(); // ERROR: lvalue reference cannot bind to a prvalue
int& lref3 = xvalue(); // ERROR: lvalue reference cannot bind to an xvalue

int&& rref1 = lvalue(); // ERROR: rvalue reference cannot bind to an lvalue
int&& rref2 = prvalue(); // OK: rvalue reference can bind to a prvalue
int&& rref3 = xvalue(); // OK: rvalue reference can bind to an xrvalue

const int& clref1 = lvalue(); // OK: const lvalue reference can bind to an lvalue
const int& clref2 = prvalue(); // OK: const lvalue reference can bind to a prvalue
const int& clref3 = xvalue(); // OK: const lvalue reference can bind to an xrvalue

int i1 = lvalue(); // OK: Lvalue-to-Rvalue Conversions 
int i2 = prvalue(); // OK: initialized with a prvalue
int i3 = xvalue(); // OK: const lvalue reference can bind to an xrvalue
```

## Conversions 

## Lvalue-to-Rvalue Conversions 

```C++
int x = 3; // x here is a variable, not an lvalue. 3 is a prvalue initializing
			// the variable x.

int y = x; // x here is an lvalue. The evaluation of that lvalue expression does not
		  // produce the value 3, but a designation of an object containing the value 3.
		 // That lvalue is then then converted to a prvalue, which is what initializes y.
```

## Temporary Materialization since C++17

Since C++17, any time a prvalue validly appears where a glvalue is expected, a temporary object is created and initialized with the prvalue and the prvalue is replaced by an xvalue designating the temporary. 

<font color=red>No copy or move operation is ever needed (this is not an optimization, but a language guarantee)</font>. And temporaries are created only when they are really needed 

```C++
class N {
public:
	N();
	N(N const&) = delete; // this class is neither copyable ...
	N(N&&) = delete; // ... nor movable
};
N make_N() {
	return N{}; // Always creates a conceptual temporary prior to C++17.
} 				// In C++17, no temporary is created at this point.

auto n = make_N(); // ERROR prior to C++17 because the prvalue needs a
				// conceptual copy. OK since C++17, because n is
				// initialized directly from the prvalue
```





# Appendix C. Overload Resolution 

<font color=red>Besides function-call expressions, overload resolution occurs when **address of overloaded set** is needed such as passing to function parameter or initializing function object/reference</font>

match rank from best to worst:

1. Perfect match 
2. minor adjustments including **array to pointer** and the addition of **const **
3. promotion 
4. standard conversions only 
5. user-defined conversions. 
6. ellipsis (...) 

**converting constructor** is not considered during template argument deduction. 

```C++
template<typename T>
class MyString {
public:
	MyString(T const*); // converting constructor
	...
};
template<typename T>
MyString<T> truncate(MyString<T> const&, int);

int main()
{
    MyString<char> str1, str2;
    str1 = truncate<char>("Hello World", 5); // OK
    str2 = truncate("Hello World", 5); // ERROR
}
```

rvalue reference collapse

```C++
template<typename T> void strange(T&&, T&&);
template<typename T> void bizarre(T&&, double&&);
int main()
{
    strange(1.2, 3.4); // OK: with T deduced to double
    double val = 1.2;
    strange(val, val); // OK: with T deduced to double&
    strange(val, 3.4); // ERROR: conflicting deductions
    bizarre(val, val); // ERROR: lvalue val doesn’t match double&&
}
```

## Implied Argument for Member Functions 

```C++
#include <cstddef>
class BadString {
public:
    BadString(char const*);
    // character access through subscripting:
    char& operator[] (std::size_t); // #1
    char const& operator[] (std::size_t) const;
    // implicit conversion to null-terminated byte string:
    operator char* (); // #2
    operator char const* ();
};
int main()
{
    BadString str("correkt");
    /* an overload resolution ambiguity!
    str.BadString::operator[](5) // int to size_t
    str.operator char*[5] // BadString to char*
    */
    str[5] = ’c’;
}
```

there can be **at most one user-defined conversion** in a conversion sequence.

## Prefer Nontemplates or More Specialized Templates 

## Conversion Sequences 

1) zero or one **standard conversion sequence**;
2) zero or one **user-defined conversion**;
3) zero or one **standard conversion sequence** (only if a **user-defined conversion** is used).

<font color=red>a conversion sequence that is a subsequence of another conversion sequence is preferable over the latter sequence  </font>

## Pointer Conversions 

**Pointers and pointers to members** undergo various special standard conversion, from best to worst

1. Derived-to-base conversions for pointers/Base-to-derived conversions for pointers to members 
2. Conversions from an arbitrary pointer type to **void\***
3. Conversions to type **bool**

## Initializer Lists 

**default constructor** better than initializer-list constructors better than other constructors

<font color=red>overall conversion of **initializer list** is given the same ranking as the worst conversion from any given element in the **initializer list** to the element type </font>

```C++
#include <initializer_list>
#include <iostream>
void ovl(std::initializer_list<char>) { // #1
	std::cout << "#1\n";
}
void ovl(std::initializer_list<int>) { // #2
	std::cout << "#2\n";
}
int main()
{
	// #1 int to char standard conversion
    // #2 char to int promotion
	ovl({’h’, ’e’, ’l’, ’l’, ’o’, 0}); // prints #2
}
```

## Functors and Surrogate Functions 

```C++
using FuncType = void (double, int);
class IndirectFunctor {
public:
    void operator()(double, double) const; // #1
    operator FuncType*() const; // #2
};
void activate(IndirectFunctor const& funcObj)
{
    // #1 need not user-defined conversion
    // but for the second argument, #2 is better than #1
	funcObj(3, 5); // ERROR: ambiguous
}
```

# Appendix D. Standard Type Utilities 

1. For traits yielding a type, you can access the type as follows:
    ```C++
    typename std::trait<...>::type
    std::trait_t<...> // since C++14
    ```
2. or traits yielding a value, you can access the value as follows:
    ```C++
    std::trait<...>::value
    std::trait<...>() // implicit conversion to its type
    std::trait<...>{} // implicit conversion to its type
    std::trait_v<...> // since C++17
    ```

## `std::integral_constant`

All standard type traits yielding a value are derived from an instance of the helper class template `std::integral_constant`

```C++
namespace std {
    template<typename T, T val>
    struct integral_constant {
        static constexpr T value = val; // value of the trait
        using value_type = T; // type of the value
        using type = integral_constant<T,val>;
        constexpr operator value_type() const noexcept {
        	return value;
        }
        constexpr value_type operator() () const noexcept { // since C++14
       	 	return value;
        }
    };
}
```

usage

```C++
namespace std {
    template<bool B>
    	using bool_constant = integral_constant<bool, B>; // since C++17
    using true_type = bool_constant<true>;
    using false_type = bool_constant<false>;
}

#include <type_traits>
#include <iostream>
int main()
{
    using namespace std;
    cout << boolalpha;
    using MyType = int;
    cout << is_const<MyType>::value << ’\n’; // prints false
    using VT = is_const<MyType>::value_type; // bool
    using T = is_const<MyType>::type; // integral_constant<bool, false>
    cout << is_same<VT,bool>::value << ’\n’; // prints true
    cout << is_same<T, integral_constant<bool, false>>::value << ’\n’; // prints true

    auto ic = is_const<MyType>(); // object of trait type
    cout << is_same<decltype(ic), is_const<int>>::value << ’\n’; // true
    cout << ic() << ’\n’; // function call (prints false)

    static constexpr auto mytypeIsConst = is_const<MyType>{};
    if constexpr(mytypeIsConst) { // compile-time check since C++17 => false
    ... // discarded statement
    }
    static_assert(!std::is_const<MyType>{}, "MyType should not be const");
}
```

## decltype

Type traits apply directly to types. `decltype` yields

1. type of a variable or function only if the entity is named with no extraneous parentheses;
2. a type that also reflects the type category of the expression for any other expression,

##  traits requirements/preconditions


```C++
std::remove_const_t<int const&> // yields int const&, a reference is not const
std::remove_const_t<std::remove_reference_t<int const&>> // int
std::remove_reference_t<std::remove_const_t<int const&>> // int const
std::decay_t<int const&> // yields int
```

There are cases where type traits have requirements. Not satisfying those requirements results
in undefined behavior.

```C++
make_unsigned_t<int> // unsigned int
make_unsigned_t<int const&> // undefined behavior (hopefully error)

add_rvalue_reference_t<int> // int&&
add_rvalue_reference_t<int const> // int const&&
add_rvalue_reference_t<int const&> // int const& (lvalue-ref remains lvalue-ref)
```

## `std::declval`

```C++
template<typename T>
add_rvalue_reference_t<T> declval() noexcept;
```

## Primary and Composite Type Categories 

<font color=red>In
general, each type belongs to exactly one **primary type category** </font>

### Traits to Check the Primary Type Category 

For type `std::max_align_t`, it might be an **integral** or **floating-point** or **class type**

#### std::is_void < T >::value
Yields ***true*** if type T is **(cv-qualified) void**. 

#### std::is_integral < T >::value
Yields ***true*** if type T is one of the following (cv-qualified) types:

1. **bool**
2. a **character type** (char, signed char, unsigned char, char16_t, char32_t, or wchar_t)
3. an **integer type** (signed or unsigned variants of short, int, long, or long long; this includes `std::size_t` and `std::ptrdiff_t`) 

#### std::is_floating_point < T >::value
Yields ***true*** if type T is **(cv-qualified) float, double, or long double**. 

#### std::is_array < T >::value
Yields ***true*** if type T is a (cv-qualified) array type.
Note that parameter declared as an **array (with or without length)** really has a pointer type.
Note that class `std::array<>` is **not an array type**, but a **class type**

 #### std::is_pointer < T >::value
Yields ***true*** if type T is a **(cv-qualified) pointer**, includes:
1. pointers to static/global (member) functions
2. parameters declared as arrays (with or without length) or function types

This does not include:
1. **pointer-to-member types**
2. the type of **nullptr**, `std::nullptr_t` 

#### std::is_null_pointer < T >::value
Provided since C++14, yields ***true*** if type T is the **(cv-qualified) `std::nullptr_t`**, which is the type of **nullptr**.
```C++
is_null_pointer_v<decltype(nullptr)> // yields true
void* p = nullptr;
is_null_pointer_v<decltype(p)> // yields false (p has not type std::nullptr_t)
```
#### std::is_member_object_pointer < T >::value
#### std::is_member_function_pointer < T >::value
#### std::is_lvalue_reference < T >::value
#### std::is_rvalue_reference < T >::value 

#### std::is_enum < T >::value
Yields ***true*** if type T is a **(cv-qualified) scoped or unscoped enumeration types**. 

#### std::is_class < T >::value
Yields ***true*** if type T is a (cv-qualified) class type declared with ***class*** or ***struct*** except for **unions**, **scoped enumeration type** and `std::nullptr_t`, 
a type generated from instantiating a class template and the type of a **lambda expression** is a **class type** 

#### std::is_union < T >::value
Yields ***true*** if type T is a (cv-qualified) union, including a union generated from a **union template**. 

#### std::is_function < T >::value
Yields ***true*** if type T is a (cv-qualified) function type. 
Yields ***false*** for a **function pointer type**, the type of a **lambda expression**, and any other type.

### Traits to Check the Composite Type Category 

#### std::is_reference < T >::value 

Same as: `is_lvalue_reference_v<T> || is_rvalue_reference_v<T>`

#### std::is_member_pointer < T >::value
Same as: `is_member_object_pointer_v<T> || is_member_function_pointer_v<T> `

#### std::is_arithmetic < T >::value
Same as: `is_integral_v<T> || is_floating_point_v<T> `

#### std::is_fundamental < T >::value 

Same as: `is_arithmetic_v<T> || is_void_v<T> || is_null_pointer_v<T>`
Same as: `!is_compound_v<T>` 

#### std::is_scalar < T >::value
Same as: `is_arithmetic_v<T> || is_enum_v<T> || is_pointer_v<T>
|| is_member_pointer_v<T> || is_null_pointer_v<T>> `

#### std::is_object < T >::value
Same as: `is_scalar_v<T> || is_array_v<T> || is_class_v<T> || is_union_v<T>)`
Same as: `! (is_function_v<T> || is_reference_v<T> || is_void_v<T>)` 

#### std::is_compound < T >::value
Same as: `!is_fundamental_v<T>`
Same as: `is_enum_v<T> || is_array_v<T> || is_class_v<T> || is_union_v<T>
|| is_reference_v<T> || is_pointer_v<T> || is_member_pointer_v<T>
|| is_function_v<T>` 

## Type Properties and Operations 

### Other Type Properties 

#### std::is_signed < T >::value
#### std::is_unsigned < T >::value
if T is an arithmetic type that include/does not include **negative value representations** such as **bool**

For type char, it is **implementation defined** whether it yields true or false.

Yields ***false*** for all **non-arithmetic types** (including enumeration types)

#### std::is_const < T >::value 

#### std::is_volatile < T >::value 

Yields ***true*** if the type is **top-level const/volatile qualified**. 

The language defines **arrays to be const/volatile qualified** if the element type is const/volatile qualified 

```C++
is_const<int* const>::value // true
is_const<int const*>::value // false
is_const<int const&>::value // false
    
is_volatile<int[3]>::value // false
is_volatile<int volatile[3]>::value // true
is_volatile<int[]>::value // false
is_volatile<int volatile[]>::value // true
```

#### std::is_aggregate < T >::value 

Available since C++17, requires that the given type is either **complete** or **(cv-qualified) void**
Yields ***true*** if T is an aggregate type (either an **array** or a **class/struct/union** that has no user-defined, explicit, or inherited constructors, no private or protected non-static data members, no virtual functions, and no virtual, private, or protected base classes)

Helps to find out whether list initialization is required 

#### std::is_trivial < T >::value
#### std::is_trivially_copyable < T >::value
#### std::is_standard_layout < T >::value
#### std::is_pod < T >::value
Same as: `is_trivial_t<T> && is_standard_layout_v<T>`

#### std::is_literal_type < T >::value 

#### std::is_empty < T >::value
Yields ***true*** if T is a class type but not a union type, whose objects hold no data.
Yields ***true*** if T is defined as **class** or **struct** with
– no non-static data members other than bit-fields of length 0
– no virtual member functions
– no virtual base classes
– no nonempty base classes 

#### std::is_polymorphic < T >::value
Yields ***true*** if T is **polymorphic class type** (a class that **declares or inherits a virtual function**). 

#### std::is_abstract < T >::value
Yields ***true*** if T is an **abstract class type** (a class for which no objects can be created because it has at least one **pure virtual member function**). 

#### std::is_final < T >::value
Available since C++14, yields ***true*** if T is an **final class type** (a class or union that can’t serve as a **base class** because it is declared as being ***final***).
For all **non-class/union types**, it returns false 

#### std::has_virtual_destructor < T >::value 

#### std::has_unique_object_representations < T >::value 

#### std::alignment_of < T >::value 

Same as: `alignof(T)`, introduced in C++11 before the `alignof(...)` construct 

#### std::rank < T >::value 
Yields the number of dimensions of an array of type T as `std::size_t`.
Yields 0 for all other types 
Yields 1 for unspecified bound array type

#### std::extent < T >::value
#### std::extent < T, IDX >::value 
Yields the size of the first or IDX-th dimension of an array of type T as std::size_t.
Yields 0, if T is not an array, the dimension doesn’t exist, or the size of the dimension is not known.

#### std::underlying_type < T >::type

Yields the underlying type of an **enumeration type** T 

#### std::is_invocable < T, Args... >::value
#### std::is_nothrow_invocable < T, Args... >::value
#### std::is_invocable_r < RET_T, T, Args... >::value
#### std::is_nothrow_invocable_r < RET_T, T, Args... >::value
Yields ***true*** if we can use T as a callable for` Args...` (with the guarantee that no exception is thrown), returning a value convertible to type `RET_T`.

#### std::invoke_result < T, Args... >::type
#### std::result_of < T, Args... >::type
Yields the return type of the callable T called for `Args...`

`invoke_result<>` is available since C++17 and replaces `result_of<>`, which is deprecated since C++17, because `invoke_result<>` provides some improvements such the easier syntax and permitting abstract types for T.

### Test for Construct/Destroy/Swap Operations 

`std::is_constructible < T, Args... >::value`
`std::is_trivially_constructible < T, Args... >::value`
`std::is_nothrow_constructible < T, Args... >::value`

`std::is_default_constructible < T >::value`
`std::is_trivially_default_constructible < T >::value`
`std::is_nothrow_default_constructible < T >::value`

`std::is_copy_constructible < T >::value`
`std::is_trivially_copy_constructible < T >::value`
`std::is_nothrow_copy_constructible < T >::value`

`std::is_move_constructible < T >::value`
`std::is_trivially_move_constructible < T >::value`
`std::is_nothrow_move_constructible < T >::value` 

need **compiler support** because no way to check whether a move constructor throws without being able to call it directly 

`std::is_assignable < TO, FROM >::value`
`std::is_trivially_assignable < TO, FROM >::value`
`std::is_nothrow_assignable < TO, FROM >::value`

`std::is_copy_assignable < T >::value`
`std::is_trivially_copy_assignable < T >::value`
`std::is_nothrow_copy_assignable < T >::value`

`std::is_move_assignable < T >::value`
`std::is_trivially_move_assignable < T >::value`
`std::is_nothrow_move_assignable < T >::value`

`std::is_destructible < T >::value`
`std::is_trivially_destructible < T >::value`
`std::is_nothrow_destructible < T >::value`

`std::is_swappable < T >::value`
`std::is_nothrow_swappable < T >::value`

`std::is_swappable_with < T1, T2 >::value`
`std::is_nothrow_swappable_with < T1, T2 >::value`

### Relationships Between Types 

#### std::is_same < T1, T2 >::value
Yields ***true*** if T1 and T2 name the same type including **cv-qualifiers (const and volatile)**. 
Yields ***false*** for the **(closure) types** associated with two distinct **lambda expressions** even if the **lambda expressions** are the same . 

```C++
auto a = nullptr;
auto b = nullptr;
is_same_v<decltype(a),decltype(b)> // yields true
using A = int;
is_same_v<A,int> // yields true
auto x = [] (int) {};
auto y = x;
auto z = [] (int) {};
is_same_v<decltype(x),decltype(y)> // yields true
is_same_v<decltype(x),decltype(z)> // yields false
```

#### std::is_base_of < B, D >::value
Yields ***true*** if B is a **base class** of D or B is the **same class** as D. 
Yields ***false*** if at least one of the types is a **union**

#### std::is_convertible < FROM, TO >::value
Yields ***true*** if the following is valid

```C++
TO test() {
	return std::declval<FROM>();
}
```

`is_constructible` does not always imply `is_convertible`

```C++
class C {
  public:
    explicit C(C const&); // no implicit copy constructor
};
is_constructible_v<C,C> // yields true
is_convertible_v<C,C> // yields false
```

## Type Construction 

### cv-qualifier 

add/remove **const** or/and **volatile** qualifiers at the top level 

`std::add_const < T >::type`
`std::add_volatile < T >::type`
`std::add_cv < T >::type `
`std::remove_const < T >::type`
`std::remove_volatile < T >::type`
`std::remove_cv < T >::type` 

### sign

`std::make_signed < T >::type`
`std::make_unsigned < T >::type `

Yields the corresponding **signed/unsigned type** of an enumeration type or a (cv-qualified) integral type other than **bool**, or **no effect** on reference type and function type

All other types lead to **undefined behavior**

### reference/pointer

`std::remove_reference < T >::type `
`std::add_lvalue_reference < T >::type`
`std::add_rvalue_reference < T >::type `

`std::remove_pointer < T >::type `
`std::add_pointer < T >::type `

Yields T if there is no such type

### array

`std::remove_extent < T >::type`
`std::remove_all_extents < T >::type` 

```C++
remove_extent_t<int[5][10]> // yields int[10]
remove_extent_t<int[][10]> // yields int[10]
remove_extent_t<int*> // yields int*
remove_all_extents_t<int> // yields int
remove_all_extents_t<int[10]> // yields int
remove_all_extents_t<int[5][10]> // yields int
remove_all_extents_t<int[][10]> // yields int
remove_all_extents_t<int(*)[5]> // yields int(*)[5]
```

### std::decay < T >::type 

`decay<>` models by-value passing of arguments or the type conversions when initializing an objects of type ***auto***. In detail, for type T the following transformations are performed:

1. First, **remove_reference** is applied.
2. If the result is an array type or **function type**,  convert to pointer type
3. Otherwise, that result is produced without any **top-level const/volatile qualifiers **

## Other Traits 

### enable_if

`std::enable_if < cond >::type`
`std::enable_if < cond, T >::type`

Yields void or T in its member type if cond is true. Otherwise, it does not define a member type.  

### `std::conditional < cond, T, F >::type `

Yields T if cond is true, and F otherwise. 

### `std::common_type < T... >::type `

Recursively replaces the first two types T1 and T2 by their common type. If at any time that process fails, here is no type member defined 

While processing the common type, the passed types are **decays**, so ff a single type is given, the result is the application of `decay_t  `to that type 

### align

`std::aligned_union < MIN_SZ, T... >::type `

Yields a **plain old datatype (POD)** usable as uninitialized storage that has a size of at least `MIN_SZ`
and is suitable to hold any of the given types T1, T2, ..., Tn.
In addition, it yields a **static member** `alignment_value` whose value is the strictest alignment of
all the given types

Requires that at least one type is provided 

`std::aligned_storage < MAX_TYPE_SZ >::type`
`std::aligned_storage < MAX_TYPE_SZ, DEF_ALIGN >::type`

Yields a plain old data type (POD) usable as uninitialized storage that has a size to hold all possible
types with a size up to MAX_TYPE_SZ, taking the default alignment or the alignment passed as DEF_ALIGN into account.
Requires that MAX_TYPE_SZ is greater than zero and the platform has at least one type with the alignment value DEF_ALIGN. 

## Combining Type Traits 

combining multiple type trait predicates by logical operators is not enough:

1. If you have to deal with traits that might fail (e.g., due to incomplete types).
2. If you want to **combine type trait definitions** we can’t use the **logical operators** because we combine the **corresponding trait classes** 

**type traits** `std::conjunction<>`, `std::disjunction<>,` and `std::negation<> ` are provided for this purpose since C++17 

`std::conjunction < B... >::value`
`std::disjunction < B... >::value `
`std::negation < B >::value `

Logically applies operator && or || or !, respectively, to the passed **Boolean traits B... ** and **short-circuit** (abort the evaluation after the first false or true) 

1. dealing with failed traits

```C++
struct X {
  X(int); // converts from int
};
struct Y; // incomplete type

// undefined behavior because of  incomplete type
static_assert(std::is_constructible<X,int>{}
				|| std::is_constructible<Y,int>{},
				"can’t init X or Y from int");
static_assert(std::disjunction<std::is_constructible<X, int>,
				std::is_constructible<Y, int>>{},
				"can’t init X or Y from int");
```

2. an easy way to define new type traits by logically combining existing type traits 

```C++
template<typename T>
struct isNoPtrT : std::negation<std::disjunction<std::is_null_pointer<T>,
				std::is_member_pointer<T>,
				std::is_pointer<T>>>
{
};

template<typename T>
constexpr bool isNoPtr = isNoPtrT<T>::value;
```

## Other Utilities 

### `std::declval <T> () `

```C++
template<typename T>
add_rvalue_reference_t<T> declval() noexcept;
```

Yields an “object” or function of any type **without calling any constructor or initialization** and usually used in unevaluated expressions
If T is **void**, the return type is **void**.

### `std::addressof (r) `

Yields the address of object or function r even if `operator&` is overloaded for its type

# Appendix E. Concepts  
A **concept** is a named set of constraints on one or more template parameters  

***concepts*** are best designed to **model real semantic notions** that arise in the problem domain  

## Using Concepts  

### Shorthand Notation for Single Requirements  

```C++
template<typename T> requires LessThanComparable<T>
T max(T a, T b) {
  return b < a ? a : b;
}
// Shorthand Notation for Single Requirements  
template<LessThanComparable T>
T max(T a, T b) {
	return b < a ? a : b;
}
```

### Dealing with Multiple Requirements  

excessive use of the `|| operator` in requires clauses may potentially tax compilation resources (i.e., make compilation noticeably slower)  

```C++
template<typename Seq>
  requires Sequence<Seq> &&
		   EqualityComparable<typename Seq::value_type>
typename Seq::iterator find(Seq const& seq,
							typename Seq::value_type const& val)
{
  return std::find(seq.begin(), seq.end(), val);
}

template<typename T>
  requires Integral<T> ||
		   FloatingPoint<T>
T power(T b, T p);
```



## Defining Concepts  

When we attempt to use this template, it will not be instantiated until the requires clause has been
evaluated and found to produce a true value  

A ***concept*** is a **Boolean predicate** that evaluates to a **constant-expression**, much like **constexpr variable templates of type bool**, but the type is not explicitly specified:  

```C++
template<typename T>
concept LessThanComparable = requires(T x, T y) {
  { x < y } -> bool; // expression x < y must be valid in a SFINAE sense and result 					of that expression must be convertible to bool
};

template<typename Seq>
concept Sequence = requires(Seq seq) {
  typename Seq::iterator; // type requirement: requires that type exists
  requires Iterator<typename Seq::iterator>; // invoke another concept
  { seq.begin() } -> Seq::iterator;
};
```

## Overloading on Constraints  

***concepts*** participate **function template overload**, that is:

1. for mutually exclusive ***concepts***, overload selects one among them
2. for ***concepts*** that one “subsumes” the other, overload resolution prefers the more specialized one