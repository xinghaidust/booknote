[TOC]

# Effective C++ 55 Specific Ways to Improve Your Programs and Designs

## **View C++ as a federation of languages**  

there are only four primary sublanguages **C**, **Object-Oriented C++**, **Template C++**, **STL**

Rules for effective C++ programming vary, depending on the part of C++ you are using  

For example, 

1. pass-by-value is generally more efficient than pass-by-reference for built-in (i.e., C-like) types, 
2. for Object-Oriented C++, the existence of userdefined constructors and destructors means that pass-by-referenceto-const is usually better.  

## prefer the compiler to the preprocessor  

`#include/#ifdef/#ifndef` remains essential, but `#define` have so many drawbacks,  

✦ For simple constants, prefer const objects or enums to #defines.
✦ For function-like macros, prefer inline functions to #defines  

prefer consts, enums, and inlines to #defines, because

1. macro name may not get entered into the symbol table.  
2. use of the constant may yield smaller code than using a #define  
3. function-like macros don’t incur the overhead of a function call  

class-specific constants should be separate defined If you do take the address of a class constant  

## Use const whenever possible

code duplication can be avoided by having the non-const version call the const version if both have essentially identical implementations  

## **Make sure that objects are initialized before used** 
data members of an object are (manual or default if no initializer) initialized before the body of a constructor is entered  

the relative order of initialization of non-local static objects defined in different translation units is undefined.   

## Know what functions C++ silently writes
Compilers may implicitly generate a class’s default constructor, copy constructor, copy assignment operator, and destructor.  

## Explicitly disallow the use of compiler-generated functions you do not want  
## **Declare destructors virtual in polymorphic base classes**  
destructors should not declare virtual because of overhead unless classes are used as base classes and the derived classes are destroyed through pointer/reference to base class.

## **Prevent exceptions from leaving destructors**  

## **Never call virtual functions during construction or destruction**
During base class construction or destruction of a derived class object, the parts of the language using runtime type information (e.g., virtual functions, dynamic_cast and typeid) treat the object as a base class type  

## Have assignment operators return a reference to *this.  
This is only a convention followed by all the built-in types as well as by all the types in standard library  

## Handle assignment to self in operator=  

making operator= **exception-safe** typically renders it self-assignment-safe, too.   

Techniques include 

1. comparing addresses of source and target objects, 
2. careful statement ordering, 
3. copy-and-swap.  

## Copy all parts of an object, including base class parts

if you add a data member to a class, you need to make sure that you update the copying functions, too.  

Don’t try to implement one of the copying functions in terms of the other. Instead, put common functionality in a third function   

## Use objects to manage resources  

## Think carefully about copying behavior in resource-managing classes  
1. Prohibit copying  
2. Reference-count the underlying resource  
3. Copy the underlying resource
4. Transfer ownership of the underlying resource

## Provide access to raw resources in resource-managing classes  

APIs often require access to raw resources, so each RAII class should offer a way to get at the resource it manages via explicit conversion <font color=red>(safer)</font> or implicit conversion <font color=red>(more convenient)</font>.

## Use the same form in corresponding uses of new and delete  
## Store newed objects in smart pointers  

## Make interfaces easy to use correctly and hard to use incorrectly  
consistency in interfaces and behavioral compatibility with built-in types.  

Ways to prevent errors 

1. creating new types, and compiler type check
2. restricting operations on types by qualifier e.g. **const**

## Treat class design as type design  

issues about design effective classes  

1. how new type be created and destroyed  
2. difference between object initialization and assignment  
3. copy constructor  
4. restrictions on legal values  
5. inheritance 
6. type conversions allowed for your new type  
7. what operators and functions should be disallowed  
8. access to the members  

## Prefer pass-by-reference-to-const to pass-by-value  

1. Prefer pass-by-reference-to-const over pass-by-value because more efficient and avoid the slicing problem  
2. pass-by-value is more efficien for built-in types and STL iterator and function object types  

<font color=red>Some compilers treat built-in and user-defined types differently, even if they have the same underlying representation</font>

## Don’t try to return a reference when you must return an object  
## **Declare data members private**  

Declare data members private and encapsulate them with functions 

**protected** is no more encapsulated than **public**  

## **Prefer non-member non-friend functions to member functions**  
Compared with a member function (which can access all the private members of a class.)
a non-member non-friend function (which can access none of these), which has the same functionality, provides more encapsulation, packaging flexibility, and functional extensibility  

## Declare non-member functions when implicit type conversions should apply to all parameters  
## Consider support for a non-throwing swap  

1. Offer a **public swap member function**  
2. Offer a non-member swap which calls **swap member function** in the same namespace as the class or template

## **Postpone variable definitions as long as possible**  
Whenever you define a variable of a type with a constructor or destructor, you incur the cost of construction when control reaches the variable’s definition, and the cost of destruction when the variable goes out of scope.   

