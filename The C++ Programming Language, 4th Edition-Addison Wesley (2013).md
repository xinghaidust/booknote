[TOC]



# Introduction

ideas and design aims of C++：

1. Leave no room for a lower-level language below C++ (except for assembly code in rare cases) 
2. You don’t pay for what you don’t use. A language feature and a fundamental abstraction must be designed not to waste a single byte or a single processor cycle compared to equivalent alternatives. This is known as the ***zero-overhead principle***

The C++ language features most directly support **four programming styles**: 

- Procedural programming
- Data abstraction
- Object-oriented programming. use of class hierarchies 
- Generic programming 

The main reasons for relying on C were to build on a proven set of low-level language facilities and to be part of a technical community. 

C++ was designed to operate within a single address space, and provided no support for multiple address spaces and no support for multiple processes,  use of multiple processes and multiple address spaces relied on operating system support. 

A nonzero value from `main()` indicates failure. Not every operating system and execution environment make use of that return value: Linux/Unix-based environments often do, but Windows-based environments rarely do. 

<font color=red>Macro substitution is almost never necessary in C++. Use **const**, **constexpr **(can be evaluated at compile time or run time), **enum**, **inline**, **template** instead</font>

**class invariant**: what is assumed to be true for a class, in order to save design work 

**static_assert(A,S)** prints S as a compiler error message if A is not true, most important use is assertions about types used as parameters in generic programming  

A class that provides the interface to a variety of other classes is often called a **polymorphic type**  

One particularly useful kind of template is the function object (sometimes called a functor),  a lambda expression 



# Types and Declarations  

## Implementation 

A C++ implementation can be either ***hosted*** (all standard-library facilities) or ***freestanding*** (fewer standard-library facilities) 

1. built-in types: fundamental types, pointers, and references   
2. user-defined types: enumerations and classes   

prefer `if (p)` over `if (p!=nullptr)` because more directly shorter.

## Types

A **char** must behave identically to either a **signed char** or an **unsigned char**. However, <font color=red>the three char types are distinct</font>, so you can’t mix pointers to different char types.  

```C++
void f(char c, signed char sc, unsigned char uc)
{
  char∗ pc = &uc; // error : no pointer conversion
  signed char∗ psc = pc; // error : no pointer conversion
  unsigned char∗ puc = pc; // error : no pointer conversion
  psc = puc; // error : no pointer conversion
}
```

character literal with more than one characters, for example, 'ab', are, **implementation-dependent**, and best avoided. The type of such a multi-character literal is int  

<font color=red>Unlike plain chars, plain ints are always signed. </font>

a user can define new suffixes for user-defined types. Suffixes not starting with _ are reserved for the standard library.  

```C++
"foo bar"s // a literal of type std::string
123_km // a literal of type Distance
```

implementation-defined aspects of fundamental types can be found in `<limits>`.  

`alignof()` operator returns the alignment of its argument expression.  
type specifier `alignas: alignas(T)` means ‘‘align just like a T.  

## Explicit Type Conversion  

In C++, **explicit type conversion** is unnecessary in most cases when C needs it  
Consider using a run-time checked cast, such as `narrow_cast<>()`, for conversion between numeric types  

1. Construction, using the ` T{v}` notation:perform only ‘‘non-narrowing’’ conversion  
2. Named conversion (`const_cast` `static_cast` `reinterpret_cast` `dynamic_cast`)
3. C-style casts `(T)e` a combination of `static_casts`, `reinterpret_casts`, `const_casts` 
4. Functional notation  `T(e)  ` for a built-in type T, `T(e)` is equivalent to `(T)e`  

## Declarations 

Definitions explicitly specify a ‘‘value’’ for the entities they define.  For types, aliases, templates, functions, and constants, the ‘‘value’’ is permanent. For non-const data types, the initial value may be changed later.   

early versions of C and C++ implicitly uses `int` to be the type when none was specified  

There is no way to use a hidden local name  

a name can be used even to specify its own initial value.  

## Initialization  

***list initialization*** does not allow **narrowing conversion** <font color=red>(convert a value and then convert the result back to its original type and cannot get the original value. )</font>

empty initializer list, `{}`, is used to indicate that a default value is desired

A ***braced-init-list*** is not an expression and therefore has no type. When using **auto**, **list initializer** has its type deduced to `std::initializer_list<T>`, so use = syntax for the initialization in declarations using **auto**  

The implementation model for `{}-lists` comes in three parts:  

1. as constructor arguments as if you had used a `()-list`.  
2. initialize the elements of an aggregate (an array or a class without a constructor )
3. used to construct an **initializer_list** object   

`{}-lists` can appear in two forms:  

1. Qualified by a type, T{...},  
2. Unqualified {...}, for which the the type must be determined from the context of use;  

## initialization of statically allocated objects 

1. **nonlocal** variables in a translation unit are initialized in their **definition order** 
2. **no guaranteed order** of initialization of global variables in different translation units 

<font color=red>Statically allocated objects of C++ can be initialized at run time while those of C must be initialized at compile time.</font>
<font  color=red>However, run-time initialization may cause multi-thread data-race and cannot be used by non-C++ program</font>

## Objects and Values  