## Minimize casting  

use type-safe containers and virtual functions instead of dynamic_casting  

## Avoid returning “handles” to object internals  

<font color=red>handles (references, pointers, or iterators) to object internals may outlive the object it refers to.</font>

```C++
class Rectangle {
public:
  const Point& upperLeft() const { return pData->ulhc; }
  const Point& lowerRight() const { return pData->lrhc; }
...
};
class GUIObject { ... };
const Rectangle boundingBox(const GUIObject& obj);

GUIObject *pgo; // make pgo point to
... // some GUIObject
const Point *pUpperLeft = &(boundingBox(*pgo).upperLeft()); 
// boundingBox(*pgo) has been destroyed and pUpperLeft is invalid
```

## Strive for exception-safe code  

Exception-safe functions offer one of three guarantees:  

1. **basic guarantee** promise that if an exception is thrown, everything in the program remains in a valid state.  
2. **strong guarantee** promise that if an exception is thrown, the state of the program is unchanged.  
3. **nothrow guarantee**: promise never to throw exceptions  

It's hard to offer **nothrow guarantee** by not calling functions that might throw. 
The **strong guarantee** can often be implemented via copy-and-swap, at the cost of efficiency or complexity. So in practice, a function usually offers a guarantee no stronger than the weakest guarantee of the functions it calls  

## Understand the ins and outs of inlining

if an inline function body is very short, the code generated for the function body may be smaller than the code generated for a function call  

if program takes the address of an inline function, compilers will ignore the ***inline*** request even if you never use function pointers  

constructors is not suitable for ***inline*** because compilers will insert code to promise automatic destruction if throw exceptions

most debuggers have trouble with **inline functions**  

## Minimize compilation dependencies between files  
## Make sure public inheritance models “is-a.”  

 common inter-class relationships 

1. is-a
2. has-a
3. is-implemented-in-terms-of  

## Avoid hiding inherited names  

Names in derived classes may hide the same names in base classes  

## Differentiate between inheritance of interface and inheritance of implementation  
1. Pure virtual functions specify inheritance of interface only  
2. Pure virtual functions plus a definition specifies inheritance of interface and default behavior <font color=red>(must be called with the qualified class name)</font>
3. Non-virtual functions specify inheritance of interface plus inheritance of a mandatory implementation.  

## **Consider alternatives to virtual functions** 

alternative designs to virtual functions 

1. **non-virtual interface idiom (NVI idiom)**, a form of the Template Method design pattern that wraps public non-virtual member functions around less accessible (usually private) virtual functions  
2. Replace virtual functions with function objects but the passed non-member function lacks access to the class’s non-public members.  
3. Replace virtual functions in one hierarchy with virtual functions in another hierarchy. This is the conventional implementation of the Strategy design pattern.    

## Never redefine an inherited non-virtual function
## Never redefine a function’s inherited default parameter value  

because default parameter values are statically bound, while virtual functions are dynamically bound for runtime efficiency

It’s right to offer default parameter values for virtual function with the same default value, but this is code duplication and it's better to use alternative designs such as **NVI idiom**

## Model “has-a” or “is-implemented-in-termsof” through composition 

**Composition**, also known as **layering, containment, aggregation, and embedding**, is the relationship between types that arises when objects of one type contain objects of another type. It has two meaning:

1. a has-a relationship if composition occurs between objects in the application domain.
2. an is-implemented-in-terms-of relationship if composition occurs in the implementation domain

## **Use private inheritance judiciously**

Private inheritance means is-implemented-in-terms-of, just as **composition** 

use private inheritance only if 

1. protected members are used or virtual functions redefined. <font color=red>In this case one alternative is to declare a private nested helper class  and composite it </font>
2. empty base optimization (EBO)  is needed for space optimization 
3. Otherwise, use composition

## **Use multiple inheritance judiciously** 

From the viewpoint of correct behavior, public inheritance should always be virtual but virtual inheritance costs in size, speed, and complexity of initialization and assignment  

<font color=red>use virtual base classes only if you must and try to avoid putting data in them. just like JAVA, to ease the costs.</font> One scenario is combining public inheritance from an Interface class with private inheritance from a implementation. 

## Understand implicit interfaces and compile-time polymorphism 

Both classes and templates support interfaces and polymorphism.

1. For classes, interfaces are explicit and centered on function signatures. Polymorphism occurs at runtime through virtual functions.
2. For template parameters, interfaces are implicit and based on valid expressions. Polymorphism occurs during compilation through template instantiation and function overloading resolution. 

## Understand the two meanings of typename 

Use **typename** to identify nested dependent type names 

## **Know how to access names in templatized base classes** 
Because **non-dependent names** are bound at the **point of definition** of the template, in derived class templates, refer to names in base class templates via a “this->” prefix, via using declarations, or via an explicit base class qualification 

## ~~Factor parameter-independent code out of templates~~ 
Templates may cause binary bloats

1. Bloat due to non-type template parameters can often be eliminated by replacing template parameters with function parameters or class data members 
2. Bloat due to type parameters can be reduced by sharing implementations for instantiation types with identical binary representations (use untype pointers e.g. void*). 

## Use member function templates to accept all compatible types
If you declare member templates for copy construction or assignment, you’ll still need to declare the normal copy constructor and copy assignment operator to avoid none of templates instantializtion and compiler generating the copy constructor and copy assignment operator. 

## ~~Define non-member functions inside templates when type conversions are desired~~ 
C++11 has fixed

## Use traits classes for information about types 

## Be aware of template metaprogramming 

## **Understand the behavior of the new-handler**

if a memory allocation attempt fails, the allocation function call **new-handler** function and repeats the previously-failed allocation attempt endlessly until allocation succeed or throw `std::bad_alloc` if **new-handler** is a null pointer

```C++
namespace std {
  typedef void (*new_handler)();
  new_handler set_new_handler(new_handler p) throw();
}
```

**new handler function** must do one of the following: 

1. Make more memory available 
2. Install a different new-handler 
3. Pass the null pointer to set_new_handler 
4. Throw an exception of type bad_alloc or some type derived from bad_alloc 
5. Not return, typically by calling abort or exit 

## Understand when it makes sense to replace new and delete 
the most common reasons: 

1. To detect usage errors 
2. To improve efficiency. Operator news and operator deletes that ship with compilers take a middle-of-the-road strategy reasonably well for everybody, but optimally for nobody.  
3. To collect usage statistics. 

C++ requires that all **operator news** return pointers that are suitably **aligned** for any data type.

## **Adhere to convention when writing new and delete** 
**operator new** conventions:

1. have the right return value. <font color=red>return a legitimate pointer
   even when zero bytes are requested as if they were really
   requests for one byte.</font>
2. call the new-handling function when insufficient memory is available
3. prepared to cope with requests for no memory 
4. handle requests for larger blocks than expected if called by derived class for class-specific versions

**operator delete** conventions:

1. do nothing if passed a pointer that is null.
2. handle blocks that are larger than expected if called by derived class for class-specific versions

## **Write placement delete if you write placement new** 
if no placement operator delete corresponding placement operator new and constructor throw an exception, program will do nothing about dealloc, causing memory leaks

be careful to avoid having class-specific new/delete hide other new/delete (including std versions) outside class scope

## Pay attention to compiler warnings 

## Familiarize yourself with the standard library, including TR1 
## Familiarize yourself with Boost

Boost, which has a uniquely close and influential relationship with the C++ standardization committee, is both a community of C++ developers and a collection of freely downloadable C++ libraries. Its web site is http://boost.org 





# More effective C++ 35 new ways to improve your programs and designs

## Distinguish between pointers and references 

## Prefer C++-style casts 

The most common use of **reinterpret_cast** is to cast between function pointer types. 

## Never treat arrays of base class polymorphically 

## Avoid gratuitous default constructors 

if a class lacks a default constructor, there are restrictions on how you can use that class 

1. the creation of arrays 
2. classes lacking default constructors are ineligible for use with many template-based container classes, but In most cases, careful template design can eliminate the need for a default constructor.  
3. Virtual base classes lacking default constructors are a pain to work with because the arguments for **virtual base class** constructors must be provided by the **most derived class** of the object being constructed 

Though restrictions, meaningless default constructors should be avoided

## Be wary of user-defined conversion functions 

## Distinguish between prefix and postfix forms of increment and decrement operators 

declare `const operator++(int) ` to prevent `i++++` that is `i.operator++(0).operator++(0); `

## Never overload &&, ||, or ,

## Understand the different meanings of new and delete

use **Placement new** because constructor cannot be called directly

## Use destructors to prevent resource leaks 

## **Prevent resource leaks in constructors** 

<font color=red>C++ destroys only fully constructed objects, so exceptions in constructor must be handled manually</font>

To handle an exception from a **constructor initializer**, we must write the ***constructor*** as a **function try block**

## Prevent exceptions from leaving destructors 

if exceptions propagate out of destructors, `terminate()` will be called and destructor won’t run to completion 

## **Understand how throwing an exception differs from passing a parameter or calling a virtual function** 
1. <font color=red>exception objects are always copied before passed to ***catch*** since it is passed out of scope.</font> When caught by value, they are copied twice 
2. only derived-based conversions and conversion from pointer to an untyped pointer `void*` is allowed
3. ***catch*** clauses are examined in the order in which they appear in the source code 

## Catch exceptions by reference 

## ~~Use exception specifications judiciously~~ 

<font color=red>exception specifications force run-time check for exception and waste performance</font>