1. Has identity: The program has the name of, pointer to, or reference to the object so that it is possible to determine if two objects are the same, whether the value of the object has changed, etc.
2. Movable: The object may be moved from (i.e., we are allowed to move its value to another location and leave the object in a valid but unspecified state, rather than copying  

- **glvalue**: have identity
- **rvalue**: movable
- **xrvalue**: have identity && movable
- **lvalue**: have identity && not movable
- **prvalue**: no identity && movable

classify objects based on their lifetimes:  

1. ***Automatic***: allocated on the stack  
2. ***Static***:
3. ***Free store***:
4. ***Temporary objects***: lifetime is determined by their use. Lifetime will extend to that of the reference if bounded to a reference
5. ***Thread-local objects***: declared `thread_local`  

## Type Aliases  

- type aliases: keyword `using` or keyword `typedef`   
- template alias: keyword `using`

```C++
template<typename T>
using Vector = std::vector<T, My_allocator<T>>;
```

## Pointers  

`void∗` pointer to an object of unknown type, can convert to pointer to any type of object but not to pointer to function or pointer to member

there are differences in the definition of **NULL** in different implementations;   

## String Literals  

C++ provides **raw string literals**  for use with the standard `regex` library  
**raw string literal** can contain a newline  

```C++
string counts {R"(1
22
333)"};
```

The order of string prefixes are significant: **R** is behind the others

```C++
"folder\\file" // implementation character set string
R"(folder\file)" // implementation character raw set string
u8"folder\\file" // UTF-8 string
u8R"(folder\file)" // UTF-8 raw str ing
u"folder\\file" // UTF-16 string
uR"(folder\file)" // UTF-16 raw str ing
U"folder\\file" // UTF-32 string
UR"(folder\file)" // UTF-32 raw str ing
```

**escape sequences**:

1. \xnn... (arbitrary number of hexadecimal digits)
1. \nnn...  (1~3 octal digits)
1. \unnnn (4 hexadecimal digits)
1. \Unnnnnnnn (8 hexadecimal digits)

## References  

A reference is an alternative name for an object, an alias, not an object. No pointer to a reference or array of reference 

references to constants introduces a temporary for a variable  

Prefer references to pointers as arguments, except where ‘‘no object’’ is a reasonable option  

## Structures 

Use of multiple **access specifiers** (i.e., ***public***, ***private***, or ***protected***) can affect structures layout  

It is not possible to declare objects of a ***struct*** until its complete declaration has been seen because the compiler is not able to determine the size of ***struct***

For reasons that reach into the prehistory of C, it is possible to declare a plain name and a ***struct/class/union/enum*** with the same name in the same scope.  

The disadvantage of **std::array** compared to a **built-in array** is that we can’t deduce the number of elements from the length of the initializer 

### Plain Old Data (POD)

a POD (‘‘Plain Old Data’’) is an object that can be manipulated as ‘‘just data’’ without worrying about complications of class layouts or user-defined semantics for construction, copy, and move 

a POD object must be of a **standard layout type**, and a **trivially copyable type**, a type with a **trivial default constructor** <font color=red>(constructor which does not need to do any work  )</font>

- **trivial type** is a type with a trivial default constructor and  trivial copy and move operations  
- Basically, a **standard layout type** is one that has a layout with an obvious equivalent in C and is in the union of what common C++ Application Binary Interfaces (ABIs) can handle. More detailed, a type has **standard layout** unless it  
  - has a non-static member or a base that is not standard layout,  
  - has a virtual function  
  - has a virtual base   
  - has a member that is a reference  
  - has multiple access specifiers for non-static data members   
  - prevents important layout optimizations
    - by having non-static data members in more than one base class or in both the derived class and a base, or
    - by having a base class of the same type as the first non-static data member  

A type is trivially **copyable** unless it has a nontrivial copy operation, move operation, or destructor. Informally, a copy operation is trivial if it can be implemented as a **bitwise copy**. So, what makes a copy, move, or destructor nontrivial?

- It is user-defined.
- Its class has a virtual function.
- Its class has a virtual base.
- Its class has a base or a member that is not trivial.  

template class `is_pod` defined in `<type_traits>` allows to ask the question ‘‘Is T a POD?’  

adding or subtracting non-default constructors does not affect layout or performance (that was not true in C++98)  

### bit-field  

**Bit-fields** are simply a convenient shorthand for using bitwise logical operators   

packing several variables into a single byte  does **not necessarily save space**. It saves data space, but the size of the code needed to manipulate these variables increases on most machine. Furthermore, it is typically much faster to access a **char** or an **int** than to access a **bit-field**.

## Unions  

Use of **unions** can be essential for compactness of data and through that for performance  

**Unions** an overused feature; avoid them when you can because most programs don’t improve much from the use of unions and unions are rather error-prone  

An **anonymous union** is **an object**, not a type, and its members can be accessed without mentioning an object name.  

## Enumerations  

two kinds of enumeration:

1. **enum classes**, not implicitly convert to other types. the default underlying type is **int**
2. **Plain enum**: implicitly convert to integers. no default underlying type, depending on the definition

A **plain enum** can be unnamed, specifying a set of integer constants, rather than a type.

**plain enum** can't be forward declared if not specify the underlying type while **enum classes** can because of default underlying type **int**

# Statements  

**Range-for** Statements  ` for (auto x : v)   ` `v ` must denote a sequence, which can be called `v.begin()` and `v.end()` or `begin(v)` and `end(v)` to obtain an iterator

**goto** Statements does not enter a new scope, and can be very useful when C++ code is generated by a program rather than written directly by a person  

# Expressions  

Size of parameter pack `sizeof... name`
Can expression throw? `noexcept ( expr )`

a set of alternative representation of operations are provided as keywords:  
`and and_eq bitand bitor compl not not_eq or or_eq xor  xor_eq`

a temporary object is destroyed at the end of the **full expression** (an expression that is not a subexpression of some other expression) in which it was created. 

### Constant Expressions  

- **constexpr**: Evaluate at compile time
- **const**: Do not modify in this scope  

For **constant expression**, operands that are not evaluated <font color=red>such as ?: && and ||</font> need not be **constant expressions**.   

A class with a **constexpr constructor** <font color=red>(must have an empty body and all members must be initialized by potentially constant
expressions  )</font> is called a **literal type**  

For a **member function constexpr implies const**, so I did not have to write `const`:

```C++
constexpr Point move(int dx, int dy) const { return {x+dx,y+dy}; }  

constexpr Point move(int dx, int dy) { return {x+dx,y+dy}; }
```

<font color=red>The address of a statically allocated object, such as a global variable, is a constant. however,  is assigned by the linker, rather than the compiler, so the compiler cannot know the value of such an address constant. </font>

```C++
constexpr const char∗ p1 = "asdf";
constexpr const char∗ p2 = p1; // OK
constexpr const char∗ p2 = p1+2; // error : the compiler does not know the value of p1
constexpr char c = p1[2]; // OK, c==’d’; the compiler knows the value pointed to by p1
```

### {}-lists  

features decided not to extend C++   

1. an unqualified list is not allowed on the left-hand side of assignments  

   ```C++
   {v} = 9; // error : not left-hand operand of assignment
   ```

2. not deduce the element type of a container represented as a template because it could be costly in compile time, and the opportunities for ambiguities and confusion if there were many overloaded versions  
 ```C++
   template<class T>
   void f2(const vector<T>&);
   f2({1,2,3}); // error : cannot deduce T
   f2({"Kona","Sidney"}); // error : cannot deduce T
 ```

### Lambda Expressions  

**lambda expression** is a simplified notation for defining and using an **anonymous function object**. 

An function object of a class generated from a **lambda** is called a **closure object** (or simply a **closure**).  

A **lambda expression** consists of a sequence of parts:  

1. A possibly empty capture list delimited by `[]`,  
2. An optional parameter list delimited by `() `  
3. An optional `mutable` specifier, indicating that the lambda expression’s body may modify the data member (captured) of the **closure** 
4. An optional `noexcept` specifier.  
5. An optional return type declaration of the form -> type  
6. A body delimited by `{}`  

Naming the lambda is often a good idea. It also simplifies code layout and allows for recursion  

```C++
auto Modulo_print = [&os,m] (int x) { if (x%m==0) os << x << '\n'; };
```

**capture list** captures local variables

1. `[]`: an empty capture list  
2. `[&]`: implicitly capture the used variables with automatic storage duration by reference 
3. `[=]`: implicitly capture the used variables with automatic storage duration by copy
4. `[capture-list]`: Variables with names preceded by & are captured by reference. Other variables are captured by value  
5. `[&, capture-list]`: *capture-default* is by reference 
6. `[=, capture-list]`: *capture-default* is by copy

capture a **variadic template** argument use` ...`  

```C++
template<typename... Var>
void algo(int s, Var... v)
{
  auto helper = [&s,&v...] { return s∗(h1(v...)+h2(v...)); }
  // ...
}
```

A lambda might outlive its caller because of captured local variables

***this*** is always captured by reference which can lead to race conditions in multi-threaded programs   

A lambda expression’s return type can be deduced from its body. Unfortunately, that is not also done for a function

1. If a lambda body does not have a return-statement, the lambda’s return type is **void** 
2. If a lambda body consists of just a **single** return-statement, the lambda’s return type is the type of the return’s expression. 
3. Otherwise, we have to explicitly supply a return type.   

The type of a lambda, defined to be the type of a function object, called the **closure type**, is unique to the lambda, so no two lambdas have the same type.  

A lambda that captures nothing can be assigned to a pointer to function.  

```C++
double (∗p1)(double) = [](double a) { return sqrt(a); };
double (∗p2)(double) = [&](double a) { return sqrt(a); }; // error : the lambda captures
double (∗p3)(int) = [](int a) { return sqrt(a); }; // error : argument types do not match
```

# Functions 

The type of a function consists of the return type and the argument types. 
<font color=red>For class member functions the name of the class is also part of the function type. </font>

<font color=red>For small objects, say, one to four words, call-by-value is typically a viable alternative and often the one that gives the best performance. </font>

The definition and all declarations for a function must specify the same type. Unfortunately, to preserve C compatibility, a **const** is ignored at the highest level of an argument type 

```C++
// two declarations of the same function
void f(int); // type is void(int)
void f(const int); // type is void(int)
```

callable things in addition to functions:

1. **Constructors**: have no names and cannot be called directly, don’t return a value, can’t take address 
2. **Destructors**: can’t be overloaded and can’t take address
3. **Function objects**: not functions (they are objects) and can’t be overloaded, 
4. **Lambda expressions**: a shorthand for defining function objects. 

### function object is better than pointer to functions

1. class member function defined in-class is trivial to ***inline***  
2. A function object with no data members can be passed with **no run-time cost** 
3. Several operations can be passed as a single object   

**inline specifier** does not affect the semantics of a function. In particular, an **inline function** still has a unique address <font color=red>(compiler will emit a non-inline version if address is needed)</font>, and so do static variables of an inline function. 

**constexpr function** can be thought as a restricted form of **inline functions**  

1. must consist of a single return-statement;
2.  no loops and no local variables are allowed, 
3. allows recursion and conditional expressions
4. cannot write to nonlocal objects

`[[noreturn]]` is one of ***attribute***, indicates that the function is not expected to return. 

five ways of exiting a function: 

1. return-statement 
2. reaching the end of the function body 
3. Throwing an exception that isn’t caught locally 
4. Terminating because an exception was thrown and not caught locally in a noexcept function 
5. Directly or indirectly invoking a system function that doesn’t return (e.g., exit(); 

initializing a local static variable recursively is undefined 

```C++
int fn(int n)
{
  static int n1 = n; // OK
  static int n2 = fn(n−1)+1; // undefined
  return n;
}
```

The main use of references to arrays is in templates, where the number of elements is then deduced 

implement functions with unspecified number of arguments 

1. Use a **variadic template** 
2. Use an `initializer_list` as the argument type 
3. Terminate the argument list with the ellipsis (...), not inherently type-safe 

`va_start()` may modify the stack in such a way that a return cannot successfully be done; `va_end()`undoes any such modifications. 

<font color=red>A default argument is type checked at the time of the function declaration and evaluated at calling time.</font>

```C++
class X {
public:
  static int def_arg;
  void f(int =def_arg);
  // ...
};
int X::def_arg = 7;
void g(X& a)
{
  a.f(); // maybe f(7)
  a.def_arg = 9;
  a.f(); // f(9)
}
```

<font color=red>Return types are not considered in overload resolution.  </font>

## Macros 

In C++, using macros only for conditional compilation and for include guards  

# Exception Handling 

In principle, exception handling can be implemented so that there is no run-time overhead when no
exception is thrown 

No traditional C function throws an exception, so most C functions can be declared **noexcept**. 

**Resource Acquisition Is Initialization (RAII)**: technique for resource management using constructors and destructors 

<font color=red>It's more elegant and more efficient to using ordering and the RAII technique than explicitly handling errors using try-blocks. </font>

deal with the failure of a precondition 

1. ***exception***: needed for long-running systems, systems with stringent reliability requirements, and libraries. 
2. ***ignnore***: the need for performance
3. ***Terminate***: when complete and timely recovery from a precondition failure is considered infeasible 

The C++ exception-handling mechanism is designed to support handling of errors that cannot be handled locally 

Asynchronous events require system-dependent mechanisms fundamentally different from exception, such as signals

‘‘traditional’’ (pre-exception) techniques 

1. To mimic RAII, give every class with a internal state
2. a function can return error_code

The standard offers two simple mechanisms to check desired conditions and invariants:

1. **assert(A)** macro in `<cassert>`: check at run time if and only if the macro NDEBUG is not defined 
2. **static_assert(A,message)**: check at compile time

**exception-safe**: a function must place all constructed objects in **valid states** before a `throw`

C++ standard library provides one of the following guarantees:

1. The **basic guarantee** for all operations: basic invariants of all objects are maintained,
   and no resources are leaked 
2. The **strong guarantee** for key operations: the operation succeeds, or it has no effect (if throw)
3. The **nothrow guarantee** for some simple operations 

Both the **basic guarantee** and the **strong guarantee** are provided on the condition:

1. user-supplied operations do not leave container elements in invalid states,
2. user-supplied operations do not leak resources
3. destructors do not throw exceptions 

we can throw an exception of any type that can be copied or moved 

By default, `terminate()` will call `abort()` and can be changed by providing a terminate handler function by a call `std::set_terminate()` from `<exception>` 

```C++
using terminate_handler = void(∗)(); // from <exception>
[[noreturn]] void my_handler() // a terminate handler cannot return
{
  // handle termination my way
}
void dangerous() // very!
{
  terminate_handler old = set_terminate(my_handler);
  // ...
  set_terminate(old); // restore the old terminate handler
}
```

<font color=red>There is no way of catching exceptions thrown during initialization or destruction of namespace and thread-local variables</font>. This is another reason to avoid global variables whenever possible 

When an exception is caught, the exact point where it was thrown is generally not known, it might
therefore be preferable not to catch exceptions from which the program isn’t designed to recover. 

We can transfer an exception thrown on one thread to a handler on another thread using the standard-library function `current_exception()`

# Namespaces 

## Argument-Dependent Lookup (ADL)

**ADL** is the set of rules for looking up the **unqualified function names** in function-call expressions

associated namespaces:

- If an argument is a class member, the associated namespaces are the class itself (including
  its **base classes**) and the **class’s enclosing namespaces**.
- If an argument is a member of a namespace, the associated namespaces are the **enclosing namespaces**.
- If an argument is a built-in type, there are no associated namespaces 

## Namespace Composition 

Only if we need to define something do we need to know the real namespace of an entity: 

```C++
namespace His_string {
  class String { /* ... */ };
  String operator+(const String&, const String&);
  String operator+(const String&, const char∗);
  void fill(char);
  // ...
}
namespace Her_vector {
  template<class T>
    class Vector { /* ... */ };
  // ...
}
namespace My_lib {
  using namespace His_string;
  using namespace Her_vector;
  void my_fct(String&);
}

void My_lib::fill(char c) // error : no fill() declared in My_lib
{
  // ...
}
void His_string::fill(char c) // OK: fill() declared in His_string
{
  // ...
}
void My_lib::my_fct(String& v)// OK: String is My_lib::String, meaning His_string::Str ing
{
  // ...
}
```



```C++
namespace My_lib {
  using namespace His_lib; // everything from His_lib
  using namespace Her_lib; // everything from Her_lib
  using His_lib::String; // resolve potential clash in favor of His_lib
  using Her_lib::Vector; // resolve potential clash in favor of Her_lib
}
```

an unnamed namespace has an implied **using-directive** 

# Source Files and Programs 

<font color=red>object files contains information about namespaces of C++ object, which is used when linking</font>

The keyword `const` implies default internal linkage, `extern` is needed to indicate external linkage

The effect of an unnamed namespace is very similar to that of internal linkage. 

Checking against inconsistent class definitions in separate translation units is beyond the ability of most C++ implementations 

## Standard-Library Headers 

No suffix is needed for standard-library header. 
The absence of a .h suffix does not imply anything about how the header is stored. 
standard headers are not required to be stored in a conventional manner.  

## Linkage to Non-C++ Code 

`extern "C"` names a linkage convention and not a language. Often, `extern "C"` is used to link to Fortran and assembler routines that happen to conform to the conventions of a C implementation. 

## Program Termination
A program can terminate in several ways:
1. By returning from **main()**
2. By calling **exit()**, **exit()** will call destructors for constructed static objects, but not for local variables of the calling function  
3. By calling **abort()**
4. By throwing an uncaught exception 
5. By violating **noexcept**
6. By calling **quick_exit()**, does not invoke any destructors but invoke functions registered by **at_quick_exit()**. 
7. Other ill-behaved and implementation-dependent ways of making a program crash  

The C (and C++) standard-library function **atexit()** offers the possibility to have code executed at program termination. Basically, **atexit()** is a C workaround for the lack of destructors. 

An argument to **atexit()** cannot take arguments or return a result, and there is an implementation-defined limit to the number of **atexit** functions. A nonzero value returned by **atexit()** indicates that the limit is reached.  



# Classes

the default semantics is memberwise copy 

The protection of private data relies on restriction of the use of the class member names. It can therefore be circumvented by address manipulation and explicit type conversion 

A constructor declared with the keyword ***explicit*** can only be used for initialization and explicit conversions. In the definition outside the class, ***explicit*** cannot be repeated.

solution to inconsistence of physical and logical Constness 

1. use a `const_cast` 
2. define the member to be `mutable` 
3. For complicated cases, place the changing data in a separate object and access it indirectly (e.g. by pointer)

***this*** is considered an ***rvalue***, so it is not possible to take the address 

Explicit use of this is required for access to members of base classes from a derived class that is a template 

In multi-threaded code, **static** data members require some kind of locking or access discipline to avoid race conditions 

<font color=red>**Nested classes** are independent classes and cannot access directly to **non-static** member of enclosing class </font>

**Concrete types/classes** , distinguish them from abstract classes, have been called **value types** and their use **value-oriented programming**.  

<font color=red>use multiple **access specifiers** for data members may cause implementation-dependent reorder of data members in layout of class object.</font>

## Construction, Cleanup, Copy, and Move 

The name of a class can only be used for constructor, within the class 

```C++
struct S {
  S(); // fine
  void S(int); // error : no type can be specified for a constructor
  int S; // error : the class name must denote a constructor
  enum S { foo, bar }; // error : the class name must denote a constructor
};
```

<font color=red>The built-in types are considered to have default and copy constructors, most often used for template arguments.  However, for a built-in type the default constructor is not invoked for uninitialized non-static variables.</font>

1. If either a default constructor or an initializer-list constructor could be invoked, prefer the default constructor. 
2. If both an initializer-list constructor and an ‘‘ordinary constructor’’ could be invoked, prefer the initializer-list constructor, necessary to avoid different resolutions based on different numbers of elements.  

```C++
struct X {
  X(initializer_list<int>);
  X();
  X(int);
};
X x0 {}; // empty list: default constructor or initializer-list constructor? (the default constructor)
X x1 {1}; // one integer: an int argument or a list of one element? (the initializer-list constructor)
```

<font color=red>`initializer_list` elements are **immutable**, we cannot apply a move constructor </font>

some constructors are specified as ***explicit*** to avoid ambiguities in initialization.

Before the constructor is called, members are **initialized just as defined in the same scope of the class object, no matter provided In-Class Initializers or not**

```c++
struct Work {
  string author; // default initialized to ""
  string name; // default initialized to ""
  int year; // default initialized to 0 if static
};

Work alpha; // the value is {"","",0}
void f()
{
  Work beta; // the value is {"","",unknown}
  // ...
}


class Club { 
  string name; // default initialized to ""
  vector<string> members; // default initialized to empty
  vector<string> officers; // default initialized to empty
  Date founded;
// ...
Club(const string& n, Date fd);
};

Club::Club(const string& n, Date fd)
: name{n}, founded{fd} // equivalent to name{n}, members{}, officers{}, founded{fd}
{
  // ...
}


class A {
public:
  A() {} // use in-class initializers
  A(int a_val) :a{a_val} {} // override value 7 of a
  A(D d) :b{g(d)} {} // override value 5 of b
  // ...
private:
  int a {7}; // the meaning of 7 for a is ...
  int b {5}; // the meaning of 5 for b is ...
  HashFunction algorithm {"MD5"}; // Cr yptographic hash to be applied to all As
  string state {"Constructor run"}; // String indicating state in object lifecycle
};
```

<font color=red>`()` cannot be used for in-class member initializers for technical reasons related to parsing and name lookup </font>

**static** member must be defined outside the class or defined inside class as **const/constexpr of literal type**

Often the better alternative to **deep copy** is not a **shallow copy**, but a **move** operation  

two major tools to **prevent slicing**:  

1. Prohibit copying of the base class: delete the copy operations
2. Prevent conversion of a pointer to a derived to a pointer to a base by making the **base class** a **private or protected base**

In particular, **move operations** typically do not throw exceptions; they don’t acquire resources or do complicated operations, so they don’t need to  

Built-in types are considered to have move operations that simply copy  

Using `=default` is always better than your own implementation because of optimizations and more maintainable

The default construct/copy/move/destruct operation, generated by compiler, is to apply the operation to each base and **non-static** data member of the class  



## Operator Overloading 

`::` `.` `.*` cannot be overloaded 

`sizeof` `alignof` `typeid` cannot be overloaded (for no particularly fundamental reason): 

`?:` ternary conditional expression operator cannot be overloaded (for no particularly fundamental reason): 



Operators defined in namespaces can be found based on their operand types just as functions can be
found based on their argument types so that users can supply new meanings for an operator without
modifying existing class declarations 

For many types, **individual access** (sometimes referred to as **get-and-set functions**) is an invitation to disaster.  

The **call operator** is also known as the **application operator**.  

An operator function must either be a member or take at least one argument of a user-defined type except that <font color=red>Class-specific `operator new()s` and `operator delete()s` are **implicitly static members**. </font>

### User-defined Literals  

**literal operators** `operator""` maps literals of built-in types plus a given suffix into a desired type.   

There are only four kinds of literals that can be suffixed to make a user-defined literal  

1. An **integer literal**: accepted by a **literal operator** taking an **unsigned long long** or a
   **const char∗** argument or by a **template literal operator**,  
2. A **floating-point literal**: accepted by a **literal operator** taking an **long double** or a
   **const char∗** argument or by a **template literal operator**,  
3. A **string literal**: accepted by a **literal operator** taking a **(const char∗, size_t)** pair of
   arguments  
4. A **character literal**: accepted by a **literal operator** taking a character argument of
   type **char, wchar_t, char16_t, or char32**_  

A **template literal operator** is a literal operator that takes its argument as a template parameter pack, rather than as a function argument.

The standard library reserves all suffixes not starting with an initial underscore,  

```C++
template<char...>
constexpr int operator"" _b3(); // base 3, i.e., ternar y

201_b3 // means operator"" b3<’2’,’0’,’1’>(); so 9*2+0*3+1 == 19
241_b3 // means operator"" b3<’2’,’4’,’1’>(); so error: 4 isn’t a ternar y digit
```

## Derived Classes  

1. **compile-time polymorphism (or static polymorphism)**: classes template
2. **run-time polymorphism (or dynamic polymorphism)**: interface inheritance, more specifically, a type with virtual functions

A **virtual member function** is sometimes called a **method**.  

**contextual keyword** `override` `final`: have a special meaning only in a few contexts but can be used as an identifier elsewhere  

It is best to use **Inheriting Constructors** from base class by `using` only where no data members are added in the derived classes.  

There is a relaxation of the rule (called **covariant return rule**, applies **only to return types that are pointers or references**) that the type of an overriding function must be the same as the type of the virtual function it overrides  

 **covariant return rule**: one can return more derived type when overriding a method

<font color=red>declaring data members ***protected*** is usually a design error.</font>

a pointer to member is to indirectly refer to a member of a class.   

## multiple inheritance  

the use of one base class for **implementation details** and another for **interface (the abstract
class)** is common to all languages supporting inheritance and compile-time checked interfaces.   

the best way to resolve **virtual function** name ambiguity of **multiple base classes** is to 

1. define a new function in the derived class if all virtual functions need only one interface
2. add intermediate classes if every virtual functions need its own interface

<font color=red>A virtual base is always considered a direct base of its most derived class.</font>

When using an abstract class (without any shared data) as an interface,  we have two choices with no fundamental run-time or space difference:  

1. Replicate the interface class. <font color=red>no implicit conversion to base class because of ambiguous</font>  
2. Make the interface class ***virtual*** to share a simple object   

A class that provides some – but not all – of the implementation for a virtual base class is often called a ***mixin***.  

## Run-Time Type Information  

The use of type information at run time is conventionally referred to as ‘**‘run-time type information**,’’ often abbreviated to `RTTI`.
Casting from a base class to a derived class is often called a **downcast** because of the convention of drawing inheritance trees growing from the root down.  

**dynamic_cast** requires a pointer or a reference to a polymorphic type in order to do a **downcast** or a **crosscast** and return **nullptr** if no or more than one subobject    

**dynamic_cast** is restricted to **polymorphic types** whose virtual function table contains **type_info**, but the target type need not be polymorphic  

**dynamic_cast** can be used to ask an object if it provides a given interface  

***Double Dispatch*** shows how to select a virtual function based on two types in the class hierarchy  

`typeid()` can find the type of an object referred to by a reference or a pointer:  

<font color=red>There is no relation between the relationships defined by `before` member of `type_info` and inheritance relationships. </font>



# Template  

When designing a class template, it is usually a good idea to debug a particular class.

<font color=red>The set of requirements on template argument is called `concept` which is not a language construct in C++  </font>

Types generated from a single template by different template arguments are different types. In particular, generated types from related arguments are not automatically related.   

In a class template

1. non-static data members can be **const**, but unfortunately not **constexpr**.  
2. a **virtual member function cannot also be a member function template**, because <font color=red>the linker would have to add a new entry to the virtual table for class template if someone calls virtual member function template</font>

For technical reasons, a template constructor is never used to generate a copy constructor,   

Avoid nested types in templates unless they genuinely rely on every template parameter.  

<font color=red>A complicated pattern of friendship is almost certainly a design error.  </font>

<font color=red>a substitution failure is not an error. It simply causes the template to be ignored  </font>

## Generic Programming 

Templates offer: 

1. The ability to pass argument types without loss of information 
2. Delayed type checking (done at instantiation time) 
3. The ability to pass constant values as arguments. 

Templates provide (compile-time) **parametric polymorphism** to support **generic programming** 

In the context of C++, **generic programming’** implies an emphasis on the design of general algorithms implemented using templates. 

**template metaprogramming**: seeing templates as type and function generators) and relying on type functions to express compile-time computation 

Design generic algorithms by ‘‘lifting’’ from concrete examples 

we can place a requirement on class template arguments or just to template arguments on individual class function members.  

use **static_assert** to performs compile-time checks of template argument properties 
Adding constraints checks makes the requirements on template arguments explicit. These constraints checks are a technique for making checking of designs based on concepts more robust, rather than an integral part of the type system. 

## Template Parameters

·A template can take parameters:

1. Type parameters. The type must be in scope and accessible  
2. Value parameters of built-in types such as ints and pointers to functions  
3. Template parameters  <font color=red>only class template allowed</font>

It is best to think of template value arguments as a mechanism for passing integers and pointers to functions. 
<font color=red>Literal types cannot be used as template value parametersi in order to simplify implementation of separately compiled translation units </font>

```C++
template<typename T, template<typename> class C>
class Xrefd {
  C<T> mems;
  C<T∗> refs;
  // ...
};
template<typename T>
  using My_vec = vector<T>; // use default allocator

Xrefd<Entry,My_vec> x1; // store cross references for Entrys in a vector

template<typename T>
class My_container {
  // ...
};
Xrefd<Record,My_container> x2; // store cross references for Records in a My_container
```

### Default Template Arguments  

Not allowing an ‘‘empty’’ argument to mean ‘‘use the default’’ was a deliberate trade-off between
flexibility and the opportunity for obscure errors.  

## Specialization  

specializations explicitly written by the programmer is called **explicit specialization**, **user-defined specialization**, or simply a **user specialization**, while the classes and functions generated from templates by compiler is called **generated specializations**

**specialization differs from overloading, but makes no practical difference.**  

the template argument, which be deduced from the function argument list, need not be specified explicitly

```C++
// OK (redundant)
template<>
bool less<const char∗>(const char∗ a, const char∗ b)
{
  return strcmp(a,b)<0;
}

template<>
bool less<>(const char∗ a, const char∗ b)
{
  return strcmp(a,b)<0;
}

// overloading
bool less(const char∗ a, const char∗ b)
{
  return strcmp(a,b)<0;
}
```

## Instantiation  

Define **template functions** to minimize dependencies on nonlocal information because a template will be used to generate functions and classes based on unknown types and in unknown contexts  

The process of finding the declaration for each name explicitly or implicitly used in a template is called **name binding**, and also follows rules of ADL

1. **Nondependent names** are bound at the **point of definition** of the template 
2. **Dependent names**  are bound at a **point of instantiation**

<font color=red>Use scope operator `::`  to be specific about function names to prevent overaggressive ADL</font>

### Templates and Hierarchies  

a class template is sometimes called a **type generator**. 

<font color=red>When having to express a general idea in code, consider whether to represent it as a template or as a class hierachy </font>

Be careful to define logically meaningful conversions between classes generated from the same template. If in doubt, use a named conversion function, rather than a conversion operator  

<font color=red>caution is recommended when mixing object-oriented and generic techniques.  </font>

The combination of templates and class hierarchies is often superior to either without the other. In fact, a template argument can be used as a base class.  

***Barton-Nackman trick***  

## Meta-programming  

<font color=red>Templates were designed to be very general and able to generate optimal code  
In fact, they constitute a complete compile-time functional programming language </font>

think of templates as generators: they are used to make classes and functions.  

template programming in writing programs that compute at compile time and generate programs is called **two-level programming**, **multilevel programming**, **generative programming**, and – more commonly – **template meta-programming**.  

benefits of meta-programming techniques:

1. Improved type safety:  
2. Improved run-time performance: improve the opportunities for ***inlining*** and using compact data structures   

a meta-program is a compile-time computation yielding types or functions to be used at run time  

The most obvious constraint on meta-programming is that code depending on complicated uses of templates can be hard to read and very hard to debug  

Prefer systematic techniques, such as specialization and the use of aliases, to macro hackery. Prefer **constexpr functions** to templates for compile-time computation  

### Type Functions  

Type functions, such as `sizeof(T)`, are **compile-time functions**. That is, they can only take arguments (types and values) that are known at compile time and produce results (types and values) that can be used at compile time.   

C++ type functions are mostly templates.   

A predicate is a function that returns a Boolean value  

Since the rules for being a POD are tricky, `std::is_pod` is most likely a compiler intrinsic rather than implemented in the library as C++ code.  

<font color=red>Note the use of `>(greater than)`  in template</font>

```C++
Conditional<sizeof(int)>4,X,Y>{}(7); // error, Conditional<sizeof(int)>
Conditional<(sizeof(int)>4),X,Y>{}(7); // ok
Conditional<4<sizeof(int),X,Y>{}(7); // ok
```

### Traits  

The standard library relies heavily on ***traits*** which is used to associate properties with a type  

The standard library provides **allocator_traits**, **char_traits**, **iterator_traits**, **regex_traits**, **pointer_traits**, **time_traits**, **type_traits** 

```C++
template<typename Iterator>
struct iterator_traits {
  using difference_type = typename Iterator::difference_type;
  using value_type = typename Iterator::value_type;
  using pointer = typename Iterator::pointer;
  using reference = typename Iterator::reference;
  using iterator_category = typename Iterator::iterator_category;
};
```

### Control Structures  

#### Selection  

1. Conditional: a way of choosing between two types (an alias for **std::conditional**)
2. Select: a way of choosing among several types  

```C++
template<bool C, typename T, typename F> // general template
struct conditional {
  using type = T;
};
template<typename T, typename F> // specialization for false
struct conditional<false,T,F> {
  using type = F;
};

if (My_cond<T>())
  using Type = Square; // error : declaration as if-statement branch
else
  using Type = Cube; // error : declaration as if-statement branch
```

#### Iteration and Recursion  

In general, if we want to iterate over a set of values at compile time, we use recursion.  

```C++
template<int N>
constexpr int fac()
{
  return N∗fac<N−1>();
}}
template<>
constexpr int fac<1>()
{
  return 1;
}
constexpr int x5 = fac<5>();
```

#### Enable_if  

```c++
template<bool B, typename T = void>
struct std::enable_if {
  typedef T type;
};

template<typename T>
  struct std::enable_if<false , T> {}; // no ::type if B==false

template<bool B, typename T = void>
  using Enable_if = typename std::enable_if<B,T>::type;
```

#### Variadic Templates  

The `ellipsis (...)` ( (means ‘‘zero or more occurrences of something’’)  ) is a separate lexical token, so you can place whitespace before or after it.

```C++
emplate<typename T, typename... Args>
void printf(const char∗ s, T value, Args... args)
{
  // ...
  return printf(++s, args...); // do a recursive call with the elements of args   as arguments
  // ...
}

template<typename... Bases>
class X : public Bases... {
public:
  X(const Bases&... b) : Bases(b)... { }
};
```

# Standard-Library Overview  

**Language Support for standard library**：

1.  **initializer_list** Support  
2. A **range-for** statement is mapped to a **for-statement** using an iterator  
3. Error Handling: The standard library is designed so that all facilities obey ‘‘the basic guarantee’’   
4. Exceptions: it is a good idea to always somewhere catch all exceptions.  

## Container Overview  

1. **Sequence containers** provide access to (half-open) sequences of elements, `vector<T,A> list<T,A> forward_list<T,A> deque<T,A>`

2. **Associative containers** provide associative lookup based on a key  
   
    1. **Ordered Associative Containers** implemented as **balanced binary trees** (usually **red-black trees**). `map<K,V,C,A> multimap<K,V,C,A> set<K,C,A> multiset<K,C,A>        `
    
    2. **Unordered Associative Containers** implemented as **hash tables** with linked overflow **(Load and Buckets: load factor 70% is often a good choice.  **).
    
       `unordered_map<K,V,H,E,A> unordered_multimap<K,V,H,E,A> unordered_set<K,H,E,A> unordered_multiset<K,H,E,A>           `
    
3. **Container adaptors** provide specialized access to underlying containers. not offer iterators or subscripting  `priority_queue<T,C,Cmp> queue<T,C> stack<T,C>       `

5. **Almost containers** are sequences of elements that provide most, but not all, of the facilities
   of a container. `T[N] array<T,N> basic_string<C,Tr,A> valarray<T> bitset<N> vector<bool>             `

Like a **built-in array**, an **array** is simply a sequence of elements, with no handle:  <font color=red>An **array** is best understood as a built-in array with its size firmly attached, without implicit, potentially surprising conversions to pointer types, and with a few convenience functions provided. </font>

it is impossible in C++ to completely faithfully mimic the behavior of a (built-in) reference with a proxy, so don’t try to be subtle about rvalue/lvalue distinctions when using a `vector<bool>`.    

## Algorithms Overview  

The standard-library containers mostly return iterators because when the `STL` was designed, there was no direct support for **move semantics**.  

the `_if` suffix is often used to indicate that an algorithm takes a predicate  

```C++
using Predicate = bool(∗)(int);
void f(vector<Predicate>& v1, vector<int>& v2)
{
  auto p1 = find(v1.begin(),v1.end(),pred); // find element with the value pred
  auto p2 = find_if(v2.begin(),v2.end(),pred); // count elements for which pred() returns true
}
```

## Iterators  Overview  

Iterators are the glue that ties **standard-library algorithms **to their data. Conversely, you can say
that iterators are the mechanism used to minimize an algorithm’s dependence on the data structures  

The iterator categories are concepts rather than classes  
Think of iterators as more general and often better behaved pointers

Use `at()` rather than **iterators** or `[]` when you want range checking  

Use **iterators** and `[]` rather than `at()` when you want to optimize speed  

## Memory and Resources

Think of a **shared pointer** as a structure with two pointers: one to the object and one to the use count:  
**Shared pointers** in a multi-threaded environment can be expensive (because of the need to prevent data races on the use count).  

Minimize the use of **weak_ptrs**

Prefer resource handles with specific semantics to smart pointers  

Prefer **unique_ptr** to **shared_ptr**  

resources that are not pure memory can be leaked by a garbage collector  

## Regular Expressions  

Note that `\i` allows you to express a subpattern in terms of a previous subpattern;

Use `?` to make patterns ‘‘lazy’’

## Concurrency  

The C++ memory model guarantees that two threads of execution can update and access separate memory locations without interfering with each other.   
A memory location is either an object of arithmetic type, a pointer, or a maximal sequence of adjacent bit-fields all having nonzero width

<font color=red>To gain performance, compilers, optimizers, and hardware reorder instructions' order.</font>

Two threads have a data race if both can access a memory location simultaneously and at least one of their accesses is a write. Ways of avoiding data races  

1. Use only a single thread.  
2. Put a lock on every data item that might conceivably be subject to a data race. That can
   eliminate the benefits of concurrency almost as effectively as single threading  
3. Try to look carefully at the code and avoid data races by selectively adding locks. This is most popular approach, but error-prone  
4. Design the code so that threads communicate only through simple **put-and-get-style** interfaces that do not require two threads to directly manipulate a single memory location  
5. Use a higher-level library or tool (e.g., `OpenMP`)

**Lock-free programming** is a set of techniques for writing concurrent programs without using explicit locks  

Primitive operations that do not suffer data races, often called **atomic operations**, can then be used in the implementation of higher-level concurrency mechanisms, such as `locks`, `threads`, and lock-free data structures.  

A **fence**, also known as a **memory barrier**, is an operation that restricts operation reordering according to some specified memory ordering  

There are various historical and technical reasons for the lack of kill, cancel, and interrupt threads

**thread_local storage** becomes important for tasks that require large amounts of non-shared data  

The standard library provides two RAII classes, **lock_guard** and **unique_lock**, to prevent leaks of resource locks