## **Understand the costs of exception handling**

<font color=red>slower compared to not handling errors, but faster than other error-handling mechanisms such as error code.</font>
<font color=red>some compiler implementation is **zero-performance-cost** by table-drive approach if no exception is thrown at the cost of larger binaries</font>

## Remember the 80-20 rule 

<font color=red>the overall performance of software is almost always determined by a small part of its constituent code </font>

1. 80% of a program’s resources (include runtime/memory) are used by about 20% of the code; 
2. 80% of the disk accesses are performed for about 20% of the code; 
3. 80% of the maintenance effort is devoted to around 20% of the code 

## Consider using lazy evaluation 

## Amortize the cost of expected computations 

**over-eager evaluation**: doing things before asked to do them to lower the average cost 

## ~~Understand the origin of temporary objects~~

## Facilitate the return value optimization 

return value optimization of compilers

## ~~Overload to avoid implicit type conversions~~ 

Overloading to avoid temporaries  

## Consider using assignment versions op= instead of stand-alone op 

## Consider alternative libraries 

## Understand the costs of virtual functions,multiple inheritance, virtual base classes, and RTTI

The real runtime cost of virtual functions has to do with their interaction with inlining. For all practical purposes, <font color=red>virtual functions aren’t ***inlined*** </font>

## Virtualizing constructors and non-member functions

class’s virtual copy constructor `clone()` may just call its real copy constructor.  

## Limiting the number of instantiated objects of a class 

use **static member**

## Requiring or prohibiting heap-based objects 

<font color=red>There’s not only no portable way to determine whether an object is on the heap, there isn’t even a semi-portable way that works most of the time.</font>

preventing objects from being allocated on the heap, three cases:

1. objects that are directly instantiated: <font color=red>delete class-specific **operator new/delete **function</font>
2. objects instantiated as base class parts of derived class objects <font color=red>same as case 1 because **operator new/delete** are inherited</font>
3. objects embedded inside other objects: <font color=red>no ways</font>

## Smart pointers 

## Reference counting 

Copy-on-Write: sharing a value with other objects until we have to write on our own copy of the value 

## Proxy Classes  

## Making functions virtual with respect to multiple dispatch

1. using virtual functions for first parameters and RTTI with type_info switch for second parameters
2. using multiple virtual functions for first parameters and the second parameters as argument
3. emulated virtual function tables  
4. Using Non-Member Collision-Processing Functions  

## Program in the future tense  

## Make non-leaf classes abstract  

## Understand how to combine C++ and C in the same program  
when C++ use C code, rename C main to RealMain and C++ main call it

## Familiarize yourself with the language standard  


# Effective STL 50 Specific Ways to Improve Your Use of the Standard Template Library  

## Choose your containers with care  

**vector** is default 

## Beware the illusion of container-independent code  

containers are not designed to be interchangeable and you can only write container-independent code by encapsulating  

## Make copying cheap and correct for objects in containers  

create containers of pointers instead of containers of objects.  

## Call empty() instead of checking size() against zero  

`size()` takes linear time for such as `list` while `empty()` always takes constant time

## Prefer range member functions to loop single-element

## Be alert for C++'s most vexing parse  

use `{}` initializer is better

```C++
int f(double (d)); // same as int f(double d);

std::ifstream dataFile;
std::list<int> data(std::istream_iterator<int> dataFile);// declare function

class Widget {... }; // assume Widget has a default constructor
Widget w(); // declare function w
```

## remember to delete the pointers before the container of raw pointers is destroyed  
## ~~Never create containers of auto_ptrs~~

containers of `auto_ptr` is not portable, but containers of `unique_ptr` and `shared_ptr` is ok.

## Choose carefully among erasing options  

## **Be aware of allocator conventions and restrictions**

1. **allocators** should have no **non-static data members**.  
2. member function `allocate` are passed the number of objects for which memory is required  
3. Be sure to provide the nested `rebind` template on which standard containers depend.  

## Understand the legitimate uses of custom allocators  

## **Have realistic expectations about the thread safety of STL containers**  

Different elements in the same container can be modified concurrently by different threads, except for the elements of `std::vector<bool>`

## Prefer `vector` and `string` to dynamically allocated arrays  

## Use `reserve()` to avoid unnecessary reallocations  

## Be aware of variations in `string` implementations  

## ~~Know how to pass `vector` and `string` data to legacy APIs~~ 

## **Use "the swap trick" to trim excess capacity**  

reduction in capacity to **shrink-to-fit**

```C++
string(s).swap(s); // do a "shrink-to-fit" on s
```

## **Avoid using `vector<bool>`**

<font color=red>use dynamic `deque<bool>` or fixed `bitset` instead of `vector<bool>`</font>

`vector<bool>` is a pseudo-container that contains not actual booIs, but a packed representation <font color=red>(a typical implementation is a single bit)</font> of bools that is designed to save space.

`vector<bool>::operator[]` returns an **proxy object** that acts like a reference to a bit

`vector<bool>` is a noble experiment that failed and discovered that it was not possible to create proxy-based containers that satisfy all the requirements of STL containers.  

## Understand the difference between equality and equivalence  

**equality** is dependent on `operator==`

 **equivalence** is the sorted order dependent on **comparison function**  

## Specify comparison types for associative containers of pointers  

## Always have comparison functions return false for equal values

comparison functions used to sort associative containers must define a "strict weak ordering" so equal values yield false

## ~~Avoid in-place key modification in set and multiset~~  

## Sorted vectors offer better performance for lookup when used as associative containers

Insertions and erasures are expensive for vectors, but they're cheap for associative containers.  

## map::operator[] is more efficient when update while map::insert is efficient when insert

## ~~Familiarize yourself with the nonstandard hashed containers~~  

## Prefer iterator to cons_iterator, reverse_iterator, and const_reverse_iterator

## **Use `distance()` and `advance()` to convert a container's cons_iterators to iterators**  

whether **cons_iterator** can be `const_cast` to **iterator ** depends on implementations 
but **const_reverse_iterator** to  **reverse_iterator** is always not allowed, they are true types

```C++
using reverse_iterator=std::reverse_iterator<iterator>
using const_reverse_iterator=std::reverse_iterator<const_iterator>
```

you must do this indirectly by using `advance` and `distance`

```C++
advance(iter, distance<const_reverse_iterator>(iter, c_iter));
```

## Understand how to use a reverse_iterator's base() iterator  

To emulate insertion at a position specified by a **reverse_iterator** ri, insert at the position `ri.base()` instead while when erase `ri.base()`) is not the iterator corresponding to ri

## Consider `istreambuf_iterators` for character-by-character input  

**istreambuf_iterator** may be faster than **istream_iterator**

## Make sure destination ranges are big enough  

## Know your sorting options  

## **Follow remove-like algorithms by erase if you really want to remove something** 

`std::remove` overwrite values that are to be "removed" with later values and do not change size of container, returns the iterator for the new logical end of the range 

## Be wary of remove-like algorithms on containers of pointers 

## Note which algorithms expect sorted ranges 

## ~~Implement case-insensitive string comparisons via `mismatch` or `lexicographical_compare~~` 
## Understand the proper implementation of `copy_if` 

## Use `accumulate` or `for_each` to summarize ranges 

## Design functor classes for pass-by-value 

some implementations of some STL algorithms won't even compile if function objects are passed by reference 

Because pass-by-value functor classes should be small and avoid slicing problem 

## Make predicates pure functions whose return value depends **only** on its parameters

because algorithm implementation may copy predicates multiple times

## Make functor classes adaptable 

`notl` `not2`.etc, requires the existence of certain typedefs 

Function objects that provide the necessary typedefs are said to be **adaptable** 

## Understand the reasons for `ptr_fun`, `mem_fun`, and `mem_fun ref` 

## Make sure `less<T>` means `operator< `

## Prefer algorithm calls to hand-written loops 

1. Efficiency  
2. Correctness 
3. Maintainability 

For efficiency, library implementers can

1. take advantage of their knowledge of container implementations to optimize 
2. use computer science algorithms that are more sophisticated 
3. eliminate of redundant computations and **inline** 

## Prefer member functions to algorithms with the same names

## Distinguish among count, find, binary_search, lower_bound, upper_bound, and equal_range 

## Consider function objects instead of functions as algorithm parameters 

a **function object's operator() function** declared ***inline*** either explicitly or implicitly, can be ***inline*** in called algorithm, faster than a pointer to a function 

## Avoid producing writer-only code 

## Always #include the proper headers 

## Learn to decipher STL-related compiler diagnostics 

## Familiarize yourself with STL-related web sites 

1. The SGI STL site, http://www.sgi.com/tech/stl/.
2. The STLport site, http://www.stlport.org/.
3. The Boost site, http://www.boost.org/ 

# Effective Modern C++

## Understand template type deduction 

During **template type deduction**, arguments that are array or function names decay to pointers, unless they’re used to initialize references  

### 1. ParamType is a Reference or Pointer, but not a Universal Reference 

```C++
template<typename T>
void f(T& param); // param is a reference

template<typename T>
void f(const T& param); // param is a reference

template<typename T>
void f(T* param); // param is a pointer
```

### 2. ParamType is a Universal Reference 

```C++
template<typename T>
void f(T&& param); // param is now a universal reference
```

### 3. ParamType is Neither a Pointer nor a Reference 

```C++
template<typename T>
void f(T param); // param is now passed by value
```



## Understand *auto* type deduction  

**auto type deduction** is **template type deduction** except for braced initializers 

an initialization of type ***auto*** always **decay**s 

```C++
auto x = { 11, 23, 9 }; // x's type is std::initializer_list<int>

template<typename T> // template with parameter
void f(T param); // declaration equivalent to
f({ 11, 23, 9 }); // error! can't deduce type for T because std::initializer_list itself is template

template<typename T>
void f(std::initializer_list<T> initList);
f({ 11, 23, 9 }); // T deduced as int, and initList's
// type is std::initializer_list<int>
```

***auto*** in a function return type or a lambda parameter implies **template type deduction**, not **auto type deduction**  

```C++
auto createInitList()
{
  return { 1, 2, 3 }; // error: can't deduce type
} // for { 1, 2, 3 }

std::vector<int> v;

auto resetV = [&v](const auto& newValue) { v = newValue; }; // C++14
resetV({ 1, 2, 3 }); // error! can't deduce type
// for { 1, 2, 3 }
```

## Understand *decltype*  

one primary use for ***decltype*** is declaring function templates where the function’s return type depends on its parameter types  

C++14 supports `decltype(auto)`, which, like ***auto***, deduces a type from its initializer, but it performs the type deduction using the ***decltype*** rules  

## **Know how to view deduced types** 

1. IDE Editors  <font color=red>not reliable  </font>
2. Compiler Diagnostics  
3. Runtime Output  using `typeid` and `std::type_info::name` <font color=red>not reliable  </font>

 **Boost TypeIndex library** (often written as **Boost.TypeIndex)** is designed to succeed  

## Prefer *auto* to explicit type declarations  

`std::function` approach is generally bigger and slower than the ***auto*** approach, and it may yield out-of-memory exceptions, too  

auto, **type inference**,  is an option, not a mandate  

## Use the explicitly typed initializer idiom when *auto* deduces undesired types  
proxy types  such as `std::vector<bool>’s operator[]`

## **Distinguish between () and {} when creating objects ** 
`{}` is uniform initialization:  

1. prohibits implicit narrowing conversions among built-in types  

2. immunity to C++’s most vexing parse.  

   ```C++
   Widget w2(); // most vexing parse! declares a function
   // named w2 that returns a Widget
   Widget w3{}; // calls Widget ctor with no args
   ```

During constructor overload resolution for **braced initializers**, **default ctor** prior to `std::initializer_list` ctor prior to normal cotr

```C++
class Widget {
public:
    Widget(); // default ctor
    Widget(std::initializer_list<int> il); // std::initializer_list ctor
};
Widget w1; // calls default ctor
Widget w2{}; // also calls default ctor
Widget w3(); // most vexing parse! declares a function!
Widget w4({}); // calls std::initializer_list ctor with empty list
```

Choosing between parentheses and braces for object creation inside templates can be challenging, such as `std::make_unique` and `std::make_shared`   These functions resolve internally use parentheses and by documenting this decision as part of their interfaces.  

## Prefer nullptr to 0 and NULL  

type of ***nullptr*** is `std::nullptr_t`, can implicitly convert to all raw pointer types 

template type deduction deduces the “wrong” types for 0 and NULL because of pointer overload

## Prefer alias declarations to typedefs 

alias templates 

## Prefer scoped enums to unscoped enums 

unscoped enumerator names leak into the scope containing enum definition  

Enumerators for unscoped enums implicitly convert to integral types 

## Prefer deleted functions to private undefined ones 
By convention, **deleted functions** are declared ***public***, not private because , C++ checks accessibility before deleted status 

<font color=red>**deleted functions** are taken into account during **overload resolution**. </font>

## Declare overriding functions *override*

## prefer const_iterators to iterators 

in C++98, const_iterators had only halfhearted support.  

In maximally generic code, prefer non-member versions of begin, end, rbegin, etc., over their member function counterparts. 

## Declare functions noexcept if they won’t emit exceptions 
**noexcept function** is more optimizable  than `throw()` one

most functions are **exception-neutral**, which throw no exceptions themselves, but functions they call might emit one 

By default, all memory deallocation functions and all destructors—both user-defined and compiler-generated—are implicitly **noexcept** and no need to declare them **noexcept**. 

C++ permits **noexcept function** calls the **non-noexcept functions**, and compilers generally don’t issue
warnings about it because the **non-noexcept functions** called may be written in C or C++98  style

## Use constexpr whenever possible 

restrictions imposed on implementations of **constexpr functions** differ between C++11 and C++14 

## Make const member functions thread safe  

const member functions may modify mutable data members so not thread safe

Use of std::atomic variables may offer better performance than a mutex  

## Understand special member function generation  
**Member function templates** never suppress generation of special member functions  

1. **Default constructor**: Generated only if the class contains no user-declared constructors.
2. **Destructor**: ***noexcept*** by default and virtual only if a base class destructor is virtual.
3. **Copy constructor**: member-wise copy construction of non-static data members. Generated only if no user-declared copy constructor. Deleted if the class declares a move operation.
   Generation of this function in a class with a user-declared copy assignment operator or destructor is deprecated.
4. **Copy assignment operator**: member-wise copy assignment of non-static data members. Generated only if no user-declared copy assignment operator. Deleted if the class declares a move operation. Generation of this function in a class with a user-declared copy constructor or destructor is deprecated.  
5. **Move constructor** and **move assignment operator**: member-wise moving of non-static data members. Generated only if the class contains no user-declared copy operations, move operations, or destructor.  

## Use `std::unique_ptr` for exclusive-ownership resource management  

## Use `std::shared_ptr` for shared-ownership resource management  
control block contains  reference count, a copy of the custom deleter, and additional data, including weak count

Avoid creating `std::shared_ptrs` from variables of raw pointer type  

`std::shared_ptrs` can’t work with arrays.  

## Use `std::weak_ptr` for `std::shared_ptr`-like pointers that can dangle  

`std::weak_ptrs` are typically created from `std::shared_ptrs` and can’t be dereferenced.

must create a `std::shared_ptr` from the `std::weak_ptr` if want to access

## Prefer `std::make_unique` and `std::make_shared` to direct use of new  

```C++
template<typename T, typename... Ts>
std::unique_ptr<T> make_unique(Ts&&... params)
{
  return std::unique_ptr<T>(new T(std::forward<Ts>(params)...));
}
```

make functions is exception safe

```C++
processWidget(std::shared_ptr<Widget>(new Widget), // potential
              computePriority()); // resource leak!
/*
code execution order may be
1. Perform “new Widget”.
2. Execute computePriority. (produces an exception and memory leak)
3. Run std::shared_ptr constructor
*/
```

## When using the Pimpl Idiom with `std::unique_ptr` , define special member functions in the implementation file  

Pimpl (“pointer to implementation”) Idiom takes advantage of  declaring a pointer to **an incomplete type**  

```C++
class Widget { // in "widget.h"
public:
  Widget();
  …
private:
    struct Impl;
  std::unique_ptr<Impl> pImpl; // use smart pointer
}; // instead of raw pointer

#include "widget.h" // in "widget.cpp"
#include "gadget.h"
#include <string>
#include <vector>
struct Widget::Impl { // as before
  std::string name;
  std::vector<double> data;
  Gadget g1, g2, g3;
};
Widget::Widget() // per Item 21, create
  : pImpl(std::make_unique<Impl>()) // std::unique_ptr
{} // via std::make_uni

#include "widget.h" // user.cpp
Widget w; // compiler error! 
```

because th destructor of Widget, generated by compile in `widget.h`, call **static_assert** to ensure that the raw pointer doesn’t point to an incomplete type when destroy `std::unique_ptr`

```C++
class Widget { // as before, in "widget.h"
public:
  Widget();
  ~Widget(); // declaration only
  …
private: // as before
  struct Impl;
  std::unique_ptr<Impl> pImpl;
};

#include "widget.h" // as before, in "widget.cpp"
#include "gadget.h"
#include <string>
#include <vector>
struct Widget::Impl { // as before, definition of
    std::string name; // Widget::Impl
  std::vector<double> data;
  Gadget g1, g2, g3;
};
Widget::Widget() // as before
: pImpl(std::make_unique<Impl>())
{}
Widget::~Widget() = default; // same effect as above

#include "widget.h" // user.cpp
Widget w; // compiler ok! 
```

<font color=red>while `std::shared_ptr` destructor do not call **static_assert** and not such problem</font>

## Understand `std::move` and `std::forward`  

`std::move` performs an unconditional cast to an rvalue.
`std::forward` casts its argument to an rvalue only if that argument is boundto an rvalue.
Neither `std::move` nor `std::forward` do anything at runtime.  

## Distinguish universal references from rvalue references  

## Use `std::move` on rvalue references, `std::forward on universal` references  
<font color=red>Never apply `std::move` or `std::forward` to local objects because the return value optimization (RVO) will do this thing and do better</font>

## Avoid overloading on universal references  

universal reference overload vacuums up far more argument types than expects  

Perfect-forwarding constructors are especially problematic, because they’re typically better matches than copy constructors for non-const lvalues  

## ~~Familiarize yourself with alternatives to overloading on universal references~~  
### Use Tag dispatch  

### Constraining templates that take universal references  



## Understand reference collapsing  

## In generic code assume that move operations are not present, not cheap, and not used  
In code with known types or support for move semantics, there is no need for assumptions  

## Familiarize yourself with perfect forwarding failure cases  

Perfect forwarding fails when template type deduction fails or when it deduces the wrong type  

1. Braced initializers  

2. 0 or NULL as null pointers  

3. Declaration-only integral **static const** data members which will be **inline**

   To make it portable, simply provide a definition without intializers for the integral static const data member

1. Overloaded function names and template names  

   The way to get a perfect-forwarding function template to accept an overloaded function name or a template name is to manually specify the overload or instantiation you want to have forwarded.  

2. Bitfields  

   the forwarded-to function will always receive a copy of the bitfield’s value  

   ```C++
   // copy bitfield value; see Item 6 for info on init. form
   auto length = static_cast<std::uint16_t>(h.totalLength);
   fwd(length); // forward the copy
   ```




## Avoid default capture modes  

two default capture modes in C++11: 

1. Default by-reference capture can lead to dangling references to a local variable   
2. Default by-value capture is susceptible to dangling pointers (especially ***this***)  

In C++14, a better way to capture a data member is to use generalized lambda capture   

```C++
void Widget::addFilter() const
{
  filters.emplace_back( // C++14:
  [divisor = divisor](int value) // copy divisor to closure
    { return value % divisor == 0; } // use the copy
  );
}
```

## Use init capture to move objects into closures  

C++ 14 introduced a new capture mechanism that’s so flexible called **init capture** also called **generalized lambda capture**

Using an init capture makes it possible for you to specify

1. the **name of a data member** in the closure class generated from the lambda and
2. an expression initializing that data member  

In C++11, emulate **init capture** via hand-written classes or std::bind  

```C++
std::vector<double> data; // object to be moved

auto func = [data = std::move(data)] // C++14 init capture
  { /* uses of data */ }

auto func =
  std::bind( // C++11 emulation
    [](const std::vector<double>& data) // of init capture
    { /* uses of data */ },
    std::move(data)
  )
```

## Use decltype on auto&& parameters to `std::forward` them  
One of the most exciting features of C++14 is **generic lambdas**—lambdas that use ***auto*** in their parameter specifications: `operator()` in the lambda’s closure class is a template.  

```C++
auto f = [](auto x){ return func(normalize(x)); };

class SomeCompilerGeneratedClassName {
public:
  template<typename T> // see Item 3 for
  auto operator()(T x) const // auto return type
  { return func(normalize(x)); }
};

/* perfect forward*/
auto f =
[](auto&& param)
{
  return
    func(normalize(std::forward<decltype(param)>(param)));
};
```

## Prefer lambdas to `std::bind`  

**Lambdas** are more readable, more expressive, and may be more efficient than using `std::bind`

In C++11 only, `std::bind` may be useful for implementing move capture or for binding objects with templatized function call operators.  

```C++
class PolyWidget {
public:
  template<typename T>
  void operator()(const T& param);
};

PolyWidget pw;
auto boundPW = std::bind(pw, _1); // C++11 has to use std::bind
auto boundPW = [pw](const auto& param) // C++14
  { pw(param); };
```

## Prefer task-based programming to thread-based  

`std::thread` API offers no direct way to get return values from asynchronously run functions, and if those functions throw, the program is terminated.  

## Specify `std::launch::async` if asynchronicity is essential  
`std::async` has two standard launch policies  

1. `std::launch::async`: must be run asynchronously, i.e., on a different thread  
2. `std::launch::deferred`: execute synchronously only when **get** or **wait** is called on the future returned by `std::async`

std::async’s default launch policy is either of two, permits to be run either asynchronously or synchronously.  

## Make `std::threads` unjoinable on all paths  

Make `std::threads` unjoinable on all paths by **RAII** since destroy joinable  `std::threads`would cause program execution to be terminated, because the two obvious alternatives were considered worse choices:

1. join-on-destruction can lead to difficult-to-debug performance anomalies.
2. detach-on-destruction can lead to difficult-to-debug undefined behavior  

## Be aware of varying thread handle destructor behavior  
the callee’s result is stored at **shared state**

The **shared state** is typically represented by a heap-based object, but its type, interface, and implementation are not specified by the Standard  

## Consider void futures for one-shot event communication  
## Use `std::atomic` for concurrency, volatile for special memory  
1. `std::atomic` is for data accessed from multiple threads without using mutexes. It’s a tool for writing concurrent software.
2. **volatile** is for memory where reads and writes should not be optimized away. It’s a tool for working with special memory.  

`std::atomic` and **volatile** serve different purposes, they can even be used together  

## Consider pass by value for copyable parameters that are cheap to move and always copied
## Consider emplacement instead of insertion  

**Insertion functions** take objects to be inserted, while
**emplacement functions** take constructor arguments for objects to be inserted so may perform **explicit type conversion condtructor**

```C++
std::vector<std::string> vs; // container of std::string
vs.push_back("xyzzy"); // make two string constructor call
vs.emplace_back("xyzzy"); // make only one string constructor call
```



# C++ Coding Standards  101 Rules, Guidelines, and Best Practices  
