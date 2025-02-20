[TOC]

# Chapter 1  

## header

Headers in C have names of the form name .h
The C++ versions of these headers  remove the .h suffix and precede the name with the letter c
Hence, **cctype** has the same contents as **ctype.h**, but in a form that is appropriate for C++ programs. In particular, the names defined in the **cname** headers are defined inside the **std namespace**, whereas those defined in the .h versions are not.  

c++ programs must declare object before use and often split declaration and definition into header file (.hpp) and source code file (.cpp) for following reason:

1. must declare when two functions call each other
2. source code will recompile every time if written in header. Source code in another file and separate compilation is better

## basic input/output

\<iostream> is header, header files have a suffix of .h/.H/.hpp/.hxx <font color=red>(while standard library headers no suffix) </font> which compilers don't care about but IDEs sometimes do

 two types: istream、ostream
$$
\text{
istream object: cin} \\
\text{outstream\ object }
\left\{
\begin{aligned}
\text{cout} \\
\text{cerr} \\
\text{clog}
\end{aligned}
\right.
$$



```c++
std::cout << "string" << std::endl;
```

input/output operator >>/<<, return left operand

scope operator ::, all stardard library namespace std (avoid name conflict)

std::endl is a special value called a **manipulator**, end the current line and flush the buffer

```c++
std::cin>>v1>>v2 //v1,v2 is int
```

## comment

1. one-line comment: start with //  <font color=red>not within double quotes</font> end with a newline, eg "//" is a string
2. start with /\* end with \*/ <font color=red>not within double quotes</font>, eg "/\*" is a string, and therefore can't nest



## loop

```c++
while (condition)
	statement
	
for (init-statement; condition; expression) // header
	body
// init-statement->test condition->exit loop or execute body->expression
//				 ->test condition->...
```
variable defined in init-statement  exists <font color=red>only inside the ***for***</font> 

## istream as condition

test state of stream is valid or not, that is, not encounter end-of-file<font color=red>(ctrl+z/ctrl+d)</font>  or invalid input
$$
\text{error\ compiler\ dectect\ }
\left\{
\begin{aligned}
&\text{syntax\ errors} \\
&\text{type\ errors} \\
&\text{declarartion\ errors}
\end{aligned}
\right.
$$









# Chapter 2. Variables and Basic Types   

## basic types

### primitive built-in types

built-in types (array) defined directly by the C++ language represent facilities of computer **hardware** , 

while additional types (vector and string ) defined in standard library, of a higher-level nature,  usually are not implement directly by computer hardware
$$
\text{primitive  built-in types}
\left\{
\begin{aligned}
&\text{void\ types} \text{\color{red}  no values, used for return type}\\
&\text{arithmetic\ types} 
	\left\{
	\begin{aligned} 
	&\text{integral\ types} 
		\left\{
		\begin{aligned} 
		&\text{boolean types:  \textbf{bool}} \\
		&\text{character types} 
			\left\{
			\begin{aligned} 
			&\text{char} \text{\color{red} a single machine byte, machine's basic character set }\\
			&\text{wchar\_t} \text{\color{red} machine's largest extended character set}\\
			&\text{char16\_t} \text{\color{red} Unicode characters}\\
			&\text{char32\_t} \text{\color{red} Unicode characters}\\
			\end{aligned}
			\right. \\
		&\text{integer}
			\left\{
			\begin{aligned} 
			&\text{short}\\
			&\text{int}\\
			&\text{long}\\
			&\text{long long}\\
			\end{aligned}
			\right.
		\end{aligned}
		\right. \\
	&\text{floating-point\ types}
		\left\{
		\begin{aligned} 
		&\text{float} \text{\color{red} 1 word (32 bit)}\\
		&\text{double} \text{\color{red} 2 words}\\
		&\text{long double} \text{\color{red} 3 or 4 words}\\
		\end{aligned}
		\right.
	\end{aligned}
	\right.
\end{aligned}
\right.
$$







### Odds and Ends

1. integral types except **bool** maybe signed or unsigned
2. unlike integer, character types have 3 distinct types: **char, signed char, unsigned char**
3. **char** is equivalent to **signed char** or  **unsigned char** depend on compiler, so not used for arithmetic expressions
4. use **unsigned** for non-negative values
5. size enlargen: short--int--long long,  long is often same size as int
6. cost of float and double is negligible, but long double provides considerable cost and usually unnecessary precision



### type conversion: do not mix signed and unsigned



## literals

$$
\text{default literal types} 
    \left\{
    \begin{aligned} 
    &\text{character} 
        \left\{
        \begin{aligned} 
        &\text{character 'a'} \text{\color{red} char }\\
        &\text{string "hello"} \text{\color{red} array of constant chars and compiler append null character '\0' to the end}\\
        &\text{prefix} 
            \left\{
        	\begin{aligned} 
        	&\text{u \color{red} char16\_t}\\
        	&\text{U \color{red} char32\_t}\\
        	&\text{L \color{red} wchar\_t}\\
        	&\text{u8 \color{red} char (only for string) }\\
        	\end{aligned}
        	\right.\\
        \end{aligned}
        \right. \\
    &\text{integer}
        \left\{
        \begin{aligned} 
        &\text{decimal e.g. 99}  \text{\color{red} int--long--long long}\\
        &\text{octal e.g. 023 and hexadecimal e.g. 0xff}  \text{\color{red} int--unsigned int--long--unsigned long--long long--unsigned long long} \\
        &\text{suffix} 
            \left\{
        	\begin{aligned} 
        	&\text{u/U \color{red} unsigned}\\
        	&\text{l/L \color{red} long}\\
        	&\text{ll/LL \color{red} long long}\\
        	\end{aligned}
        	\right.\\
        \end{aligned}
        \right.\\
    &\text{floating-point\ types}
        \left\{
        \begin{aligned} 
        &\text{0./.01/0e0/0E0} \text{\color{red} double}\\
        &\text{suffix} 
            \left\{
        	\begin{aligned} 
        	&\text{f/F \color{red} float}\\
        	&\text{l/L \color{red} long double}\\
        	\end{aligned}
        	\right.\\
        \end{aligned}
        \right.\\
    &\text{boolean type \color{red} true/false (lowercase)} \\
    &\text{pointer type \color{red} nullptr (lowercase)} \\
\end{aligned}
\right.
$$







### Odds and Ends


1. arithmetic literal is non-negative, **minus operator not included in literals**. eg -1u=0-(1u)=2*32-1

2. string literals separate only by space, tab or newline **auto-concatenate **

3. **escape sequence **

   \x[hex digit]+ using prefix when more than 2 hex digit  eg u'\x1234', u8"你好"

​		\ \[oct digit]{1,3} eg ''\1234"="\123""4"





## variable/object

### definition and declaration

A declaration specifies the type and name, a definition is a declaration and also allocates storage and may provide an initial value.  

**initialization** is not only assignment

```c++
type_specifier var_name = inital_value;
type_specifier var_name(inital_value);
// list initialization
// forbid list intialize built-in type when initializer may lead to loss of information for example type conversion
type_specifier var_name = {inital_value};
type_specifier var_name{inital_value};
```

**default initialized** i.e. define a variable without an initializer, depend on type and where it's defined

for built-in type,  

1. variable defined outside any function is **default initialized to zero **
2. variable **inside a function is **uninitialized <font color=red>**except that local static object is initialized before the first time execution passes through the object’s definition.**</font>  

compiler will complain about creating an object of a class that require explicit initialization  but with no initializer 

to declare shared variable across multi-file: just add **extern**  before **type specifier**, but an initializer will override **extern** declaration to be a definition <font color=red>(do this inside function is an error)</font>








### identifier and scope

**identifier**:\[a-zA-Z\_][a-z0-9A-Z_]* but can't use keywords

1. identifier defined outside function not begin with underscore
2. identifier  not contains two consecutive underscore
3. identifier not begins with underscore followed by an uppercase letter
4. variable lowercase
5. class name begins with an uppercase letter

**scope** mostly delimited by curly braces.

**global scope**: defined outside function, names at the global scope are accessible throughout the program  

**nest scope**: inner/outer scope, use ::var to refer var in outer scope (outer scope has no name so left-hand of :: is empty)





### compound type

define variables of  **compound type**:  **base_type declarator** 

declarator gives variables names and types related to base type e.g. **reference, pointer** and **array** 

#### reference and pointer

**reference** is **not an object**, but an alias, **bound** to a object <font color=red>not literal or expression</font> of **base type**, no way to **rebind** or bind to other reference

**reference** must be initialized

```c++
// int is base type, &ref is declarator
int i1=1, &ref=i1;
```

**pointer** can't point to a reference or literal except C-style string literal

in declaration **type modifier & and * **  used to form compound type, while in expression **address-of operator &,  dereference operator *** 

```c++
int ival=42;
int *p = &ival;
```

null pointer can be initialized with **nullptr** (literal) or 0 or **NULL** ( avoid using, preprocessor variable,  defined as  in **cstdlib** header)

**void*** pointer used to deal with memory, not refer to an object





#### type match

**type of reference**  must match type of object referred <font color=red>except two cases</font>:

1. reference to **const** can be initialized from expression that can be converted to the type of the reference 

2. reference to ***const*** can be initialized from any **non-const** object, literal or general expression which can be converted to the type of reference

   therefore bind a **const** reference to **non-const** object to realize **<font color=red>read-only access to non-const object</font> ** 

   ```c++
   // compiler creates an unnamed temporary object when reference to a diferent type object and bound reference to it
   // dval is a diferent type object or literal or expression
   // this is forbidden because of error when change dval via ri
   int temp = dval;
   int &ri = temp;
   // but this is ok because dval can't be changed by adding const
   const int temp = dval;
   const int &ri = temp;
   ```



**type of pointer**  must match type of object pointed <font color=red>except two cases</font>:

1. pointer to ***const*** can be initialized from **corresponding non-const type** object

   therefore pointer to ***const*** realize **<font color=red>read-only access to non-const object</font> ** 

   ````c++
   // same case with pointr to const
   int temp = 1;
   double nums = 1.0;
   const int *cptr = &temp; // ok
   cptr = &num; // error
   ````

2. **derived-to-base class** conversion



### ***const*** qualifier

add **const** before **type specifier** in a definition

```c++
const int bufSize=1;
```

<font color=red>**const**  object must be initialized</font> because **const** type can use only operations not changing the object itself

**const** object is same as **non-const** one except operations changing itself

**const** object is **local** to a file <font color=red>while const var is global in C</font> 

to use **const** object across multiple files, add **extern** on both definition and declarations

#### reference to const

**const** reference means reference to **const**, and **const** reference can't assign to **non-const** reference, but can refer to different type object [see type match section](####type match)

#### pointer to const & const pointer

```c++
// read from right to left
int i=1, *const cur=&i; // const pointer
double const pi = 3.14, *p;
// equivalent as follows
// const double pi = 3.14;
// const double *p; // pointer to const
const double *const ppi=&pi; // const pointer to const
```

#### top-level const & low-level const

top-level const: const pointer

low-level const: pointer to const object

$$
\text{const}
\left\{
\begin{aligned}
&\text{top-level const} &&\text{\color{red} behave same as non-const when copied, declared auto and more, object iteself is \bf const}\\
&  &&\text{\color{red}  no type modifier between {\bf 'const'} and variable name when defined}\\
&\text{low-level const}  &&\text{\color{red} check const qualification (type match) for both objects when copied, only in base type of compound types} \\
&  &&\text{\color{red}  type modifier exists between {\bf 'const'} and object name when defined}\\

\end{aligned}
\right.
$$

#### constexpr variable & literal type

**constant expression**: literal or calling **constexpr** function or **const** object initialized from a constant expression, whose value cannot change and can be evaluated at compile time.

**constexpr** declaration implies variables **top-level const** and must be initialized by **constant expression** at compile time, otherwise raise an error.

```c++
constexpr int sz = size(); // ok only if size is constexpr function, it must be simple enough
```

only "literal type" can be in **constexpr** declaration

```c++
const int *p = nullptr; // p is a pointer to a const int
constexpr int *q = nullptr; // q is a const pointer to int
```





### type alias

type alias can be defined in two ways

```c++
// substitute variable name in variabel definition with type alias
typedef double wages, *p;
using wages = double;
// type alias is a whole
typedef char* pstring;
// can't intepret as const char* cstr
const pstring cstr = 0; // top-level const, cstr is const pointer to char
const pstring *ps; // low-level const, ps is pointer to const pointer to char

using int_array = int[4]; // new style type alias declaration; 
typedef int int_array[4]; // equivalent typedef declaration; 

// format: using alias_name = declaration left after remove variable name
using ptr_to_array = int(*)[10];// a pointer to an array of ten ints
```



### type specifier: ***auto*** 

variable using **auto** type specifier must have an initializer for compiler to deduce the type

<font color=red>a declaration can involve only one base type</font>, so variable in auto declaration must have consistent types

```c++
auto i=0, *p=&i; // base type is int
auto sz=0, pi=3.14; // error, inconsistent deduce types
```

rules for compiler to deduce type from initializer, descending priority

1. reference as initializer, use referred object type for deduction

2. ignore top-level **const** except asking for reference to **auto-deduced** type or using **const auto**, while low-level **const** not

   **Explanation**: *right-hand operand of **assignment operator** is rvalue, except **reference** need lvalue. So can't priori assume **top-level const** or create reference for rvalue, but must preserve **const** for lvalue and must explicitly declare to create reference*. 

   ```c++
   int i=0;
   const int ci = i, &cr = ci;
   auto b = ci; // b is int, top-level const ignored
   auto c = cr; // c is int, const is low-level, but according to first rule: cr is alias for ci
   auto d = &i; // d is int*
   auto e = &ci; // e is const int*
   // use const to add top-level const for deduced type
   const auto f = ci; // f is const int
   auto &g = ci; // g is const int&, reference to auto-deduced type 
   auto &h = 42; // error
   const auto &j = 42; // j is const int&
   ```

   

### type specifier: ***decltype*** 

**decltype** gets type of expression but not initialize with this expression, lets compiler to deduce the operand type but does not evaluate

```c++
decltype(f()) sum = x; // sum has whatever type f returns
```

rules for compiler to deduce type differ subtly from **auto** does because **auto** is **declare+assignment** while **decltype** just **declare but not initialize** 

```c++
decltype(expr)
```

1. if *expr* is a variable,  **decltype** returns the type of that variable, including top-level **const**  and references  
    ```c++
    const int ci = 0, &cj = ci;
    decltype(ci) x = 0; // x has type const int
    decltype(cj) y = x; // y has type const int& and is bound to x
    decltype(cj) z; // error: z is a reference and must be initialized
    ```

2. if *expr* is not a variable, get the type that that expression yields.

    <font color=red>attention</font>  variable wrapped with one or more sets of parentheses,  will be treated as expression  

    ```c++
    // decltype of a parenthesized variable is always a reference
    decltype((i)) d; // error: d is int& and must be initialized
    decltype(i) e; // ok: e is an (uninitialized) int
    ```

    

3. **decltype** returns a reference type for expressions ~~that yield objects~~ that can stand on the left-hand side of the assignment  
   
    ```c++
    // decltype of an expression can be a reference type
    int i = 42, *p = &i, &r = i;
    decltype(r + 0) b; // ok: addition yields an int; b is an (uninitialized) int
    decltype(*p) c; // error: c is int& and must be initialized
    ```



## Understanding Complicated Declarations  

parenthesis from the inside out   

start with variable name. Priority right combination with type modifier

replace &var with temp, var is reference  to type of temp

replace *var with temp, var is pointer to type of temp

replace var[] with temp, var is array of object with type of temp

replace func_name(para_list) with temp, func_name is function whose return type is type of temp

```c++
int *(&arry)[10] = ptrs; // arry is a reference to an array of ten pointers
int (*f1(int))(int*, int); // f1 is function(int), whose return type is pointer to function(int*, int) who return int
```



## define data structure

### structure definition

 it is a bad idea to define an object as part of a class definition  

Each object has its own copy of the class **data members**  

member without an initializer are default initialized  

data members can be initialize by **in-class initializer**  

**in-class initializer** either enclosed inside curly braces (list initialization) or follow an = sign (copy initialization) but <font color=red>not inside parentheses (direct initializaton)</font> .  

```C++
struct class_name {
/*clasee body forms a new scope, and can be empty*/
/*members should have unique name within class*/
};

struct Sales_data {
std::string bookNo;
unsigned units_sold = 0;
double revenue = 0.0;
};
```



### header guards  

Typically, classes are stored in headers whose name derives from the name of the class 

use preprocessor variable and directive ***#ifdef……#endif*** to ensure it is safe even if the header is included multiple times.  

```c++
// thie is a header file
#ifndef HEADER_GUARD
#define HEADER_GUARD
/*header file content*/
#endif
```

#### preprocessor 

preprocessor do **Including files, Conditional compilation, Macro definition and expansion** 

When the preprocessor sees a **#include**, it replaces the **#include** with the contents of the specified header  

**#include <file.h>**:  **file.h** is searched for in the standard compiler include paths. 

**#include "file.h"**:  **file.h** is searched in current source file directory then C compilers and programming environments

```c++
// preprocessor variable names usually all uppercase and must be unique
#define PREPROCESSOR_VARIABLE
// preprocessor variable names do not respect C++ scoping rules
```







# Chapter 3 Strings, Vectors, and Arrays  

## Namespace **using** Declarations  

**using**  declaration lets us use a name from a namespace without qualifying the name with a **"namespace_name::"** prefix.

```c++
using namespace::name;
```

```c++
#include <iostream>
// using declarations for names from the standard library
using std::cin;
using std::cout; using std::endl;
int main(){
cout << "Enter two numbers:" << endl;
return 0;}
```

**<font color=red>code inside headers should not use using declarations</font>**, because unexpected name conflicts might occur in main program





## library type: string

### string defining & initializing

#### direct initialization & copy initialization

```C++
#include <string>
using std::string;

string s1; // default initialization; s1 is the empty string
string s2(s1); // s2 is a copy of s1, direct initialization
string s2 = s1;  // equivalent to s2(s1), but is copy initialization
string s3("hiya"); // s4 is a copy of the string literal, not including the null(\0)
string s3 = "hiya"; // equivalent to s3("hiya"), but is direct initialization
string s4(n, 'c'); // initialize s4 with n copies of 'c', direct initialization
```

For historical reasons, and for compatibility with C, ,**<font color=red>string literals are not standard library string </font> ** 

The string library converts both character literals and character string literals to **string **

```C++
string s1 = "hello", s2 = "world"; // no punctuation in s1 or s2
string s3 = s1 + ", " + s2 + '\n';

// at least one operand to each + operator must be of string type
string s4 = s1 + ", "; // ok: adding a string and a literal
string s5 = "hello" + ", "; // error: no string operand
string s6 = s1 + ", " + "world"; // ok: each + has a string operand
string s7 = "hello" + ", " + s2; // error: can't add string literals
```



### string operations

```C++
ostream << s  // return ostream

/* The string input operator >> reads and discards any leading whitespace (e.g., spaces, newlines, tabs). It then reads
characters until the next whitespace character is encountered */
istream >> s  // return their left-hand operand istream

/* reads the given stream up to and including the first newline and stores what it read——not including the newline——in its string
argument*/
getline(istream, s)  // return istream

s.epmty()  // returns a bool 

/*type size_type is one of these companion types which make it possible to use the library types in a machine-independent manner. size_type is an unsigned type big enough to hold the size of any string*/
// should not use ints in expressions that use size()
s.size()  // returns a string::size_type value

s[n]  // take a string::size_type n return a reference to the char at position n in s
s1 + s2 // return a new string
s1 = s2 // replace characters in s1 with a copy of elements in s2

s1 == s2
s1 != s2
<,<=,>,>=  // case-sensitive, dictionary ordering
```

#### dictionary ordering

1. If the vectors have differing sizes, but the elements that are in common are equal, then the vector with
   fewer elements is less than the one with more elements.

2. If the elements have differing values, then the relationship between the vectors is determined by the
   relationship between the first element that differs 

### range for statement.

<font color=red>The body of a **range for** must not change the size of the sequence over which it is iterating </font> 

```C++
for (declaration : expression)
	statement
// expression is an object of a type that represents a sequence
// declaration declare loop variable

string str("some string");
// print the characters in str one character to a line
for (auto c : str) // for every char in str
	cout << c << endl; // print the current character followed by a newline
	
// define the loop variable as a reference type to change the value of the characters in a string
string s("Hello World!!!");
// convert s to uppercase
for (auto &c : s) // for every char in s (note: c is a reference)
    c = toupper(c); // c is a reference, so the assignment changes the char in s
cout << s << endl;
```



### subscript/index & iterator 

**subscript operator** (the **[ ] operator**) takes a **string::size_type**,  start at zero  

<font color=red>ensure that the subscript is in range  >= 0 and < size() before using a subscript</font>, otherwise the result of subscripting an empty string or using an index outside this range is undefined, causing **buffer overflow errors **   

**tips** : use **decltype** to give index the appropriate type

```C++
// process characters in s until we run out of characters or we hit a whitespace
for (decltype(s.size()) index = 0;index != s.size() && !isspace(s[index]); ++index)
    s[index] = toupper(s[index]); // capitalize the current character
```



**iterators** give us indirect access to an object  

<font color=red> All of the library containers have **iterators** , but only a few of them support the subscript operator. </font> 

loops that use iterators should not add elements to the container to which the iterators refer  

## library vector type

A vector is a **class template**, not a type. 

A vector is a collection of objects, all of which have the same type. 

vectors are containers, can hold objects of most any type , but not including reference that is not object

### template

C++ has both class and function templates  

templates are instructions to the compiler for generating classes or functions. The process called **instantiation  ** 

### vector defining 

earlier versions of C++ used a slightly different syntax to define a vector whose elements are themselves vectors (or another template type)  

```C++
vector<int> ivec; // ivec holds objects of type int
vector<vector<string>> file; // vector whose elements are vectors

/* old-style declaration, a space between the closing angle bracket of the
outer vector and its element type */
vector<vector<string> > file; 
```





### vector initializing

copy initialization accepts only a single initializer

 in-class initializer accepts either copy initialization or curly braces

vectors holding objects of a type that requires  an explicit initializer must be given initial element values;

```C++
vector<T> v1; // vector can hold objects of type T, default initialized to empty container
vector<T> v2(v1); // copy elements of v1 into v2
vector<T> v2 = v1; // equivalent to v2(v1)

// value initialization
vector<T> v3(n, val); // v3 has n elements with value val
vector<T> v4(n); // v4 has n objects of type T, each object is itself default initialized

// List Initializing
vector<T> v5{a,b,c...};
vector<T> v5 = {a,b,c...}; // equivalent to v5{a,b,c...}
```
list initialize  a vector from a list of zero or more initial element values enclosed in curly braces
```C++
vector<string> v1{"a", "an", "the"}; // list initialization
vector<string> v2("a", "an", "the"); // error
```

#### Value Initialization  

```C++
vector<int> ivec(10); // ten elements, each initialized to 0
vector<string> svec(10); // ten elements, each an empty string
vector<int> vi = 10; // error: must use direct initialization to supply a size
```

#### Using an Array to Initialize a vector  

use two pointers marking the range of values in arrays used to initialize, left closed interval, right open interval

```C++
int int_arr[] = {0, 1, 2, 3, 4, 5};
// ivec has six elements; each is a copy of the corresponding element in int_arr
vector<int> ivec(begin(int_arr), end(int_arr));
// copies three elements: int_arr[1], int_arr[2], int_arr[3]
vector<int> subVec(int_arr + 1, int_arr + 4);
```



#### ambiguity for brace and curly brace in initialization

if values inside curly braces mismatch element type and list initialization isn’t possible,  the compiler looks for other explanation to initialize 

```C++
vector<int> v1(10); // v1 has ten elements with value 0
vector<int> v2{10}; // list initialization: v2 has one element with value 10
vector<int> v3(10, 1); // v3 has ten elements with value 1
vector<int> v4{10, 1}; // list initialization: v4 has two elements with values 10 and 1

vector<string> v5{"hi"}; // list initialization: v5 has one element
vector<string> v6("hi"); // error: can't construct a vector from a string literal
vector<string> v7{10}; // list initialization isn’t possible, v7 has ten default-initialized elements
vector<string> v8{10, "hi"}; // list initialization isn’t possible, v8 has ten elements with value "hi"
```

### vector operation

<font color=red>std::vector\<T>::size_type</font> 

```C++
v.empty()
v.size() // return a std::vector<T>::size_type value

v.push_back(t)  // add a element with value t to the end of v 

v[n] // a std::vector<T>::size_type n return a reference to the element at position n

v1 = v2  // replace elements in v1 with a copy of elements in v2
v1 = {a,b,c...} // replace elements in v1 with a copy of elements in list

// vectors are comparable only if the elements in vectors are comparable
v1 == v2
v1 != v2
<,<=,>,>=  // dictionary ordering
```



#### vector Grow Efficiently  

It is usually more efficient to define an empty vector and add elements at run time  than to define a vector of a specific size with different element values. The exception is all the elements with the same value. 



## Introducing Iterators  

a string is not a container type, but string supports many of the container operations  and also have iterators.  

container type defines iterator types named **container_type::iterator** and **container_type::const_iterator** 

```C++
vector<int>::iterator it; // it can read and write vector<int> elements
string::iterator it2; // it2 can read and write characters in a string
vector<int>::const_iterator it3; // it3 can read but not write elements
string::const_iterator it4; // it4 can read but not write characters
```



### get iterators

types that have iterators have members that return iterators.
The **begin/cbegin** member returns an iterator that denotes the first element (or first character)  

The **end/cend** member returns an **off-the-end iterator** or abbreviated as **“the end iterator”**  positioned “one past the end”— a nonexistent element “off the end” — of the associated container (or string). 
If the container is empty, the iterators returned by **begin** and **end** are equal—they are both off-the-end iterators  

```C++
// the compiler determines the type of b and e; 
// b denotes the first element and e denotes one past the last element in v
auto b = v.begin(), e = v.end(); // b and e have the same type
```

If the object is const, then **begin** and **end** return a **const_iterator**;
if the object is not const, they return **iterator**  
**cbegin** and **cend** always return a **const_iterator**  



### iterator operations

```C++
*iter // return reference to the element denoted by iterator iter
iter->mem //  arrow operator (the -> operator), equivalent to (*iter).mem

++iter/iter++  // increments iter to refer to the next elemetn in the container
--iter/iter--  // decrements iter to refer to the previous elemetn in the container
iter + n  // yield an iterator forward n elements
iter - n  // yield an iterator backward n elements
iter += n
iter -= n

iter1 - iter2  // return a signed integral type named container_type::difference_type

/*two iterators are equal if they denote the same element for the same container, neglect out of range*/
iter1 == iter2
iter1 != iter2
```

empty **iterator** can add or subtract an integral constant expression whose value is 0. 
result of subtracting two null iterators is 0  

```C++
std::vector<int> ivec1{};
std::vector<int> ivec2{};
ivec1.begin()-ivec2.begin(); // result is 0
```

 A valid iterator either denotes an element or denotes a position one past the last element in a container  
Dereferencing an invalid iterator or an off-the-end iterator has undefined behavior.  

<font color=red>any operation that changes the size of a container potentially invalidates all iterators into that container </font> 

all of the library containers have iterators that define the == and != operators,  but most do not have the < operator , so use != rather than < for loops

## arrays

Unlike **vector**, **arrays** have fixed size, they sometimes offer better run-time performance  



### array defining & initializing 

**Arrays** are a compound type, and the number of elements in an array is part of the array’s type  

```C++
// dimension must be a constant expression, means dimension must be known at compile time
type_specifier array_name[dimension]
```

By default, the elements in an array are **default initialized**  

Same as **vector**, **arrays** hold objects, not including references, and cannot use **auto**  to deduce the type from a list of initializers.

#### explicit initialization 

<font color=red> illegal to assign one array to another </font> 

```C++
const unsigned sz = 3;
int ia1[sz] = {0,1,2}; // array of three ints with values 0, 1, 2
int a2[] = {0, 1, 2}; // an array of dimension 3
int a3[5] = {0, 1, 2}; // equivalent to a3[] = {0, 1, 2, 0, 0}
string a4[3] = {"hi", "bye"}; // same as a4[] = {"hi", "bye", ""}
int a5[2] = {0,1,2}; // error: too many initializers

int a[] = {0, 1, 2}; // array of three ints
int a2[] = a; // error: cannot initialize one array with another
a2 = a; // error: cannot assign one array to another
```



#### initialize charactr array with string literal
Character arrays have an additional form of initialization from a string literal 
The ending null character along with the characters in the literal is copied into the array 

```C++
char a1[] = {'C', '+', '+'}; // list initialization, no null
char a2[] = {'C', '+', '+', '\0'}; // list initialization, explicit null
char a3[] = "C++"; // null terminator added automatically
const char a4[6] = "Daniel"; // error: no space for the null!
```

#### array-pointer conversion

compiler ordinarily converts the array to a pointer by substituting array_name with a pointer to the first element  

<font color=red>**caution** for the conversion in **auto** declaration for array type</font> 

<font color=red>but this conversion does not happen when we use **decltype** or reference to array </font>

```C++
int ia[] = {0,1,2,3,4,5,6,7,8,9}; // ia is an array of ten ints
auto ia2(ia); // ia2 is an int* that points to the first element in ia
ia2 = 42; // error: ia2 is a pointer, and we can't assign an int to a pointer

// ia3 is an array of ten ints
decltype(ia) ia3 = {0,1,2,3,4,5,6,7,8,9};
ia3 = p; // error: can't assign an int* to an array
ia3[4] = i; // ok: assigns the value of i to an element in ia3
```



### pointers are iterators

arrays are not class types, have no member functions **begin** and **end**

Instead **iterator header** provides **begin** and **end** function

```C++
#include <iterator>
int ia[] = {0,1,2,3,4,5,6,7,8,9}; // ia is an array of ten ints
int *beg = std::begin(ia); // pointer to the first element in ia
int *last = std::end(ia); // pointer one past the last element in ia
```

pointer can use iterator operations, except for different return type

null pointer is empty iterator

```C++
ptr1 - ptr2  // return a machine-specific library signed integral type named ptrdiff_t defined in the cstddef header
```

### array subscript

**index variable** should normally have type **size_t**, a machine-specific unsigned type defined in the **cstddef** header, which is the C++ version of the
**stddef.h** header in C library  
**size_t** guaranteed to be large enough to hold the size of any object in memory.

<font color=red>**built-in subscript operator converted to arithmetic and deference operator of pointer \*(ptr+n) **  , so the index is not an unsigned type, unlike subscripts for library types like vector and string,</font>

```C++
int *p = &ia[2]; // p points to the element indexed by 2
int j = p[1]; // p[1] is equivalent to *(p + 1), is the same element as ia[3]
int k = p[-2]; // p[-2] is the same element as ia[0]
```



### C-style string

**C-style character strings** are not a type, instead, are a convention for how to represent and use character strings :**Strings are stored in character arrays and are null terminated.  **

Character string literals are an instance of C-style character strings

C-style strings should not be used by C++ programs though supported because they are rich source of bugs and root cause of many security problems.

C header **string.h** defines C library string functions, whose C++ version is  **cstring**  header



#### interface to C-style sting

C++ may have to interface to code that uses arrays and/or C-style character strings. The C++ library offers facilities to make the interface easier to manage  

##### Mixing Library strings and C-Style String  

we can use a null-terminated character array anywhere that we can use a string literal:  initialize **string**, assign, +=

**string** has member function named **c_str** that return a **const char*** type pointer to its version of C-Style String that will change with the **string** object



## Multidimensional Arrays: Arrays of Arrays

### initializing

````C++
int ia[3][4] = { // three elements; each element is an array of size 4
{0, 1, 2, 3}, // initializers for the row indexed by 0
{4, 5, 6, 7}, // initializers for the row indexed by 1
{8, 9, 10, 11} // initializers for the row indexed by 2
}
// equivalent initialization without the optional nested braces for each row
int ia[3][4] = {0,1,2,3,4,5,6,7,8,9,10,11};

// explicitly initialize only element 0 in each row
int ia[3][4] = {{ 0 }, { 4 }, { 8 }};
// explicitly initialize row 0; the remaining elements are value initialized
int ix[3][4] = {0, 3, 6, 9};
````

### for range 

To use a multidimensional array in a range for, the loop control variable for all but the innermost array must be references.  

use **auto, decltype or type alias** to simplify pointers to Multidimensional Arrays  

```C++
int ia[3][4] = { // three elements; each element is an array of size 4
{0, 1, 2, 3}, // initializers for the row indexed by 0
{4, 5, 6, 7}, // initializers for the row indexed by 1
{8, 9, 10, 11} // initializers for the row indexed by 2
}
for (const auto &row : ia) // type of row is referemce to const int[4]
    for (auto col : row) // type of col is int
    	cout << col << endl;
    	
for (auto row : ia) // type of row is int*
    for (auto col : row) // error
    	cout << col << endl;
```





# Chapter 4. Expressions & Operators

An expression is composed of operators and operands and yields a result 

An expression with two or more operators is a **compound expression**   

## Expression is either Lvalue or Rvalue 

Lvalue/Rvalue, inherited from C, originally means: lvalues could stand on the left-hand side of an assignment whereas rvalues could not  

In C++, an lvalue expression yields an object or a function. However, some lvalues, such as **const** objects, may not be the left-hand operand of an assignment. Moreover, some expressions yield objects but return them as rvalues, not lvalues. 

an object can use its identity (location in memory) as an rvalue,  and can also use its value (contents) as an rvalue. 

<font color=red>With one exception of move</font>, an lvalue can be use when an rvalue is required, but the reverse can't . 

**Operators require lvalue or rvalue operands and return lvalue or rvalue**

<font color=red>**definition** and **declaration** is not **expression**</font>

### decltype to an Lvalue/Rvalue expression

<font color=red>decltype(expr) where `expr` is not variable, result a reference if  `expr` yields an lvalue </font>

## Overloaded Operator

C++ language defines what the operators mean when applied to built-in and compound types. 

**overloaded operators**: define an alternative meaning, <font color=red> including the type of its operand(s) and the result but not including the number of operands and the precedence and the associativity</font>, to most existing operator symbols when applied to class types.

## built-in meaning of operators


$$
\text{operator} 
    \left\{
    \begin{aligned} 
    &\text{arithmetic operators (+-*/\% and unitary plus/minus)}  
    	\left\{
        \begin{aligned} 
        &\text{\color{red} operand is rvalue}\\
        &\text{\color{red} result is rvalue} \\
        &\text{\color{red} left-associative} \\
        \end{aligned}
        \right.\\
    &\text{relational operators(<,<=,>,>=,==,!=)} 
    	\left\{
        \begin{aligned} 
        &\text{\color{red} operand is rvalue, arithmetic or pointer type}\\
        &\text{\color{red} result is rvalue, bool type} \\
        &\text{\color{red} left-associative} \\
        \end{aligned}
        \right.\\
    &\text{logical operators(\&\&,||,!)} 
    	\left\{
        \begin{aligned} 
        &\text{\color{red} operand is rvalue, any type that can be converted to bool}\\
        &\text{\color{red} result is rvalue, bool type} \\
        &\text{\color{red} left-associative} \\
        \end{aligned}
        \right.\\
    &\text{assignment operator(=) } 
    	\left\{
        \begin{aligned} 
        &\text{\color{red} left-hand operand modifiable lvalue}\\
        &\text{\color{red} right-hand operand is converted to the type of the left if differs}\\
        &\text{\color{red} result is its left-hand operand, which is an lvalue} \\
        &\text{\color{red} right-associative} \\
        \end{aligned}
        \right.\\
    &\text{compound assignment operators(+=,-=,*=,/=,\%=,<<=,>>=,\&=,|=,} \wedge\!\!=) 
    	\left\{
        \begin{aligned} 
        &\text{\color{red} same as assignment operator}\\
        &\text{\color{red} except left-hand operand is evaluated only once}\\
        \end{aligned}
        \right.\\
    &\text{prefix/postfix increment/decrement operators(++,--)} 
        \left\{
        \begin{aligned} 
        &\text{\color{red} operand is lvalue}\\
        &\text{\color{red} result is lvalue for prefix, yields the changed object}\\
        &\text{\color{red} result is rvalue for postfix, yields a copy of original, unchanged value}\\
        &\text{\color{red} prefix version avoids unnecessary work to store the original value} \\
        &\text{\color{red}*pbeg++ is equivalent to *(pbeg++)}\\
        \end{aligned}
        \right.\\
    &\text{member access operators: dot(.)and arrow(->) operator} 
    	\left\{
        \begin{aligned} 
        &\text{\color{red} ptr->mem \color{black} equivalent to \color{red} (*ptr).mem}\\
        &\text{\color{red} arrow operator requires a pointer operand and yields an lvalue}\\
        &\text{\color{red} dot operator yields an lvalue/rvalue if the member is an lvalue/rvalue.}\\
        \end{aligned}
        \right.\\
    &\text{condition operator(cond?expr1:epxr2)} 
    	\left\{
        \begin{aligned} 
        &\text{\color{red} expr1 and expr2 are types that can be converted to a common type}\\
        &\text{\color{red} only one of expr1 or expr2 is evaluated}\\
        &\text{\color{red} result is lvalue if both expressions are lvalues or can convert to a common lvalue type. }\\
        &\text{\color{red} Otherwise the result is an rvalue.}\\
        \end{aligned}
        \right.\\
    &\text{bitwise operator(~,<<,>>,\&,|,}\wedge)
    	\left\{
        \begin{aligned} 
        &\text{\color{red} operands are integral type as a collection of bits or library type named bitset.}\\
        &\text{\color{red} operand is rvalue}\\
        &\text{\color{red} yield an rvalue in built-in meaning}\\
        \end{aligned}
        \right.\\
    &\text{sizeof operator(sizeof(type) or sizeof expr}
    	\left\{
        \begin{aligned} 
        &\text{\color{red} result is a constant expression of type size\_t.}\\
        &\text{\color{red} not evaluate operand and return the size of the type returned by expr}\\
        &\text{\color{red} sizeof a reference type returns the size of referenced type}\\
        &\text{\color{red} sizeof an array does not convert the array to a pointer}\\
        &\text{\color{red} right-associative}\\
        \end{aligned}
        \right.\\
    &\text{comma operator(,}
    	\left\{
        \begin{aligned} 
        &\text{\color{red} evaluate the order from left to right.}\\
        &\text{\color{red} result is the value of its right-hand expression}\\
        \end{aligned}
        \right.\\
\end{aligned}
\right.
$$

#### Odds and Ends

1. Division between integers returns an integer truncated toward zero;
2. By implication, if **m%n** is nonzero, it has the same sign as m.  
3. except for the obscure case of overflows, **(-m)/n == m/(-n) == -(m/n), (-m)%n == m%(-n) == -(m%n)  **
4. **logical AND/OR operator** define the order operands are evaluated known as **short-circuit evaluation**
5. **true/false** is 1/0 when converted to int type
6. <font color=red>If the left-hand operand of **assignment operator** is of a built-in type, the curly-braced initializer list may contain at most one value, and that value must not require a narrowing conversion. </font>
7. If the initializer list is empty, the compiler generates a value-initialized temporary and assigns that value to the left-hand operand.  
8. **bitwise operators** can be uesd on a library type named **bitset**  that represents a flexibly sized collection of bits
9. **<< operator** inserts 0-valued bits on the right, **\>> operator** inserts 0-valued bits to **unsigned** on the left and inserts copies of the sign bit or 0-valued bits to the **signed** on the left. So recommend using **unsigned types** with the bitwise operators.  
10. result of **bitwise shift** negative or more than the number of bits  is undefined
11. **sizeof** a **string** or a **vector** returns only the size of the fixed part of these types; it does not return the size used by the object’s elements  



## Operator Precedence and Associativity

**Precedence** determines how operators are grouped in a compound expression. 

**Associativity** determines how operators at the same precedence level are grouped. Left-Associativity: a/b/c=(a/b)/c 

Parentheses override Precedence and Associativity  

**<font color=red>Precedence and Associativity say nothing about the order in which the operands are evaluated  </font>**

| Associativity | Operator                                           | Function               | Use                    |
| ------------- | -------------------------------------------------- | ---------------------- | ---------------------- |
| L             | ::                                                 | global scope           | ::name                 |
| L             | ::                                                 | class scope            | class::name            |
| L             | ::                                                 | namespace scope        | namespace::name        |
| ------------- | --------                                           | ---------------        | ---------------        |
| L             | .                                                  | member selector        | object.member          |
| L             | ->                                                 | member selector        | pointer->member        |
| L             | []                                                 | subscript              | expr[expr]             |
| L             | ()                                                 | function call          | name(expr_list)        |
| L             | ()                                                 | type construction      | type(expr_list)        |
| ------------- | --------                                           | ---------------        | ---------------        |
| R             | ++                                                 | postfix increment      | lvalue++               |
| R             | --                                                 | postfix decrement      | lvalue--               |
| R             | typeid                                             | type ID                | typeid(type)           |
| R             | typeid                                             | run-time type ID       | typeid(expr)           |
| R             | explicit cast                                      | type conversion        | cast_name\<type>(expr) |
| ------------- | --------                                           | ---------------        | ---------------        |
| R             | ++                                                 | prefix increment       | ++lvalue               |
| R             | --                                                 | prefix decrement       | --lvalue               |
| R             | ~                                                  | bitwise NOT            | ~expr                  |
| R             | !                                                  | logical NOT            | !expr                  |
| R             | -                                                  | unary minus            | -expr                  |
| R             | +                                                  | unary plus             | +expr                  |
| R             | *                                                  | derefence              | *expr                  |
| R             | &                                                  | address-of             | &lvalue                |
| R             | ()                                                 | type conversion        | (type)expr             |
| R             | sizeof                                             | size of object         | sizeof expr            |
| R             | sizeof                                             | size of type           | sizeof(expr)           |
| R             | sizeof...                                          | size of parameter pack | sizeof...( name \)     |
| R             | new                                                | allocate object        | new type               |
| R             | new[]                                              | allocate array         | new type[size]         |
| R             | delete                                             | deallocate object      | delete expr            |
| R             | delete[]                                           | deallocate array       | delete[] expr          |
| R             | noexcept                                           | can expr throw         | noexcept(expr)         |
| ------------- | --------                                           | ---------------        | ---------------        |
| L             | ->*                                                | ptr to member select   | ptr->*ptr_to_member    |
| L             | .*                                                 | ptr to member select   | obj.*ptr_to_member     |
| ------------- | --------                                           | ---------------        | ---------------        |
| L             | *                                                  | multiply               | expr*expr              |
| L             | /                                                  | divide                 | expr/expr              |
| L             | %                                                  | modulo(reminder)       | expr%expr              |
| ------------- | --------                                           | ---------------        | ---------------        |
| L             | +                                                  | add                    | expr+expr              |
| L             | -                                                  | substract              | expr-expr              |
| ------------- | --------                                           | ---------------        | ---------------        |
| L             | <<                                                 | bitwise shift left     | expr<<expr             |
| L             | >>                                                 | bitwise shift right    | expr>>expr             |
| ------------- | --------                                           | ---------------        | ---------------        |
| L             | <                                                  | less than              | expr<expr              |
| L             | <=                                                 | less than or equal     | expr<=expr             |
| L             | >                                                  | greater than           | expr>expr              |
| L             | >=                                                 | greater than or equal  | expr>=expr             |
| ------------- | --------                                           | ---------------        | ---------------        |
| L             | ==                                                 | equality               | expr==expr             |
| L             | !=                                                 | inequality             | expr!=expr             |
| ------------- | --------                                           | ---------------        | ---------------        |
| L             | &                                                  | bitwise AND            | expr&expr              |
| ------------- | --------                                           | ---------------        | ---------------        |
| L             | ^                                                  | bitwise XOR            | expr^expr              |
| ------------- | --------                                           | ---------------        | ---------------        |
| L             | \|                                                 | bitwise OR             | expr\|expr             |
| ------------- | --------                                           | ---------------        | ---------------        |
| L             | &&                                                 | logical AND            | expr&&expr             |
| ------------- | --------                                           | ---------------        | ---------------        |
| L             | \|\|                                               | logical OR             | expr\|\|expr           |
| ------------- | --------                                           | ---------------        | ---------------        |
| R             | ?:                                                 | conditiona;            | expr?expr:expr         |
| ------------- | --------                                           | ---------------        | ---------------        |
| R             | =                                                  | assignment             | lvalue=expr            |
| R             | +=, -=,<br>*=, /=, %=,<br><<=, >>=,<br>&=, \|=, ^= | compound assign        | lvalue+=expr           |
| ------------- | --------                                           | ---------------        | ---------------        |
| R             | throw                                              | throw exception        | throw expr             |
| ------------- | --------                                           | ---------------        | ---------------        |
| L             | ,                                                  | comma                  | expr, expr             |



### Order of Evaluation  

Order of operand evaluation is independent of precedence and associativity  

There are four operators that do guarantee the order in which operands are evaluated: **logical AND (&&) operator, logical OR (||) operator, the conditional (? :) operator, and the comma (,) operator**

## Type Conversion

### When Implicit Conversions Occur  

1. **Integral Promotion**: In **most expression**s, values of integral types smaller than **int** are first promoted to an appropriate larger integral type.
2.  In **condition**, **non-bool** expressions are converted to **bool**.
3.  In **initializations**, the initializer is converted to the type of the variable; 
4. in **assignments**, the right-hand operand is converted to the type of the left-hand.
5. In **arithmetic and relational expressions** with operands of mixed types, the types are converted to a common type.
6. conversions also happen during function calls.  

### arithmetic conversion

**<font color=red>implicit conversions among the arithmetic types are defined to preserve precision</font>**

#### Integral Promotion  

The types **bool, char, signed char, unsigned char, short, unsigned short** and larger char types (**wchar_t, char16_t, and char32_t**)  are promoted to the smallest type of **int, unsigned int, long, unsigned long, long long**, or **unsigned long long** in which all possible values of that character type <font color=red>(treated as unsigned integer)</font> fit.  

#### operands converted to a common integral type  
integral promotions on operands happen first, and then convert to the type of promoted operands whose range of non-negative values is larger, sometime machine-dependent, e.g. long and unsigned int

1. signedness the same , then the operand with the smaller type is converted to the larger type  
2. signedness differs and the type of the unsigned operand is the same as or larger than that of the signed operand, the signed operand is converted to
   unsigned.  
3. signedness differs. If all values in the unsigned type fit in the larger type, then the unsigned operand is converted to the signed type. If the values don’t fit, then the signed operand is converted to the unsigned type  



$$
\text{implicit conversion} 
	\left\{
	\begin{aligned} 
	&\text{Arithmetic Conversion} \\
	&\text{Array to Pointer Conversion} 
		\left\{
		\begin{aligned} 
		&\text{not performed \color{red}when an array is used with decltype} \\
		&\text{\color{red}when an array as the operand of the address-of(\&), sizeof, or typeid  operator}\\
		&\text{\color{red}when initialize a reference to an array} \\
		\end{aligned}
		\right. \\
	&\text{Function to Pointer Conversion}\\
	&\text{Pointer Conversion}
		\left\{
		\begin{aligned} 
		&\text{constant integral value of 0 and the literal {\bf nullptr} converted to any pointer type.} \text{\color{red}}\\
		&\text{a pointer to any non-const type can be converted to {\bf void*}} \text{\color{red}  }\\
		&\text{a pointer to any type can be converted to a {\bf const void*}} \text{\color{red} }\\
		&\text{additional pointer conversion that applies to types related by inheritance} \text{\color{red} }\\

		\end{aligned}
		\right.\\
	&\text{Conversions to bool:\color{red} automatic conversion from arithmetic or pointer types to bool. } \\
	&\text{Conversions to const:\color{red} convert a pointer to a {\bf non-const} type to a pointer to the corresponding {\bf const} type} \\
	&\text{Conversions Defined by Class Types:\color{red}compiler will apply only one of multiple class-type conversion at a time} \\
\end{aligned}
\right.
$$

## explicit conversion

use a **cast** to request an explicit conversion.  

```C++
cast-name<type>(expression);
```


where **type** is the target type of the conversion, and **expression** is the value to be cast. 

If **type** is a reference, then the result is an lvalue. The cast-name may be one of **static_cast, dynamic_cast, const_cast**, and **reinterpret_cast**.   

<font color=red>avoid casts particularly **reinterpret_cast** </font>

### static_cast

Any well-defined type conversion, other than those involving **low-level const**, can be requested using a **static_cast**

### dynamic_cast

suppors the run-time type identification  

### const_cast

A **const_cast** changes only a low-level **const** object to a **non-const** type  

using a **const_cast** in order to write to a **const **object is undefined.  

A **const_cast** is most useful in the context of overloaded function  

```C++
// return a reference to the shorter of two strings
const string &shorterString(const string &s1, const string &s2)
{
  return s1.size() <= s2.size() ? s1 : s2;
}

string &shorterString(string &s1, string &s2)
{
  auto &r = shorterString(const_cast<const string&>(s1),
                          const_cast<const string&>(s2));
  return const_cast<string&>(r);
}
```



### reinterpret_cast  

A **reinterpret_cas**t generally performs a low-level reinterpretation of the bit pattern of its operands  

A reinterpret_cast is inherently machine dependent because how bit pattern of its operands store and interpret is machine-depend 



### old-style cast

the old-style cast does one of **static_cast** or **const_cast** which is legal for type, If neither cast is legal, then does  **reinterpret_cast**  

```C
// explicit cast in early versions of C++
type (expr); // function-style cast notation
(type) expr; // C-language-style cast notation
```



## Value categories

Each C++ [expression](https://en.cppreference.com/w/cpp/language/expressions) (an operator with its operands, a literal, a variable name, etc.) is characterized by two independent properties: **a *[type](https://en.cppreference.com/w/cpp/language/type)* **and **a *value category***. Each expression has some non-reference type, and each expression belongs to exactly one of the three primary value categories: [*prvalue*](https://en.cppreference.com/w/cpp/language/value_category#prvalue), [*xvalue*](https://en.cppreference.com/w/cpp/language/value_category#xvalue), and [*lvalue*](https://en.cppreference.com/w/cpp/language/value_category#lvalue).

In C++11, expressions that:

- have identity and cannot be moved from are called *lvalue* expressions;
- have identity and can be moved from are called *xvalue* expressions;
- do not have identity and can be moved from are called *prvalue* ("pure rvalue") expressions;
- do not have identity and cannot be moved from are not used.

![Taxonomy](D:\计算机与信息类\c&c++\value_category.png)

### Value transformations

Value transformations are conversions that change the [value category](https://en.cppreference.com/w/cpp/language/value_category) of an expression

#### Lvalue-to-rvalue conversion

glvalue of any non-function, non-array complete type can be converted to an prvalue 

#### Temporary materialization

A prvalue of any complete type can be converted to an xvalue of the same type



# Chapter 5. Statements  

## Simple Statement

### expression statement: expression followed by a semicolon  

### null statement: a single semicolon  

### declaration statement  [declares and defines variables](### declaration statement  )

### compound statement (block): treated as a single statement  

**block**: a (possibly empty) sequence of statements and declarations surrounded by a pair of curly braces, <font color=red>not terminated by a semicolon.  </font> 

## Statement Scope & Object Lifetime

**<font color=red>In C++, names have scope, objects have lifetimes.  </font>**

<font color=red>names exist only to help the compiler figure out.  </font>

A block is a scope  Names introduced inside a block are accessible only in that block and in blocks nested inside that block. Names are visible from where they are defined until the end of the (immediately) enclosing block  

variables defined inside the control structure of the **if, switch, while**, and **for** statements, are visible only within that statement and are out of scope after the statement ends.

**<font color=red>name declared without extern (definition includes declaration)  hides the same name in outer scope.  </font>**

**<font color=red>name in the same scope (not including inner scope) can be declared multiple times but defined only once except overload.  </font>** 

variables defined inside a block are **local variables** , divided into two types according to lifetime **automatic objects  **and **local static object **  

**automatic objects : **created when control passes through the variable’s definition, destroyed when control passes through the end of the block in which the variable is defined  

**local static object: **<font color=red>initialized before the first time execution passes through the object’s definition</font>, destroyed when the program terminates, it is value initialized when no explicit initializer

Objects defined outside any function exist throughout the program’s execution  

## Conditional Statements  

```C++
// condition can be an expression or an initialized variable declaration, and must be enclosed in parentheses. 
if (condition)
	statement
// statement just include nearest single statement
if (condition)
	statement1
else
	statement2
```

**dangling *else***: each **else** is matched with the closest preceding unmatched **if**.  <font color=red>use curly brace enclosing the inner **if** in a block to prevent matching the closest **else** </font>   

```C++
// expression may be an initialized variable declaration and is converted to integral type
/* execution starts at matched case label and continues across all the remaining cases or until the program explicitly interrupts it. */
switch (expression) {
	// case label must be integral constant expression
	case expr1:
	// A label can stand alone if followed a statement or another label
	case expr2:
		break;
	// default label are matched when no case label matches
	default:
	// A case/default label must precede a statement or another case/default label.
		; // null statement or an empty block if do nothing
}

```



**case expr** is only label, interpreted as jump to the label, and it's illegal for **switch**-based control flow to jump over initialization

variable can be used though its definition **<font color=red>(only if no any implicit or explicit initialization)</font>** is in scope but not executed because C++ is ***Statically* Typed *Language*** 

```C++
case true:
    // this switch statement is illegal because these initializations might be bypassed
    string file_name; // error: control bypasses an implicitly initialized variable
    int ival = 0; // error: control bypasses an explicitly initialized variable
    int jval; // ok: because jval is not initialized
    break;
case false:
    // ok: jval is in scope but is uninitialized
    jval = next_num(); // ok: assign a value to jval
    if (file_name.empty()) // file_name is in scope but wasn't initialized
    // ...
```





##  Iterative Statements (loop)

### while \& for

**init-statement** may be only a single declaration statement.  

```C++
// condition can be an expression or an initialized variable declaration
// Variables defined in a while condition or while body are created and destroyed on each iteration
while (condition)
	statement
	
// init-statement must be a single declaration statement, an expression statement, or a null statement
/* the visibility of any object defined within the for header is limited to the body of the for loop. */
for (init-statement condition; expression)
	statement
```

### range for

<font color=red>**range for** statement declares a loop variable which reference to element of sequence is assigned to on each iteration.</font> 

In a **range for**, the value of end() is cached. If we add elements to (or remove them from) the sequence, the value of **end** might be invalidated   

```C++
/* expression must represent a sequence, such as a braced initializer list or an object of a type that has begin and end members that return iterators.*/
/* declaration defines a loop variable. It must be possible to convert each element of the sequence to the variable’s type
often use the auto type specifier and a reference type */
for (declaration : expression)
	statement
```

### do-while

A do while ends with a **semicolon** after the parenthesized condition 

```C++
/* do-while loop does not allow variable definitions inside the condition 
because refer before assignment*/

do
	statement
while (condition);
```



## Jump Statements  

A **break** statement terminates the nearest enclosing **while, do while, for**, or **switch** statement .

A **continue** statement terminates the current iteration of the nearest enclosing  **for, while**, or **do-while** loop   and immediately begins the next iteration.   

**goto** statement provides an unconditional jump to a another **labeled statement (any statement that is preceded by an identifier named *label* followed by a colon:)** in the **same function**.  

label identifiers are independent of names used for variables and other identifiers  

```C++
goto label;
label: statement1; // labeled statement; may be the target of a goto
```

**return** statement: returns the value (if any) in the return, and transfers control out of the called function back to the calling function.  

## *try* Blocks and Exception Handling  

A ***throw* expression**, usually followed by a semicolon, raises an exception. 

try block exception classes  

```C++
try 
// variables declared inside a try block are inaccessible outside the block—in particular for the catch clauses.
    {program-statements} 
catch (exception-declaration) // declaration of a (possibly unnamed) object within parentheses
    {handler-statements}  // exception handler
catch (exception-declaration) 
    {handler-statements} 
```

### Exception Handling  Procedure 

The search for a handler reverses the call chain. When an exception is thrown, search matching **catch** in the function that threw the exception first. If no matching **catch** is found, that function terminates and search matching **catch** in that function’s caller until a matching  **catch** is found.

Once the **catch** finishes, execution continues with the statement immediately following the last **catch** clause of the **try** block.  

If no appropriate **catch** is found, execution is transferred to a library function named **terminate**, which is system dependent but is guaranteed to stop further execution of the program.



### Standard Exceptions  

C++ standard library defines several exception classes in four headers:  

1. The ***exception*** header defines the most general kind of exception class named **exception**. It communicates only that an exception occurred but provides no additional information.
2. The ***stdexcept*** header defines several general-purpose exception classes(**except, runtime_error, range_error, overflow_error, underflow_error, logic_error, domain_error, invalid_argument, length_error, out_of _ange)**
3. The ***new*** header defines the **bad_alloc** exception type
4. The ***type_info*** header defines the **bad_cast** exception type  

**exception, bad_alloc**, and **bad_cast** can only be default initialized, not possible to provide an initializer for objects of these exceptions types.  

**The other exception types are opposite**: can only be initialized with either a string or a C-style string, but cannot default initialized

The exception types define only a single operation named **what**, which takes no arguments and returns a **const char*** that points to a C-style character string  to provide some textual description of the exception thrown.

For the types that take a string initializer, the **what** function returns that string. For the other types, the value of the string that **what** returns varies by compiler.  

## exception safe

Programs that properly “clean up” during exception handling are said to be exception safe.   



# Chapter 6. Functions  

A function is a block of code with a name. 

## Function Declaration & Definition

A function definition typically consists of a return type (**return type can not be an array type or a function type, return a pointer to them instead **), a name, a list of zero or more parameters (**parameter names are optional, but no way to use an unnamed parameter**), and a body. In a function declaration (**function prototype**),  a semicolon replaces the function body and no need for parameter names.   

functions should be declared in header files and defined in source files so that **separate compilation **can be used when only part of code changes. The header that declares a function should be included in the source file that defines that function   

```C++
void f1(){ /* ... */ } // implicit void parameter list
void f2(void){ /* ... */ } // explicit void parameter list, for compatibility for C
```

## Argument Passing   

function is a scope, and parameters are **automatic objects** 

A **function call** initializes the function’s parameters from the corresponding arguments, and it transfers control to that function  

**Parameter initialization works the same way as variable initialization**, so the type of each argument must match the corresponding parameter in the same way that the type of any initializer must match the type of the object it initializes. and compiler is free to evaluate the arguments in whatever order it prefers.

### Use Reference to const When Possible  

As in any other initialization, top-level consts are ignored on a parameter 

In C++, generally use reference parameters instead of using pointer in C

use **reference (const when not change)** instead of **passed by value** to operate on objects of a type that cannot be copied (including the IO types)  and to avoid copy large object 

use **plain reference instead of const reference **will limit the type of arguments that can be passed, e.g. **const** type and literals

In C++, we can define several different functions that have the same name.
However, we can do so only if their parameter lists are sufficiently different.
Because top-level const are ignored, we can pass exactly the same types to either version of `fcn`. The second version of `fcn` is an error.   

### Array Parameter

array can't be copy to initialize so cannot pass an array by value, have to use **pointer** or **reference**.

 pass a **reference** argument to get additional return value

**reference** limits the size of array but **pointer/array parameter** not (<font color=red>**array parameter will convert to pointer, but no such conversion for reference parameter**</font>), so use pointer to a **const** type if the function not change element values.  

````C++
// despite appearances, these three declarations of print are equivalent
// each function has a single parameter of type const int*
void print(const int*);
void print(const int[]); // shows the intent that the function takes an array
void print(const int[10]); // dimension for documentation purposes (at best)

// can call f only for an array of exactly ten ints:
f(int (&arr)[10]) // arr is a reference to an array of ten ints
````

three common techniques used to manage **pointer** parameters  

1. using a marker to specify the extent of an array e.g. C-style strings end with a null character,
2. pass pointers to the first and one past the last element in the array, this is used in the standard library,
3. define a second parameter that indicates the size of the array, which is common in C programs and older C++ programs.

### *main*: Handling Command-Line Option  

```C++
int main(int argc, char *argv[]) { ... }
/* argv, is an array of pointers to C-style character strings. The first element in argv points either to the name of the program or to the empty string. Subsequent elements pass the command line arguments. The element just past the last pointer is
guaranteed to be 0. */
// argc, passes the number of strings in that array
```



### Varying Arguments

two primary ways to write a function that takes a varying number of arguments:

1. pass a library type named **initializer_list**if all the arguments have the same type.  
2. write a special kind of function, known as a **variadic template** if the argument types vary
3. special parameter type, **ellipsis**, <font color=red>should be used only in programs that need to interface to C functions. </font>

#### *initializer_list* Parameters  

An **initializer_list** is a library template type defined in the ***initializer_list*** header, that represents an array of **const** values of the specified type.
must enclose the sequence of values in curly braces to pass to an **initializer_list** parameter

```C++
// default initialization, an empty list of the elements of type T
initializer_list<T> lst;
// elements are copies of initializers and const
initializer_list<T> lst{a,b,c...};
// copying or assigning lst does not copy the element but reference to
lst2(lst);
lst2 = lst;
lst.size();
// return a pointer to the first and one past the last element in lst
lst.begin();
lst.end();
```

#### Ellipsis Parameters  

Ellipsis parameters are in C++ to allow programs to interface to C code that uses a C library facility named **varargs**.   

<font color=red>Objects of most class types are not copied properly when passed to an ellipsis parameter</font>. So ellipsis parameters should be used only for types that are common to both C and C++. 
An ellipsis parameter may appear only as the last element in a parameter list and may take either of two forms:  

```C++
// No type checking is done for the arguments that correspond to the ellipsis parameter. 
void foo(parm_list, ...);
// the comma is optional
void foo(parm_list ...); 

void foo(...);
```

## Return Types  

A **return** statement terminates the function that is currently executing and returns control to the point from which the function was called.

In a **void** function, an implicit **return** takes place after the function’s last statement.  

```C++
/*two forms of return statements*/
return; // used only in a function with a return type of void
// for void function, expression must be another function call that returns void, otherwise compile-time error
// the expression value type can be implicitly converted to function return type
return expression;

// function call result is a variable definition
return_type temporary = expression;
```

return value is used to initialize a temporary of the **return type** at the call site, and that temporary is the result <font color=red>(is lvalue if return reference, rvalue for other return types)</font> of the function call.

Values are returned in exactly the same way as variables are initialized, including curly brace list:  for a function that returns a built-in type, a braced list may contain at most one value, which not require a narrowing conversion. If the function returns a class type, then the class itself defines how the initializers are use.

Array cannot copy to initialize, so a function cannot return an array type, return reference/pointer to array instead.

```C++
// return value as list initialization
vector<string> process()
{
    // . . .
    // expected and actual are strings
    if (expected.empty())
    	return {}; // return an empty vector
    else if (expected == actual)
    	return {"functionX", "okay"}; // return list-initialized vector
    else
    	return {"functionX", expected, actual};
}

// function call returns a non-const reference(lvalue) to be assgined
char &get_val(string &str, string::size_type ix)
{
	return str[ix]; // get_val assumes the given index is valid
}
int main()
{
    string s("a value");
    cout << s << endl; // prints a value
    get_val(s, 0) = 'A'; // changes s[0] to A
    cout << s << endl; // prints A value
    return 0;
}
```

It is invalid to return a reference or pointer to a local object. 

***main*** function, as an exception, is allowed to terminate without a return, and the compiler implicitly inserts a return of 0. The value returned from ***main*** is treated as a **status indicator** where usually a zero return indicates success; most other values indicate failure, but is indeed machine-dependent. To make return values machine independent, the **cstdlib** header defines two preprocessor variables **EXIT_FAILURE** indicate failure and **EXIT_SUCCESS** success.

***main*** function may not call itself.

### complicate return type declaration 

understand function with complicate return type declaration [see this](## Understanding Complicated Declarations), ways to simplify such function declaration:

1. use a type alias;

2. use **auto** and a **trailing return type**, which is right-hand part of assignment in **using type alias**

   ```C++
   // fcn takes an int argument and returns a pointer to an array of ten ints
   auto func(int i) -> int(*)[10]
   ```

3. use **decltype**, **decltype** not convert array to pointer.



## Features for Specialized Uses  

<font color=red>Default arguments ordinarily should be specified with the function
declaration in an appropriate header.  </font>

<font color=red>**inline** and **constexpr** functions may be defined multiple times in the program and all the definitions must match exactly, thus normally are defined in headers.  </font>

### Default Arguments  

Arguments in the call are resolved by position. So those least likely to use a default value appear first and those most likely to use a default appear last.  

Parameters that follow a parameter that has a default argument must also have default arguments.  

A call to a function that has default arguments may have fewer arguments than it actually does.  

#### Default Argument Declarations  

it is legal to redeclare a function multiple times, but each parameter can have its default specified only once so can't change once specified. Any subsequent declaration can add a default only for a parameter not previously specified and only if all parameters to the right already have defaults. 

```C++
typedef string::size_type sz; // typedef
string screen(sz, sz, char = ' ');
string screen(sz, sz, char = '*'); // error: cannot change an already declared default value:
string screen(sz = 24, sz = 80, char); // ok: adds default arguments
```

#### Default Argument Initializers  

Default argument can be **any expression** that has a type that is convertible to the type of the parameter.

Names used as default arguments are resolved <font color=red>in the scope of the **function declaration** because default arguments are specified with the function
declaration. The value that those names represent is evaluated at calling time.</font>

```C++
// the declarations of wd, def, and ht must appear outside a function
sz wd = 80;
char def = ' ';
sz ht();
string screen(sz = ht(), sz = wd, char = def);
string window = screen(); // calls screen(ht(), 80, ' ')

void f2()
{
  def = '*'; // changes the value of a default argument
  sz wd = 100; // hides the outer definition of wd but does not change the default
  window = screen(); // calls screen(ht(), 80, '*')
}
```

<font color=red>Spaces may be necessary to avoid a compound assignment token if the parameter name is absent.</font>

```C++
void f1(int*=0);         // Error, '*=' is unexpected here
void g1(const int&=0);   // Error, '&=' is unexpected here
void f2(int* = 0);       // OK
void g2(const int& = 0); // OK
void h(int&&=0);         // OK even without spaces, '&&' is a token here
```



### ***inline*** Functions

Calling a function is apt to be slower than evaluating the equivalent expression because of work such as storing register states, passing parameters and branch.

Putting the keyword **inline** before the function’s return type makes function expanded “in line” at each call, meant to optimize speed of small, straight-line functions that are called frequently.  

The **inline** specification is only a request to the compiler. The compiler may choose to ignore this request.  

### ***constexpr*** Functions  

***constexpr* function:** <font color=red>return type and the type of each parameter must be a [**literal type**](Literal Classes)</font>, and the function body must contain exactly one return statement. 

<font color=red>**constexpr functions** is not required to return a **constant expression**,</font> whether a **constexpr function** returns a **constant expression** is determined by whether argument passed to is **constant expression** or not.  

<font color=red>If **constexpr functions** returns a **constant expression**, they are implicitly **inline**</font>, and the compiler will replace a call to a **constexpr function** with its resulting value at compile time

### Debug Preprocessor Facilities: assert and NDEBUG  

**assert**, defined in the **cassert** header, is a **preprocessor macro**.

**macro names** must be unique within the program and need not be declared as variable names

```C++
assert(expr);
/* evaluates expr and if the expression is false, then assert writes a message and terminates the program. If the expression is true, then assert does nothing. */
```

The behavior of **assert** depends on the status of a preprocessor variable named **NDEBUG**. If **NDEBUG** is defined, assert does nothing. By default, **NDEBUG** is not defined, so **assert** performs a run-time check.

We can “turn off” debugging by providing a **#define** to define **NDEBUG** or command-line option provided by compiler to define preprocessor variables.

names that can be useful in debugging to report additional information in error messages:
**_ _func_ _:** local static array of **const char** defined in every function by compiler that holds the name of the function;
**_ _FILE_ _:** string literal defined by preprocessor containing the name of the file;
**_ _LINE_ _:** integer literal defined by preprocessor containing the current line number;
**_ _TIME_ _:** string literal defined by preprocessor containing the time the file was compiled;
**_ _DATE_ _:** string literal defined by preprocessor containing the date the file was compiled.   

## Overloaded Functions  

Functions with the same name but different parameter lists, appear in the same scope are **overloaded**.

***main*** function may not be overloaded.  

Overloaded functions must differ in the number or the type(s) of their parameters (top-level **const** is indistinguishable from **non-const** one, <font color=red>but low-level **const** is distinguishable from **non-const** one.  </font>). 

<font color=red>In C++, name lookup happens before type checking. Once a name (**declaration**) is found, the compiler ignores uses of that name (**declaration**) in any outer scope.</font>

### Calling an Overloaded Function    

**Function matchin**g (also known as **overload resolution**) is the process by which a particular function call is associated with a specific function from a set of overloaded functions. The compiler determines which function to call by comparing the arguments in the call with the parameters offered by each function in the overload set  

three possible outcomes of **function matching:** 

1. **best match :** compiler finds exactly one function that is a overall best match  

2. **no match :** compiler issues an error message, **no viable function**  

3. **ambiguous call :** an error, more than one **viable functions** and none of the matches is clearly best  

Casts should not be needed to call an overloaded function to best match. The need for a cast suggests that the parameter sets are designed poorly.  

### Function Matching 

1. determine **candidate functions** that are with the same name as the called function and for which a declaration is visible at the point of the call.
2. select **viable functions** from the set of **candidate functions**. **Viable functions** can be called with the same number of arguments in the call <font color=red>(call to function with default arguments may have fewer arguments)</font>, and the type of each argument must match or be convertible to the type of its corresponding parameter.  
3. determine, **argument by argument**, **which viable function** provides the best match for the call. There is an overall best match if there is one and only one function for which
   • The match for each argument is no worse than the match required by any other viable function
   • There is at least one argument for which the match is better than the match provided by any other viable function  

### Argument Type Conversions  Rank

1. An exact match:
   • The argument and parameter types are identical.
   • The argument is converted from an array or function type to the corresponding pointer type. 
   • A top-level **const** is added to or discarded from the argument.

2. Match through a low-level **const** conversion.

3. Match through a promotion.

4. Match through an arithmetic or pointer conversion 

5. Match through a class-type conversion.

```C++
void ff(int);
void ff(short);
ff('a'); // char promotes to int; calls f(int)

void manip(long);
void manip(float);
manip(3.14); // error: ambiguous call

Record lookup(Account&); // function that takes a reference to Account
Record lookup(const Account&); // new function that takes a const reference
const Account a;
Account b;
lookup(a); // calls lookup(const Account&)
lookup(b); // calls lookup(Account&)
```

Casts should not be needed to call an overloaded function. The need for a cast suggests that the parameter sets are designed poorly  

## Pointers to Functions  

A function’s type is determined by its return type and the types of its parameters. The names of function and parameters are not part of its type.  

<font color=red>declare a pointer in place of the function name  </font>

the function name is automatically converted to a pointer when used as value except that **decltype** returns the function type; the automatic conversion to pointer is not done. 

**dereference operator** can be omitted when use a pointer to a function to call the function to which the pointer points.  

```C++
// compares lengths of two strings
bool lengthCompare(const string &, const string &);
// function type is bool(const string&, const string&)

// pf points to a function returning bool that takes two const string references
bool (*pf)(const string &, const string &); // uninitialized

pf = lengthCompare; // pf now points to the function named lengthCompare
pf = &lengthCompare; // equivalent assignment: address-of operator is optiona

bool b1 = pf("hello", "goodbye"); // calls lengthCompare
bool b2 = (*pf)("hello", "goodbye"); // equivalent call
bool b3 = lengthCompare("hello", "goodbye"); // equivalent call
```

<font color=red>No conversion between pointers to one function type and pointers to another function type.</font> However, as usual, we can assign **nullptr** or a zero-valued integer constant expression to a function pointer to indicate that the pointer does not point to any function.  

```C++
string::size_type sumLength(const string&, const string&);
bool cstringCompare(const char*, const char*);
pf = 0; // ok: pf points to no function
pf = sumLength; // error: return type differs
pf = cstringCompare; // error: parameter types differ
pf = lengthCompare; // ok: function and pointer types match exactly
```

### Pointers to Overloaded Functions  

the compiler uses the type of the pointer to determine which overloaded function to use. The type of the pointer must match one of the overloaded functions **exactly**:  

```C++
void ff(int*);
void ff(unsigned int);
void (*pf1)(unsigned int) = ff; // pf1 points to ff(unsigned)

void (*pf2)(int) = ff; // error: no ff with a matching parameter list
double (*pf3)(int*) = ff; // error: return type of ff and pf3 don't match
```

### Function Pointer As Parameter or Return Type

cannot define parameters of function type or return a function type but can use a pointer to function.

<font color=red>unlike what happens to parameters that have function type, the return type is not automatically converted to a pointer type. </font>

```C++
// third parameter is a function type and is automatically treated as a pointer to function
void useBigger(const string &s1, const string &s2,
				bool pf(const string &, const string &));
// equivalent declaration: explicitly define the parameter as a pointer to function
void useBigger(const string &s1, const string &s2,
				bool (*pf)(const string &, const string &))
```

use type alias or **auto trailing return** or **decltype** to simplify.

 



# Chapter 7. Classes  

**forward declaration**: declare a class without body. The type is an **incomplete type**, and can only used to define pointers or references to such types  

```C++
class Vector; // forward declaration
// class definition
class Vector
{
    // ...
    friend Vector operator*(const Matrix&, const Vector&);
};
```

the compiler processes classes in two steps—compile member declarations (including data members and member functions) first, then member function bodies. <font color=red>  So member function definition can locate in different file from class definition.</font>

classes can control what happens when we initialize, copy, assign, or destroy objects of the class type, and the compiler will synthesize them if we do not define these operations.

## Abstract Data Types  

**Abstract Data Types**: A class that uses data abstraction and encapsulation.
The fundamental ideas behind **classes** are **data abstraction** and **encapsulation**.
**Data abstraction**, <font color=red>the ability to define both data and function members </font>, relies on the separation of **interface **(operations that users of the class can execute) and **implementation** (includes the class’ data members, the bodies of the functions, and any internal functions).
**Encapsulation** enforces the separation of a class’ interface and implementation.  

## Member functions

Member functions must be declared inside the class.
Member functions may be defined inside the class itself or outside the class body (name of a member must include the name of the class when defined outside the class)

member functions may be overloaded.

Functions <font color=red>(include member function and non-member function)</font> defined in the class are implicitly ***inline*** while not inline by default if defined outside the class, but can explicitly declare a member function as **inline** as part of its declaration inside the class body or alternatively specify ***inline*** on the function definition outside the class body  

Member functions have implicit parameter named ***this***, which is <font color=red>is a const **pointer**</font> and initialized with the address of the object on which the function was invoked.

Inside a member function, any direct use of a member of the class is assumed to be an implicit reference through this.

```C++
return *this;// return reference to the object on which the function was called
```

### const member functions  

By default, the type of ***this*** is a **const pointer** to the **non-const** version of the class type.

To call a member function on a const object, add **const** after function headers

```c++
std::string isbn() const { return this->bookNo; }
```

return type of **const** member function that returns ***this** should be a reference to **const**  

```C++
// noconst_method() compiler error, const_method need overload
obj.const_method().noconst_method(); 
```

well-designed C++ programs tend to have lots of small member functions and encapsulte it into **const** and **non-const** version  

## Constructors  

**constructor**: one or more special member functions define how objects of its type can be initialized, is run whenever an object of a class type is created.  

**default constructor** is constructor that takes no arguments  

**Constructors** features

1. have the same name as the class  

2. no return type in function header

3. class can have multiple constructors like any other overloaded function

4. <font color=red>constructors may not be declared as **const**, **constness** is not assumed before the constructor completes the object’s initialization</font>

when not explicitly define any constructors, compiler will implicitly define **synthesized default constructor** as **default constructor**, which uses **in-class initializer** otherwise **default-initializer** the member.  If class has any constructor, we must use ***= default*** explicitly to ask the compiler to
synthesize the default constructor’s definition  

```C++
class_name() = default; // explicitly use synthesized default constructor
```
In practice, it is almost always right to provide a default constructor if other constructors are being defined
```C++
Sales_data obj(); // oops! declares a function, not an object
Sales_data obj2; // ok: obj2 is an object, not a function
```



### Constructor Initializer List  

<font color=red>can use direct initialization `()` or list initialization `{}`</font>

specify initial values for one or more data members, <font color=red>will override in-class initializers but should not</font>

you can also assign value in constructor body rather than initialization
The distinction between initialization and assignment is strictly a matter of <font color=red>low-level efficiency.
More importantly some data members (**const** or references) must be initialized.</font>

```C++
Sales_data(const std::string &s): bookNo(s) { }
Sales_data(const std::string &s): bookNo{s} { } // the same as above
Sales_data(const std::string &s, unsigned n, double p):
	bookNo(s), units_sold(n), revenue(p*n) { }
Sales_data(const std::string &s):
	bookNo(s), units_sold(0), revenue(0){ }
```

In constructor initializer list, members are initialized in the order in which they appear in the class definition: The order in which initializers appear in the constructor initializer list does not matter

### Delegating Constructors  
A delegating constructor uses another constructor from its own class to perform its initialization. , the constructor initializer list and function body of the delegated-to constructor are both executed  

```C++
class Sales_data {
public:
  // nondelegating constructor initializes members from corresponding arguments
  Sales_data(std::string s, unsigned cnt, double price):
  bookNo(s), units_sold(cnt), revenue(cnt*price) { }
  // remaining constructors all delegate to another constructor
  Sales_data(): Sales_data("", 0, 0) {}
  Sales_data(std::string s): Sales_data(s, 0,0) {}
  Sales_data(std::istream &is): Sales_data() { read(is, *this); }
  // other members as before
};
```

### Implicit Class-Type Conversions  

A **constructor** that can be called with a single argument <font color=red>(called **converting constructors**)  </font> defines an **implicit conversion** from the constructor’s parameter type to the class type.  

compiler will automatically apply class-type conversion at most once.  

```C++
// error: requires two user-defined conversions:
// (1) convert "9-999-99999-9" to string
// (2) convert that (temporary) string to Sales_data
item.combine("9-999-99999-9");
```

declaring the constructor as ***explicit***  to suppressing implicit conversion defined by constructors. but can still use a ***static_cast*** to perform an explicit, rather than an implicit, conversion.   

<font color=red>***explicit*** keyword is used only on the constructor declaration inside the class, not on a definition outside </font>



### Aggregate Classes  

A class is an aggregate class if  
1. All of its data members are public
2.  It does not define any constructors
3. It has no in-class initializers 
4.  It has no base classes or virtual functions,  

for an aggregate class you can a braced list of member initializers in declaration order of the data members to initialize them.

The list of initializers must not contain more elements than the class has members.  
If the list of initializers has fewer elements than the class has members, the trailing members are value initialized.

### Literal Classes 

1. An aggregate class whose data members are all of literal type.

2. A non-aggregate class, that meets the following restrictions:
   1. The data members all must have literal type.
   2. The class must have at least one **constexpr constructor**.
   3. If a data member has an in-class initializer, the initializer for a member of built-in type must be a constant expression, or if the member has class type, the initializer must use the member’s own constexpr constructor.
   4. The class must use default definition for its destructor

#### constexpr Constructors

Although constructors can’t be **const**, constructors in a **literal class** can be **constexpr functions**,  used to generate objects that are **constexpr**

A **constexpr constructor** must initialize every data member. The initializers must either use a **constexpr constructor** or be a **constant expression**.


## Access Control and Encapsulation  

**access specifiers** for class member, specifies the access level of the succeeding members until the next access specifier or the end of the class body  

1. ***public*** specifier are accessible to all parts of theprogram. 
2. ***private*** specifier are only accessible to the member functions of the class

The only difference between using ***class*** and using ***struct*** to define a class is the default access level. <font color=red>***class*** is ***private***, ***struct*** is ***public***</font>

including a declaration preceded by the keyword ***friend*** allows another class or function to access all the members, including **non-public** members. 

**Friend declarations** may appear only inside a class definition;  they are neither members of the class, nor a general declaration of the function. We must declare the function separately from the friend declaration.  In other word, <font color=red> **friend declaration** affects access but is not a declaration in an ordinary sense</font>

friendship is not transitive  

<font color=red>overloaded functions do not share friendship, class must declare as a friend for every one.</font>

### Defining a Type Member  

class can define its own local names for types with **access specifiers** public or private  

## Data Members  

two classes with exactly the same member list are different types  

<font color=red>data members can be specified to be of a class type only if the class has been defined because compiler needs to know how much storage of data member</font>, so a class cannot have data members of its own type  

### mutable Data Members  

data members declared with keyword  ***mutable*** is never **const** even when it is a member of a **const** object and can be changed by **const** member function

### Initializers for Data Members  

<font color=red>in-class initializers must use either the = form or curly braces of initialization  </font>

<font color=red>direct initialization syntax `()` is prohibited [to avoid parsing problems](http://www.open-std.org/JTC1/SC22/WG21/docs/papers/2008/n2756.htm#konaissue).</font>

```C++
struct S {
    int i(x); // data member with initializer
    // ...
    static int x;
};

struct T {
    int i(x); // member function declaration
    // ...
    typedef int x;
};
```



## Class Scope  

a class is itself a scope  

for member function defined outside its class, must provide the class name as well as the function name, then  the parameter list and the function body, in the scope of the class. 
can refer to other class members without qualification.  But the return type  not in the scope of the class because it is before function name

```C++
// return type is seen before we're in the scope of Window_mgr
Window_mgr::ScreenIndex Window_mgr::addScreen(const Screen &s)
{
  screens.push_back(s);
  return screens.size() - 1;
}
```

Ordinarily, an inner scope can redefine a name from an outer scope. However, <font color=red>in a class, can not redefine the name which is a type from an outer scope </font>

Name inside Member Definitions Lookup  priority

1. look for a declaration of the name inside the member function  
2. look for a declaration inside the class  
3. look for a declaration that is in scope before the member function definition.  

But it is still possible to <font color=red>qualify the member’s name with the name of its class or by using the ***this*** pointer explicitly, and still possible to access outer scope name by using the scope operator. </font>

```C++
// bad practice: names local to member functions shouldn't hide member names
void Screen::dummy_fcn(pos height) {
  cursor = width * this->height; // member height
  // alternative way to indicate the member
  cursor = width * Screen::height; // member height
}

// good practice: don't use a member name for a parameter or other local variable
void Screen::dummy_fcn(pos ht) {
  cursor = width * height; // member height
}
// bad practice: don't hide names that are needed from surrounding scopes
void Screen::dummy_fcn(pos height) {
  cursor = width * ::height;// which height? the global one
}
```

## static Class Members  

access a **static member** 

1. directly through the **scope operator**:  
2. an object, reference, or pointer of the class type to access a static member:  
3. Member functions can use static members directly, without the scope operator:  

More like member functions rather than data member, **static members** of a class exist outside any object.  

The ***static*** keyword is only used with the **declaration** of a static member, inside the class definition, but not with the definition of that static member:

The declaration inside the class body is not a definition and may declare the member to be of [incomplete type](https://en.cppreference.com/w/cpp/language/incomplete_type)

***non-const static*** class **data member** must be **defined outside** and may **not be initialized in the class body**.<font color=red>(just like member function, ***inline*** if defined inside class)</font>
we can **only** provide in-class initializers for static members that have **const** integral type and **must** do so for static members that are **constexprs** of literal type
If an initializer is provided inside the class, the member’s definition must not specify an initial value.

<font color=red>A **constexpr static** data member is implicitly ***inline***, need not to be redeclared at namespace scope</font>



# Chapter 8. Copy Control  

A class controls these operations by defining five special member functions: **copy constructor**, **copy-assignment operator**, **move constructor**, **move-assignment operator**, and **destructor**  

We can explicitly ask the compiler to generate the **synthesized versions** of the copy-control members by defining them as **= default**, and just like any other member function whether ***inline*** or not is decided by where defined

```C++
class Sales_data {
public:
  // copy control; use defaults
  Sales_data() = default; /* implicitly inline */
  Sales_data(const Sales_data&) = default; /* implicitly inline */
  Sales_data& operator=(const Sales_data &); /* defined outside not inline */
  ~Sales_data() = default; /* implicitly inline */
// other members as before
};
/* defined outside not inline */
Sales_data& Sales_data::operator=(const Sales_data&) = default;
```



## Copy Constructor 

the first parameter is a reference to the class type and any additional parameters have default values 

controls how objects of that class are initialized 

```C++
class Foo {
public:
  Foo(); // default constructor
  Foo(const Foo&); // copy constructor
  // ...
};
```

considering circumstances, copy constructor is able to take a reference to **nonconst** but usually use a reference to **const**. The copy constructor is used implicitly in several circumstances so should not be ***explicit***

**synthesized copy constructor** member-wise copies the non-static members of its argument
into the object being created. 

How member is copied is determined by its type:

1. members of built-in type <font color=red>(include array)</font> are copied directly though we cannot directly copy an array.
2. members of class type are copied by using the members’ copy constructor 

### Copy Initialization 

**direct initialization**: ordinary function matching

**copy initialization**: use copy constructor or <font color=red>move constructor</font>

```C++
string dots(10, '.'); // direct initialization
string s(dots); // direct initialization

string s2 = dots; // copy initialization
string null_book = "9-999-99999-9"; // copy initialization
string nines = string(100, '9'); // copy initialization
```

**copy initialization** happens when

1. define variables using an = 
2. pass an object as an argument to a parameter of nonreference type. <font color=red>So copy constructor’s own  parameter must take a reference, otherwise will indefinitely call copy constructor</font>
3. return an object from a function that has a nonreference return type 
4. curly-brace initialize the elements in an array or the members of an aggregate class 

compiler is permitted (but not obligated) to skip the copy/move constructor and create the object directly, rewrite

```C++
string null_book = "9-999-99999-9"; // copy initialization
```
into
```C++
string null_book("9-999-99999-9"); // compiler omits the copy constructor 
```

## Copy-Assignment Operator 

controls how objects of its class are assigned.

Overloaded operators are functions that have the name operator followed by the
symbol for the operator being defined. Hence, the assignment operator is a function
named ***operator=*** 

Some operators, assignment among them, must be defined as member functions. When an operator is a member function, the left-hand operand is bound to the implicit ***this*** parameter. The right-hand operand in a binary operator, such as assignment, is passed as an explicit parameter 

```C++
class Foo {
public:
  Foo& operator=(const Foo&); // assignment operator
  // ...
};
```

two points when write an assignment operator:

1. Assignment operators must work correctly if an object is assigned to itself.
2. Most assignment operators share work with the destructor and copy constructor

```C++
class HasPtr {
  std::string *ps;
  int i;
};
// WRONG way to write an assignment operator!
HasPtr&
HasPtr::operator=(const HasPtr &rhs)
{
  delete ps; // frees the string to which this object points
  // if rhs and *this are the same object, we're copying from deleted memory!
  ps = new string(*(rhs.ps));
  i = rhs.i;
  return *this;
}
```

### Synthesized Copy-Assignment Operator 

just like copy constructor and returns a reference to its left-hand object 

## Destructor 

The destructor is a member function with the name of the class prefixed by a **tilde (~)**. It has **no return value** and **takes no parameters** so it **cannot be overloaded**. There is always only one destructor for a given class 

```C++
class Foo {
public:
~Foo(); // destructor
// ...
};
```

Contrast to constructors, in a destructor, the function body is executed first and then the members are destroyed. Members are destroyed in **reverse order** from the order in which they were initialized. 

Members of class type are destroyed by running the member’s own destructor. The built-in types do not have destructors, so nothing is done to destroy members of built-in type <font color=red>(include built-in pointer type)</font>

The destructor is used automatically whenever an object of its type is destroyed 

1. Variables are destroyed when they go out of scope.
2. Members of an object are destroyed when the object of which they are a part is destroyed.
3. Elements in a container—whether a library container or an array—are destroyed when the container is destroyed.
4. Dynamically allocated objects are destroyed when the delete operator is applied to a pointer to the object.
5. Temporary objects are destroyed at the end of the full expression in which the temporary was created 

### Synthesized Destructor 

the synthesized destructor has an empty function body and the members are automatically destroyed after the (empty) destructor body is run 

## The Rule of Three/Five 

1. If the class needs a destructor, it almost surely needs a copy constructor and copy assignment operator as well. <font color=red>Except for ***virtual*** destructor of base class</font>
2. If a class needs a copy constructor, it almost surely needs a copy-assignment operator. And vice versa 
3. Ordinarily, if a class defines any of five copy-control members, it usually should define them all

## Preventing Copies 

for some classes, for example, the iostream classes prevent copying to avoid letting multiple objects write to or read from the same IO buffer 

### Defining a Function as Deleted 

prevent copies by defining the copy control member functions as deleted functions

```C++
struct NoCopy {
  NoCopy() = default; // use the synthesized default constructor
  NoCopy(const NoCopy&) = delete; // no copy
  NoCopy &operator=(const NoCopy&) = delete; // no assignment
  ~NoCopy() = default; // use the synthesized destructor
  // other members
};
```

The **= delete **signals to the compiler that we are intentionally not defining these members.

Unlike **= default**, <font color=red>**= delete**  must appear on the first declaration of a deleted function to prohibit operations that attempt to use it. </font>
Unlike **= default** can be used only on member functions that have a synthesized version, **= delete** can be used on any function

#### The Destructor Should Not be a Deleted Member 

If a member has a deleted destructor, then that member cannot be destroyed and compiler will not let us define variables or create temporaries of a type that has a deleted destructor. 
we can dynamically allocate objects with a deleted destructor. However, we cannot free them: 

#### The Copy-Control Members May Be Synthesized as Deleted 

1. The **synthesized destructor** is defined as deleted if the class has a member whose own destructor is deleted or is inaccessible (e.g., ***private***) 
2. The s**ynthesized copy constructor** is defined as deleted if the class has a member whose own copy constructor is deleted or inaccessible. It is also deleted if the class has a member with a deleted or inaccessible destructor. 
3. The **synthesized copy-assignment operato**r is defined as deleted if a member has a deleted or inaccessible copy-assignment operator, or if the class has a **const** or **reference** member. 
4. The **synthesized default constructor** is defined as deleted 
   if the class has a member with a deleted or inaccessible destructor;
   or has a **reference member** that does not have an in-class initializer 
   or has a **const member** whose type does not explicitly define a default constructor and that member does not have an in-class initializer. 

In essence, <font color=red>the copy-control members are synthesized as deleted when it is impossible to copy, assign, or destroy a member of the class. </font>

<font color=red>it is not allowed to create objects that we could not
destroy </font> so a member that has a deleted or inaccessible destructor causes the synthesized default and copy constructors to be defined as deleted 

### private Copy Control 

Prior to the new standard which introduces **= delete**, classes prevented copies by declaring their copy constructor and copy-assignment operator as ***private*** and not defining them to prevent copies by ***friends*** and members because an attempt to use an undefined member results in a link-time failure. 

## Copy Control and Resource Management 

define the copy operations to make the class behave like a value (each object has to have its own copy of the resource that the class manages ) or like a pointer. 

library containers and ***string*** class have valuelike behavior. 
the ***shared_ptr*** class provides pointerlike behavior, 
the IO types and ***unique_ptr*** do not allow copying or assignment, so they provide neither valuelike nor pointerlike behavior. 

pointerlike class can use ***shared_ptr***s to manage the resources in the class or use a ***reference count***

***reference count*** should store in dynamic memory and take care of **self-assignment **

## Swap 

Defining an overloading **swap** function is optimization for classes that allocate resources  

**swap** functions should call **swap**, not **std::swap** in order to use **type-specific swap** function for a class member 

### copy and swap 

use **swap** to define their **assignment operator** are automatically exception safe and correctly handle self-assignment. 

## Moving Objects 

1. moving, rather than copying, the object can provide a significant performance boost. 

2. classes, such as the IO or `unique_ptr` classes, have a resource (such as a pointer or an IO buffer) that may not be shared. Hence, objects of these types can’t be copied but can be moved 

Under the new standard, we can use containers on types that cannot be copied so long as they can be moved. 

calling **std::move** on an object is a dangerous operation, calling only when you are certain that you need to do a move and that the move is guaranteed to be safe when outside of class implementation code such as move constructors or move-assignment operators

### Rvalue References

**rvalue reference** using **&&** rather than **&** to be bound only to an temporary object that is about to be destroyed 

1. <font color=red>**rvalue reference** can bind to **prvalue** or **xvalue** such as expressions that require a conversion, to literals, or to expressions that return an rvalue while **lvalue ** **references** cannot.</font>
2. <font color=red>**lvalue reference** can bind to **lvalue**</font>
3. <font color=red>**const lvalue reference** can bind to **prvalue** or **xvalue**.</font>

<font color=red>A variable is an lvalue</font>; we cannot directly bind an **rvalue reference** to a variable even if that variable was defined as an **rvalue reference type**.  

call to **std::move** function that return an rvalue reference is **xvalue**

We can destroy a moved-from object and can assign a new value to it, but we cannot use the value of a moved-from object.  

<font color=red>**rvalue references** and **lvalue references to const** extend the lifetimes of temporary objects bounded, with some [exceptions](https://en.cppreference.com/w/cpp/language/reference_initialization#Lifetime_of_a_temporary)</font>:

1. a temporary bound to a return value of a function in a return statement is not extended: it is destroyed immediately at the end of the return expression.
2. a temporary bound to a reference parameter in a function call exists until the end of the full expression containing that function call
3. a temporary bound to a reference in the initializer used in a new-expression exists until the end of the full expression containing that new-expression, not as long as the initialized object.

<font color=red>In general, the lifetime of a temporary cannot be further extended by "passing it on": a **second reference**, initialized from the reference variable or data member to which the temporary was bound, does not affect its lifetime.</font>

```C++
//  Restrictions on temporary lifetimes
/* case 1 */
const int& foo(const int& fooRef)
{
    return fooRef + 1;
}

const int& numberRef = foo(5);     // numberRef is a dangling reference

struct S
{
    int mi;
    const std::pair<int, int>& mp; // reference member is a dangling reference
};

/* case 2 */
std::ostream& buf_ref = std::ostringstream() << 'a';
                     // the ostringstream temporary was bound to the left operand
                     // of operator<< but its lifetime ended at the semicolon so
                     // the buf_ref is a dangling reference
template<class T> 
const T& foo(const T& in) 
{ 
  return in; 
}

const Xray& ref1 = X(1); // True, the lifetime is extended.

Xray& ref2 = foo(X(2)); // False, the lifetime is not extended, 
                        // ref2 — dangling reference.
std::cout << ref2.mValue; // Undefined behavior

/* case 3 */
S a {1, {2, 3}}; // temporary pair {2, 3} bound to the reference member
                 // a.mp and its lifetime is extended to match 
                 // the lifetime of object a

S* p = new S{1, {2, 3}}; // temporary pair {2, 3} bound to the reference
                         // member p->mp, but its lifetime ended at the semicolon
                         // p->mp is a dangling reference
```



### Rvalue References in function overloading

```C++
#include <iostream>
#include <utility>
 
void f(int& x)
{
    std::cout << "lvalue reference overload f(" << x << ")\n";
}
 
void f(const int& x)
{
    std::cout << "lvalue reference to const overload f(" << x << ")\n";
}
 
void f(int&& x)
{
    std::cout << "rvalue reference overload f(" << x << ")\n";
}
 
int main()
{
    int i = 1;
    const int ci = 2;
 
    f(i);  // calls f(int&)
    f(ci); // calls f(const int&)
    f(3);  // calls f(int&&)
           // would call f(const int&) if f(int&&) overload wasn't provided
    f(std::move(i)); // calls f(int&&)
 
    // rvalue reference variables are lvalues when used in expressions
    int&& x = 1;
    f(x);            // calls f(int& x)
    f(std::move(x)); // calls f(int&& x)
}
```



### Move Constructor and Move Assignment  

**move constructor and move-assignment operator** are similar to the corresponding copy operations, but they “steal” resources from their given object rather than copy them  

move constructor has an initial parameter that is a **rvalue reference** to the class type, any additional parameters must all have default arguments.
In addition to moving resources, the move constructor must ensure that the moved-from object is left in a state such that destroying that object will be harmless  

```C++
StrVec::StrVec(StrVec &&s) noexcept // move won't throw any exceptions
// member initializers take over the resources in sC++ Primer, Fifth Edition
  : elements(s.elements), first_free(s.first_free), cap(s.cap)
{
  // leave s in a state in which it is safe to run the destructor
  s.elements = s.first_free = s.cap = nullptr;
}
```

#### noexcept  

***noexcept*** is a way for us to promise that a function does not throw any exceptions, and must specify ***noexcept*** on both the declaration in the class header and on the definition   

library containers provide guarantees as to what they do if an exception happens. As one example, vector guarantees that if an exception happens when we call push_back, the vector itself will be left unchanged. So to avoid this potential problem, <font color=red>vector will use a **copy constructor** instead of a **move constructor** during reallocation unless it knows that the element type’s move constructor cannot throw an exception </font>

#### A Moved-from Object Must Be Destructible  

After a move operation, the “moved-from” object must remain a valid, destructible object but users may make no assumptions about its value  

#### Synthesized Move Operations  

The compiler will synthesize a **move constructor** or a **move-assignment operator** <font color=red>only if the class doesn’t define any of its own copy-control members （include copy
constructor, copy-assignment operator, and destructor） </font>and if every **nonstatic** data member of the class can be moved. The compiler can move members of built-in type. It can also move members of a class type if the member’s class has the corresponding move operation:  

Unlike the copy operations, a move operation is never implicitly defined as a deleted function. 
The move operation will be defined as deleted, if we explicitly ask the compiler to generate a move operation by using = default but the compiler is unable to move all the members, more detailed:

1. for **move constructor**, if the class has a member that defines its own copy constructor but does not also define a move constructor, or if the class has a member that doesn’t define its own copy operations and for which the compiler is unable to synthesize a move constructor. Similarly for **move-assignment**.  
2. if the class has a member whose own **move constructor or move-assignment
   operator** is deleted or inaccessible  
3. the **move constructor** is defined as deleted if the destructor is deleted or inaccessible.  
4. the **move-assignment operator **is defined as deleted if the class has a **const** or **reference** member  

#### move operations and the synthesized copy-control members  

If the class defines either a **move constructor and/or a move-assignment operator**, then the **synthesized copy constructor and copy-assignment operator** for that class will be defined as deleted, so must also define their own copy operations  

## Move vs Copy

**rvalue reference** limits **value category**, so **rvalues** will best match **move constructor**, while **lvalues** match **copy constructor**.

If a class has no **move constructor**, function matching can bind **T&&** to **const T&** (while **T&** cannot) so that objects of that type are copied, even if we attempt to move them by calling ***move***.

Member functions other than constructors and assignment can also benefit from providing both copy and move versions, Overloaded functions that distinguish between moving and copying a parameter typically have one version that takes a **const T&** and one that takes a **T&&**  

### Member Function: reference qualifier  

The **reference qualifier** can be either **&** or **&&**, indicating that this may point to an **rvalue** or **lvalue**. Like the **const** qualifier, a **reference qualifier** must appear in both the declaration and definition of the function.

```C++
class Foo {
public:
  Foo &operator=(const Foo&) &; // may assign only to modifiable lvalues
  // other members of Foo
};
Foo &Foo::operator=(const Foo &rhs) &
{
  // do whatever is needed to assign rhs to this object
  return *this;
}
```

 <font color=red>reference
qualifier must follow the const qualifier:  </font>

```C++
class Foo {
public:
  Foo someMem() & const; // error: const qualifier must come first
  Foo anotherMem() const &; // ok: const qualifier comes first
};
```

reference qualifier can limit value category of parameter and return *this of member function

```C++
Foo &retFoo(); // returns a reference; a call to retFoo is an lvalue
Foo retVal(); // returns by value; a call to retVal is an rvalue
Foo i, j; // i and j are lvalues
i = j; // ok: i is an lvalue
retFoo() = j; // ok: retFoo() returns an lvalue
retVal() = j; // error: retVal() returns an rvalue
i = retVal(); // ok: we can pass an rvalue as the right-hand operand to assignment
```

we can define two versions that differ only in that one is **const** qualified and the other is not 
<font color=red>but if a member function has a reference qualifier, all the overload versions of that member **with the same parameter list** must have reference qualifiers. </font>

```C++
class Foo {
public:
  Foo sorted() &&;
  Foo sorted() const; // error: must have reference qualifier
  // Comp is type alias for the function type (see § 6.7 (p. 249))
  // that can be used to compare int values
  using Comp = bool(const int&, const int&);
  Foo sorted(Comp*); // ok: different parameter list
  Foo sorted(Comp*) const; // ok: neither version is reference qualified
};
```





# Chapter 9. Overloaded Operations and Conversions  

can only overload existing operators and <font color=red>cannot invent new operator symbols.</font>

overloaded operators are functions with special names: the keyword ***operator*** followed by the symbol for the operator being defined, and has the **same number of parameters as the operator has operands**. 

<font color=red>Except for the overloaded function-call operator, ***operator()***, an overloaded operator may **not have default arguments** </font>

An operator function must either be a member of a class or have at least one parameter of class type, so we are **not able to change the meaning** of an operator when applied to operands of **built-in type**.  

### Operators That Shouldn’t Be Overloaded  

An overloaded operator has the same precedence and associativity, but <font color=red>do not preserve **order of evaluation** and/or **short-circuit evaluation**</font>, so, ordinarily, **logical AND, and logical OR operators** should not be overloaded.  

the **comma**, **address-of** have built-in meaning and should not be overloaded. 

the following operators can't be overloaded

1. scope resolution operator (**::**)
2. member of object (**.**)
3. pointer to member of object (**.***)
4.  conditional operator (**?:**)

### Calling an Overloaded Operator Function Directly  

```C++
// equivalent calls to a nonmember operator function
data1 + data2; // normal expression
operator+(data1, data2); // equivalent function call

data1 += data2; // expression-based ''call''
data1.operator+=(data2); // equivalent call to a member operator function
```

### Choosing Member or Non-member Implementation  

When we define an operator as a member function, then the left-hand operand must be an object of the class of which that operator is a member  

1. The assignment (`=`), subscript (`[]`), call (`())`, and member access arrow (`->`) operators **must** be defined as **members**.
2. The **compound-assignment operators** ordinarily ought to be members, but not required.
3. Operators that change the state of their object or that are closely tied to their given type—such as increment, decrement, and dereference—usually should be members.
4. **Symmetric operators**  such as the arithmetic, equality, relational, and bitwise operator, usually should be defined as ordinary non-member functions <font color=red>include, **static** member function </font>  

## Overloaded Operators

### Input and Output Operators  

IO operators must be **non-member functions** usually ***friends*** to the class because ordinarily the first parameter of an input/output operator is a reference to the stream  

Generally, ***output operators <<*** should print with minimal formatting lets users control the details of their output. They should not print a newline.  

```C++
/*a reference ostream & because we cannot copy an ostream object.*/
/*const Sales_data & avoid copying the argument, and does not change that object */
ostream &operator<<(ostream &os, const Sales_data &item)
{
  os << item.isbn() << " " << item.units_sold << " "
    << item.revenue << " " << item.avg_price();
  return os;
}

istream &operator>>(istream &is, Sales_data &item)
{
  double price; // no need to initialize; we'll read into price before we use it
  is >> item.bookNo >> item.units_sold >> price;
  if (is) // check that the inputs succeeded
    item.revenue = item.units_sold * price;
  else
    item = Sales_data(); // input failed: give the object the default state
  return is;
}
```

 ***Input operators >>*** should decide what to do about **error recovery**, putting the object into a valid state, because the input might fail

1. the stream contains data of an incorrect type  
2. end-of-file or some other error on the input stream.  

 ***Input operators >>*** can set input stream the ***failbit*** to indicate **user-defined input error**, not use ***eofbit*** imply that the file was exhausted, and ***badbit*** indicate that the stream was corrupted  

### Arithmetic and Relational Operators  

Ordinarily, define the arithmetic and relational operators as **non-member functions**  

Classes that define both an arithmetic operator and the related compound assignment ordinarily ought to implement the arithmetic operator by using the compound assignment  

```C++
// assumes that both objects refer to the same book
Sales_data
operator+(const Sales_data &lhs, const Sales_data &rhs)
{
  Sales_data sum = lhs; // copy data members from lhs into sum
  sum += rhs; // add rhs into sum
  return sum;
}
```

***operator==***

1. equality operator should be transitive, meaning that if a == b and b == c , then a == c
2. If a class defines ***operator==***, it should also define ***operator!=***, and one of the equality or inequality operators should delegate the work to the other 

define **relational operators** only if class has unique ordering and should be consistent with ***operator==***

### Assignment Operators  

Assignment operators **must be defined as members**, and compound-assignment operators **should but not required**.

In addition to the copy- and move-assignment operators that assign the same type,  a class can define additional assignment operators that allow other types as the right-hand operand.  

```C++
// member binary operator: left-hand operand is bound to the implicit this pointer
// assumes that both objects refer to the same book
Sales_data& Sales_data::operator+=(const Sales_data &rhs)
{
  units_sold += rhs.units_sold;
  revenue += rhs.revenue;
  return *this;
}
```

### Subscript Operator  

The subscript operator must be a member function.

the subscript operator usually defines both ***const*** and ***non-const*** versions of this operator and **returns a reference to the element**

```C++
class StrVec {
public:
  std::string& operator[](std::size_t n)
    { return elements[n]; }
  const std::string& operator[](std::size_t n) const
    { return elements[n]; }
  // other members as in § 13.5 (p. 526)
private:
  std::string *elements; // pointer to the first element in the array
};
```

### Increment and Decrement Operators  

The increment (`++`) and decrement (`--`) operators are usually defined member function but not required, and most often implemented for iterator classes  

Classes that define increment or decrement operators should define both the **prefix and postfix** versions, To be consistent with the built-in operators  

1. **prefix operators** should return a reference to the incremented or decremented object.  
2. **postfix operators** should return the old (unincremented or undecremented) value, not a reference.  

Normal overloading cannot distinguish between prefix and postfix operator. To solve this problem, the postfix versions take an extra (unused) parameter of type int. When we use a postfix operator, the compiler supplies 0 as the argument for this parameter.

```C++
class StrBlobPtr {
public:
  // increment and decrement
  StrBlobPtr& operator++(); // prefix operators
  StrBlobPtr& operator--(); // prefix operators
  StrBlobPtr operator++(int); // postfix operators
  StrBlobPtr operator--(int); // postfix operators
  // other members as before
};
```
pass a value for the integer argument to directly call the **postfix version**, The value passed usually is ignored but is necessary  
```C++
StrBlobPtr p(a1); // p points to the vector inside a1
p.operator++(0); // call postfix operator++
p.operator++(); // call prefix operator++
```

### Member Access Operators  

**arrow  operators-> ** must be a member. The ***dereference operator\**** usually should be a member but not required

```C++
class StrBlobPtr {
public:
  std::string& operator*() const
    { auto p = check(curr, "dereference past end");
    return (*p)[curr]; // (*p) is the vector to which this object points
    }
  std::string* operator->() const
    { // delegate the real work to the dereference operator
    return & this->operator*();
    }
  // other members as before
};
```

#### Constraints on the Return from Operator Arrow  

The arrow operator will **recursively call arrow operator** until meet its fundamental meaning of member access  

`point->mem` is equivalent to 

```C++
(*point).mem; // point is a built-in pointer type
(point.operator->())->mem; // point is an object of class type
```

The **overloaded arrow operator** must return either a pointer to a class type or an object of a class type that defines its own **operator arrow**.  

### Function-Call Operator  

The **function-call operator must be a member function**. A class may define **multiple versions** of the call operator, each of which must differ as to the number or types of their parameters.  

Objects of classes that define the call operator are referred to as **function objects**, most often used as arguments to the **generic algorithms**  

#### Lambdas Are Function Objects  

Classes generated from a lambda expression have a deleted default constructor, deleted assignment operators, and a default destructor. Whether the class has a defaulted or deleted copy/move constructor depends in the usual ways on the types of the captured data members

#### Library-Defined Function Objects  

C++ has several kinds of callable objects: functions and pointers to functions, lambdas, objects created by bind, and classes that overload the function-call operator, they may share the same **call signature**

define a function table to store “pointers” to callables with  the same **call signature** using a new library type named ***function*** that is defined in the ***functional*** header;   

store a function pointer instead of the name of the function to avoid overloaded function   

## Overloading, Conversions, and Operators  

**non-explicit constructor**: Implicit conversions from the argument’s type to the class type.   

**conversion operators**: Implicit conversions from the class type to other type.  

Converting constructors and conversion operators define **class-type conversions**, also referred to as **user-defined conversions**  

### Order of Implicit Conversions

**Implicit conversion sequence** consists of the following, in this order:

1) zero or one **standard conversion sequence**;
2) zero or one **user-defined conversion**;
3) zero or one **standard conversion sequence** (only if a user-defined conversion is used).

### Conversion Operators  

**Conversion Operators**: a special kind of member function that converts a value of a class type to a value of some other type  

**A conversion function must be a member function**, may **not specify a return type** but return a value of its corresponding type, and **must** have **an empty parameter list**. The function usually should be **const**  

```C++
operator type() const;
```

***type*** is any type <font color=red>(other than void)</font> that can be a function return type. 
**an array or a function type are not permitted.**
pointer types—both data and function pointers—and to reference types are allowed.  

```C++
class SmallInt {
public:
  SmallInt(int i = 0): val(i)
  {
    if (i < 0 || i > 255)
    throw std::out_of_range("Bad SmallInt value");
  }
  operator int() const { return val; }
private:
  std::size_t val;
};

si = 4; // implicitly converts 4 to SmallInt then calls SmallInt::operator=
si + 3; // implicitly converts si to int followed by integer addition
// the double argument is converted to int using the built-in conversion
SmallInt si = 3.14; // calls the SmallInt(int) constructor
// the SmallInt conversion operator converts si to int;
si + 3.14; // that int is converted to double using the built-in conversion
```

#### explicit Conversion Operators  

prevent **user-defined implicit conversion**

```C++
explicit operator type() const;
```

We must call conversion explicitly through a cast, <font color=red>except that the compiler will apply an **explicit conversion** to an expression used as a condition</font>, that is

1. The condition of an ***if***, ***while***, or ***do*** statement
2. The condition expression in a ***for*** statement header
3. An operand to the ***logical NOT (!), OR (||), or AND (&&) operators***
4. The condition expression in a ***conditional (?:) operator***  

#### classes rarely provide conversion operators  

only one common use is to define conversions to **bool** and should be defined as ***explicit***.  

##### Ambiguous Conversions 

ambiguities are easy to generate if a class defines both conversion operators and overloaded operators  

ensure that there is only one way to convert from the class type to the target type  

1. mutual conversion  
2. multiple conversions from or to types that are themselves related by conversions, eg arithmetic types.  

rules

1. Don’t define mutually converting classes
2. Avoid conversions to the built-in arithmetic types. In particular, if you do, then
   1. Do not define overloaded versions of the operators that take arithmetic types.
   2. Do not define a conversion to more than one arithmetic type. Let the
      standard conversions provide conversions to the other arithmetic types.    

### Function Matching and Overloaded Operators  

In a call to an overloaded function, the **rank** of an additional standard conversion (if any) matters only if the viable functions require the same user-defined conversion. If different user-defined conversions are needed, then the call is **ambiguous**  

<font color=red>overloaded operator with an operand of class type, the candidate functions contain both nonmember and member functions, **(operator new/delete is exception)** include </font>

1. ordinary nonmember versions of that operator
2. built-in versions of the operator.
3. if the left-hand operand has class type, the overloaded versions of the operator, if any, defined by that class are also included.  

# Chapter 10. Object-Oriented Programming  

**OOP** (object-oriented programming) are 

1. **data abstraction**, separate interface from implementation 
2. **inheritance**, define classes that model the relationships among similar types
3. **dynamic binding**, happens when a **virtual function** is called through a reference (or a pointer) to a base class.  

## Defining Base and Derived Classes  

```C++
class Quote {
public:
  Quote() = default; // = default see § 7.1.4 (p. 264)
  Quote(const std::string &book, double sales_price):
  bookNo(book), price(sales_price) { }
  std::string isbn() const { return bookNo; }
  // returns the total sales price for the specified number of items
  // derived classes will override and apply different discount algorithms
  virtual double net_price(std::size_t n) const
    { return n * price; }
  virtual ~Quote() = default; // dynamic binding for the destructor
private:
  std::string bookNo; // ISBN number of this item
protected:
  double price = 0.0; // normal, undiscounted price
};

class Bulk_quote : public Quote { // Bulk_quote inherits from Quote
  Bulk_quote() = default;
  Bulk_quote(const std::string&, double, std::size_t, double);
  // overrides the base version in order to implement the bulk purchase     discount policy
  double net_price(std::size_t) const override;
private:
  std::size_t min_qty = 0; // minimum purchase for the discount to apply
  double discount = 0.0; // fractional discount to apply
};
```

If a **base class** defines a **static member**, there is only one such member defined for the entire hierarchy

Like any other class, a derived **class declaration** contains the class name but does not include its derivation list

```C++
class Bulk_quote; // ok
class Bulk_quote : public Quote; // error, not include derivation list
```

A **direct base class** is named in the **derivation list**. An **indirect base** is one that a derived class inherits through its direct base class.  

A class **must be defined**, not just declared, before we can use it **as a base class**. So it is impossible to derive a class from itself.  

```C++
class Quote; // declared but not defined
// error: Quote must be defined
class Bulk_quote : public Quote { ... };
```

### Preventing Inheritance  

prevent a class from being used as a base by following the class name with ***final*** 

```C++
class NoDerived final { /* */ }; // NoDerived can't be a base class
class Base { /* */ };
// Last is final; we cannot inherit from Last
class Last final : Base { /* */ }; // Last can't be a base class
class Bad : NoDerived { /* */ }; // error: NoDerived is final
class Bad2 : Last { /* */ }; // error: Last is final
```

### Derived-to-Base Conversion  

A derived object contains a subobject containing the (nonstatic) members defined in the derived class itself, plus subobjects corresponding to each base class from which the derived class inherits. But the standard does not specify how derived objects are laid out in memory  

#### Static Type and Dynamic Type  

We can bind a pointer or reference to a base-class type to an object of a type derived from that base class, so **static type** <font color=red>(type known
at compile time)</font> of a pointer or reference to a base class may differ from its **dynamic type** <font color=red>(type known
at run time)</font>.  

Only if **call virtual function** through a reference or pointer, **static type **may differ from **dynamic type**

The compiler has no way to know <font color=red>(at compile time)</font> that a specific conversion will be safe at run time. The compiler looks only at the **static types**<font color=red> (can be cast by ***static_cast***) </font> of the pointer or reference to determine whether a conversion is legal. If the base class has one or more virtual functions, we can use a ***dynamic_cast*** to request a conversion that is checked at run time. 

1. conversion from derived to base applies only to **pointer or reference types**.

2. converting from an object of a derived class to its base-class type is realized by base-type construction and conversion from reference to derived to reference to base  
   **sliced down:** copying, moving, or assigning a derived-type object to a base-type object copies, moves, or assigns only the members in the base-class part of the object.  

3. There is no implicit conversion from the base-class type to the derived type.

4. Like any member, the **derived-to-base conversion** may be inaccessible due to access controls.  




## Virtual Functions  

Which virtual member function is called is determined by **dynamic type** when called by pointer or reference, while calls to nonvirtual functions are bound at compile time  

<font color=red>**Every virtual function must be defined if declared** because the compiler has no way to determine whether it is used while ordinary function need not if not used.</font>

**Any non-static member function, other than a constructor, may be virtual.** 
The ***virtual*** keyword appears only on the declaration inside the class and may not be used on a function definition outside the class body. 

<font color=red>A function that is virtual in a base class is implicitly virtual in its derived classes, and keyword ***virtual*** is not required when a derived class overrides a virtual function.</font>

<font color=red>If a derived class does not **override** a virtual from its base, then, like any other member, the derived class inherits the version defined in its base class. </font>

A derived-class function that overrides an inherited virtual function must have exactly the **same parameter type(s)** as the base-class function that it overrides, and the same return type  <font color=red>(except return a reference (or pointer) to types that are themselves (such return types require that derived-to-base conversion is accessible)</font>  

use the scope operator to call a particular version of virtual function and avoid infinite recursion

```C++
// calls the version from the base class regardless of the dynamic type of baseP
double undiscounted = baseP->Quote::net_price(42);

class base {
public:
  string name() { return basename; }
virtual void print(ostream &os) { os << basename; }
private:
  string basename;
};
class derived : public base {
public:
  /*print(os) is infinite recursion*/ 
  /* use base::print(os) instead*/
  void print(ostream &os) { print(os); os << " " << i; }
private:
  int i;
};
```

### ***override*** and ***final***

***final*** and ***override*** specifiers appear after the parameter list (including any ***const*** or **reference qualifiers**) and after a trailing return  

It is legal to define a function with the same name as a virtual in its base class but with a different parameter list, does not override the version in the base class but hide the name.

***override*** force compiler to reject if the marked function does not override an existing virtual function  

```C++
struct B {
  virtual void f1(int) const;
  virtual void f2();
  void f3();
};
struct D1 : B {
  void f1(int) const override; // ok: f1 matches f1 in the base
  void f2(int) override; // error: B has no f2(int) function
  void f3() override; // error: f3 not virtual
  void f4() override; // error: B doesn't have a function named f4
};
```

***final*** force compiler to reject if the marked function is overrided

```C++
struct D2 : B {
  // inherits f2() and f3() from B and overrides f1(int)
  void f1(int) const final; // subsequent classes can't override f1 (int)
};
struct D3 : D2 {
  void f2(); // ok: overrides f2 inherited from the indirect base, B
  void f1(int) const; // error: D2 declared f2 as final
};
```

#### Virtual Functions and Default Arguments  

a virtual function can have default arguments. 

<font color=red>If a call uses a default argument, the **default value** is the one defined by the **static type** through which the function is called, so virtual functions should use the same **default argument values** in the base and derived classes. Better to use a non-virtual member function with default arguments to encapsulate the virtual one.</font>



## Abstract Base Classes  

A class containing (or inheriting without overridding) a **pure virtual function** is an **abstract base class**. 

An **abstract base class** defines an interface for subsequent classes to override. 

We cannot (directly) create objects of a type that is an **abstract base class**.   

### pure virtual function  

**pure virtual function** is a virtual function that is written `= 0` in place of function body inside class

**pure virtual function** does not have to be defined, and if defined, the function body of definition must be outside the class.

```C++
// class to hold the discount rate and quantity
// derived classes will implement pricing strategies using these data
class Disc_quote : public Quote {
public:
  Disc_quote() = default;
  Disc_quote(const std::string& book, double price, 
             std::size_t qty, double disc):Quote(book, price),
                                           quantity(qty), discount(disc) {}
  double net_price(std::size_t) const = 0;
protected:
  std::size_t quantity = 0; // purchase size for the discount to apply
  double discount = 0.0; // fractional discount to apply
};
```

### Refactoring  

Adding Disc_quote to the Quote hierarchy is an example of refactoring.
Refactoring involves redesigning a class hierarchy to move operations and/or data from one class to another. 

Refactoring is common in object-oriented applications.  

## Access Control and Inheritance  

Member access does not affect visibility 

Member access check is the last step

A derived class must specify from the class(es) it inherits in **class derivation list**, which is a colon followed by a comma-separated list of names of previously defined classes. 

Each base class name may be preceded by an optional access specifier, which is one of ***public***, ***protected***, or ***private***.  

**access specifier**

1. ***private*** members are only accessible to member and friend of this class.  
2. ***protected*** members are accessible to members and friends of this class, <font color=red>For derived classes accessible **only** through a derived object, cannot directly through base-class objects</font>
3.  ***public*** members are accessible to all including users of this class

```C++
class Base {
protected:
  int prot_mem; // protected member
};
class Sneaky : public Base {
  friend void clobber(Sneaky&); // can access Sneaky::prot_mem
  friend void clobber(Base&); // can't access Base::prot_mem
  int j; // j is private by default
};
// ok: clobber can access the private and protected members in Sneaky objects
void clobber(Sneaky &s) { s.j = s.prot_mem = 0; }
// error: clobber can't access the protected members in Base
void clobber(Base &b) { b.prot_mem = 0; }
```

A derived object contains two parts

1. subobject containing the (nonstatic) members defined in the derived class 
2. subobject corresponding to base class

**derivation access specifier**: base class subobject is ***public/protected/private*** part of derived object

| accessibility                                   | members and friends of this class | members and friends of deriving class | users of this class  |
| ----------------------------------------------- | --------------------------------- | ------------------------------------- | -------------------- |
| access to base class subobject private member   | no                                | no                                    | no                   |
| access to base class subobject protected member | yes                               | no when private Base                  | no                   |
| access to base class subobject public member    | yes                               | no when private Base                  | yes when public Base |
| access to this class subobject private member   | yes                               | no                                    | no                   |
| access to this class subobject protected member | yes                               | yes                                   | no                   |
| access to this class subobject public member    | yes                               | yes                                   | yes                  |

```C++
class Base {
public:
  void pub_mem(); // public member
protected:
  int prot_mem; // protected member
private:
  char priv_mem; // private member
};
struct Pub_Derv : public Base {
  // ok: derived classes can access protected members
  int f() { return prot_mem; }
  // error: private members are inaccessible to derived classes
  char g() { return priv_mem; }
};
struct Priv_Derv : private Base {
  // private derivation doesn't affect access in the derived class
  int f1() const { return prot_mem; }
};

Pub_Derv d1; // members inherited from Base are public
Priv_Derv d2; // members inherited from Base are private
d1.pub_mem(); // ok: pub_mem is public in the derived class
d2.pub_mem(); // error: pub_mem is private in the derived class

struct Derived_from_Public : public Pub_Derv {
  // ok: Base::prot_mem remains protected in Pub_Derv
  int use_base() { return prot_mem; }
};
struct Derived_from_Private : public Priv_Derv {
  // error: Base::prot_mem is private in Priv_Derv
  int use_base() { return prot_mem; }
};
```

### Accessibility of Derived-to-Base Conversion  

<font color=red>Whether for member functions and friends or user code, derived-to-base conversion is accessible if public members of the base class are accessible at the given code  </font>

### Friendship and Inheritance  

friendship is not inherited Just as friendship is not transitive

each class controls access to its members  

<font color=red>***friends*** of base class can access to members of base class subobjects that are embedded in an derived object if base class subobjects is accessible.</font>

```C++
class Base {
  friend class Pal; // Pal has no access to classes derived from Base
protected:
  int prot_mem; // protected member
};
class Sneaky : public Base {
  int j; // j is private by default
};
class Sneaky_Priv : private Base {
  int j; // j is private by default
};
class Pal {
public:
  int f(Base b) { return b.prot_mem; } // ok: Pal is a friend of Base
  int f2(Sneaky s) { return s.j; } // error: Pal not friend of Sneaky
  // access to a base class is controlled by the base class, even inside a derived object
  int f3(Sneaky s) { return s.prot_mem; } // ok: Base is public and Pal is a friend
  int f3(Sneaky_Priv s) { return s.prot_mem; } // error: Base is private
};
```



### Exempting Access Level of Individual Members  

A ***using*** declaration inside a class can name any accessible member of a direct or indirect base class. Access to a name specified in a ***using*** declaration depends on the **access specifier** preceding the ***using*** declaration.  

### Default Inheritance Protection Levels  

By default, a derived class defined with the ***class*** keyword has **private** inheritance; 
a derived class defined with ***struct*** has **public** inheritance:  

```C++
class Base { /* ... */ };
struct D1 : Base { /* ... */ }; // public inheritance by default
class D2 : Base { /* ... */ }; // private inheritance by default
```

## Class Scope under Inheritance  

Each class defines its own scope, the scope of a derived class is nested inside the scope of its base classes  

A derived-class member <font color=red>(inner scope)</font> with the same name as a member of the base class hides direct use of the base-class member<font color=red>(outer scope)</font>.  

Using the Scope Operator to Use Hidden Members  

### Name Lookup and Inheritance  

1. determine the **static type**   
2. Look for name in static type class, if not found then search **direct base class** and continue up the chain of inheritance classes, error if not found
3. do normal type checking  
4. check if virtual to generates a normal function call or code to determine at run time  

#### Name Lookup Happens before Type Checking  

the derived member hides the base-class member even if the types are inconsistent, so virtual functions must have the same parameter list   

```C++
struct Base {
  int memfcn();
};
struct Derived : Base {
  int memfcn(int); // hides memfcn in the base
};
Derived d; Base b;
b.memfcn(); // calls Base::memfcn
d.memfcn(10); // calls Derived::memfcn
d.memfcn(); // error: memfcn with no arguments is hidden
d.Base::memfcn(); // ok: calls Base::memfcn
```

### Overriding Overloaded Functions  

Because of <font color=red>name hide</font>, a derived class must override <font color=red>all or none</font> of overloaded functions <font color=red>(**virtual function is nothing special**, so a set of overloaded virtual functions is useless)</font> it inherits. If a derived class wants to make all the overloaded versions available through its type

a ***using*** declaration for a base-class member function adds all the overloaded instances of
that function to the scope of the derived class<font color=red>(not consider accessibility)</font>, the derived class member with same name **hides or overrides** <font color=red>(doesn't conflict with)</font> the member that is introduced from the base class.

every overloaded instance of the function in the base class must be **accessible** to the derived class.  Otherwise <font color=red>compile error </font>



## Constructors and Copy Control  

a base class generally should define a **virtual destructor** because **static type** of the pointer might differ from the **dynamic type** of the object being destroyed. Therefore the compiler will not synthesize a **move operation** for that class  

The base class is initialized first, and then the members of the derived class are initialized in the order in which they are declared in the class  

### Synthesized Copy Control and Inheritance  

The synthesized copy-control members in a derived class memberwise initialize, assign, or destroy the members of the class itself and use the corresponding operation from the base class to initialize, assign, or destroy the direct base part of an object.

It doesn’t matter whether the base-class member is synthesized or user-provided. All that matters is that the corresponding member is accessible and not deleted.  

a derived-class copy-control member to be defined as deleted

1.  the corresponding member in the derived class is defined as deleted  if the default constructor, copy constructor, copy-assignment operator, or destructor in the base class is deleted or inaccessible  
2. the synthesized default and copy constructors in the derived classes are defined as deleted if the base class has an inaccessible or deleted destructor
3. the compiler will not synthesize a deleted move operation. But if we use **= default** to request a move operation, it will be a deleted function in the derived if the corresponding operation or destructor in the base is deleted or inaccessible  

base classes generally do not get synthesized move operations  

### Derived-Class Copy-Control Members  

#### Defining a Derived Copy、Move Constructor or Assignment Operator   

<font color=red>When a derived class defines a **copy or move operation**, that operation is responsible for copying or moving the **entire object**, including **base-class** members.</font>

Due to **derived-to-base conversion**, reference to derived class will match copy constructor of base class.

```C++
class Base { /* ... */ } ;
class D: public Base {
public:
  // by default, the base class default constructor initializes the base part of an object
  // to use the copy or move constructor, we must explicitly call that
  // constructor in the constructor initializer list
  D(const D& d): Base(d) // copy the base members
  /* initializers for members of D */ { /* ... */ }
  D(D&& d): Base(std::move(d)) // move the base members
  /* initializers for members of D */ { /* ... */ }

  // Base::operator=(const Base&) is not invoked automatically
  D & operator=(const D &rhs)
  {
    Base::operator=(rhs); // assigns the base part
    // assign the members in the derived class, as usual,
    // handling self-assignment and freeing existing resources as appropriate
    return *this;
  }
};

// probably incorrect definition of the D copy constructor
// base-class part is default initialized, not copied
D::D(const D& d) /* member initializers, but no base-class initializer */
{ /* ... */ }
```

  

#### Derived-Class Destructor  

Unlike the constructors and assignment operators, <font color=red>the **destructor** is responsible **only** for destroying the resources allocated by the derived class.</font>

Objects are destroyed in the **opposite order** from which they are constructed: The derived destructor is run first, and then the **base-class destructors** are invoked, back up through the inheritance hierarchy.  

```C++
class D: public Base {
public:
  // Base::~Base invoked automatically
  ~D() { /* do what it takes to clean up derived members */ }
};
```



#### Calls to Virtuals in Constructors and Destructors  

If a constructor or destructor **calls a virtual**, the compiler treats the object as if its type changes during construction or destruction and <font color=red>the version that is run is the one corresponding to the type of the constructor or destructor itself/</font>

### Inherited Constructors

A derived class inherits its base-class constructors by providing a ***using*** declaration that names its (direct) base class. 

Ordinarily, a ***using*** declaration only makes a name visible in the current scope. When applied to a constructor, a ***using*** declaration causes the compiler to generate code. That is, <font color=red>for each constructor in the base class, the compiler generates a constructor in the derived class that has the same parameter list, which initializes base class subobject and leaves  data members of the derived class default initialized.</font>

```C++
class Bulk_quote : public Disc_quote {
public:
  using Disc_quote::Disc_quote; // inherit Disc_quote's constructors
  double net_price(std::size_t) const;
};
```

#### Characteristics of an Inherited Constructor  

1. Unlike ***using*** declarations for ordinary members, a constructor ***using*** declaration does not change the **access level** of the inherited constructor(s)
2. ***using*** declaration **ignores member access** and will generate a **private** constructor in the derived for **private** constructor in the base
3. a ***using*** declaration can’t specify ***explicit*** or ***constexpr*** and the inherited constructor preserves the same ***explicit*** and ***constexpr***.
4. If a base-class constructor has **default arguments**, those arguments are **not inherited**. Instead, the derived class gets multiple inherited constructors for every **call signature** 
5. If the derived class defines a constructor with the same parameters as a constructor in the base, then that constructor is **not inherited**.  
6. the **default, copy, and move constructors are not inherited**. These constructors are **synthesized** using the normal rules. An inherited constructor is not treated as a user-defined constructor. Therefore, a class that contains only inherited constructors will have a **synthesized default constructor**  



## Containers and Inheritance  

Objects of types related by inheritance cannot be put directly into a container, use **container of pointers** <font color=red>(preferably smart pointers)</font> instead, because container cannot holds elements of differing types and derived objects will be “sliced down”  

often define auxiliary classes to help manage the complexity of pointers and **hide the pointers**, this requires classes in hierarchy have a **virtual clone member** that allocates a copy of itself  

```C++
class Quote {
public:
  // virtual function to return a dynamically allocated copy of itself
  // these members use reference qualifiers; see §13.6.3 (p. 546)
  virtual Quote* clone() const & {return new Quote(*this);}
  virtual Quote* clone() &&
    {return new Quote(std::move(*this));}
  // other members as before
};
class Bulk_quote : public Quote {
  Bulk_quote* clone() const & {return new Bulk_quote(*this);}
  Bulk_quote* clone() &&
    {return new Bulk_quote(std::move(*this));}
  // other members as before
};

class Basket {
public:
  void add_item(const Quote& sale) // copy the given object
    { items.insert(std::shared_ptr<Quote>(sale.clone())); }
  void add_item(Quote&& sale) // move the given object
    { items.insert(
    std::shared_ptr<Quote>(std::move(sale).clone())); }
  // other members as before
};
```



# Chapter 11. Templates and Generic Programming 

Templates are the foundation for generic programming in C++ 

A template is a blueprint or formula for creating classes or functions.  

A **template definition** starts with the keyword ***template*** followed by a **template parameter list** <font color=red>(cannot be empty)</font>, which is a comma-separated list of one or more **template parameters** bracketed by the less-than (`<`) and greater-than (`>`) tokens. 

The definition of a class template must be visible at the point of implicit instantiation, which is why template libraries typically provide all template definitions in the header.

Template programs should try to minimize the number of requirements placed on the argument types. 

## Template Parameters 

A template parameter name has no intrinsic meaning and can use any name: 

Template parameters follow normal scoping rules: <font color=red>hides same name in an outer scope but cannot be reused within the template. </font>

```C++
typedef double A;
template <typename A, typename B> void f(A a, B b)
{
  A tmp = a; // tmp has same type as the template parameter A, not double
  double B; // error: redeclares template parameter B
}
```

A **template declaration** must include the **template parameters** 

### Template Type Parameters 

**type parameter** is used as a **type specifier**, ordinarily named ***T***

Each type parameter must be preceded by the keyword ***class*** or ***typename***, These keywords have the same meaning and can be used interchangeably.

### Nontype Template Parameters 

A **nontype parameter** represents a **value** rather than a type 

**nontype parameters** are replaced with a **constant expression** supplied by the user or deduced by the compiler, which allows the compiler to instantiate the templates during compile time.  A **nontype parameter** can be used as **constant expressions**, for example, to specify the size of an array. 

```C++
template<unsigned N, unsigned M>
int compare(const char (&p1)[N], const char (&p2)[M])
{
  return strcmp(p1, p2);
}

/* compare("hi", "mom") instantiate the template to the following */
int compare(const char (&p1)[3], const char (&p2)[4])
```

**nontype parameter** must be a  [structural type](https://en.cppreference.com/w/cpp/language/template_parameters#Non-type_template_parameter):

1. an **integral type** 
2. **a pointer or lvalue reference** to an object or to a function type. Arguments bound to a pointer or reference **nontype parameter** must have ***static* **lifetime 

### Default Template Arguments 

Under the new standard, can supply **default arguments** for **both function templates and class templates**. Earlier versions, allowed default arguments **only with class templates** 

all of the parameters on the right side of template parameter with a default argument must have default arguments 

use **empty <>** to use default arguments for all of the template parameters 

```C++
template <class T = int> class Numbers { // by default T is int
public:
  Numbers(T v = 0): val(v) { }
  // various operations on numbers
private:
  T val;
};
Numbers<long double> lots_of_precision;
Numbers<> average_precision; // empty <> says we want the default type
```



## Function Templates 

A function template is a formula from which we can generate type-specific versions of that function 

```C++
template <typename T>
int compare(const T &v1, const T &v2)
/* const T & can be used on types that cannot be copied */
{
  /* using only the < operator, so type T need only implement < operator */
  if (v1 < v2) return -1;
  if (v2 < v1) return 1;
  return 0;
}
// use above to compare two pointers may be undefined

// version of following compare that will be correct even if used on pointers
template <typename T> int compare(const T &v1, const T &v2)
{
  if (less<T>()(v1, v2)) return -1;
  if (less<T>()(v2, v1)) return 1;
  return 0;
}
```

Much like **function parameters**. **template parameters** represent **types or values** used in the definition of a class or function. When we use a template, we specify—either implicitly or explicitly—template argument(s) to bind to the template parameter(s). 

A **function template** can be declared ***inline*** or ***constexpr*** in the same ways as non-template functions. <font color=red>The ***inline*** or ***constexpr*** specifier follows the template parameter list and precedes the return type: </font>

```C++
// ok: inline specifier follows the template parameter list
template <typename T> inline T min(const T&, const T&);
// error: incorrect placement of the inline specifier
inline template <typename T> T min(const T&, const T&);
```

### Instantiating a Function Template 

When we call a function template, the compiler (ordinarily) uses the arguments of the call to <font color=red>deduce</font> the **template argument(s)** to ***instantiate*** a specific version of the function. It creates a new **“instance”** of the template using the **actual template argument(s)** in place of the **corresponding template parameter(s)**. 

## Template Compilation 

code is generated only when we use a template (and not when we define it) , so unlike non-template code, **headers for templates typically include definitions as well as declarations**.

three stages during which the compiler might flag an error 

1. compile the template itself. 

2. compiler sees a use of the template, only check the number of the arguments 

3. instantiation. It is only then that type-related errors can be found. Depending on how the compiler manages instantiation, these errors may be reported at link time. 

caller should guarantee that the arguments passed to the template support any operations that template uses, 

## Class Templates  

A class template is a blueprint for generating classes. 

<font color=red>Differ from function templates</font>, compiler **cannot deduce** the template parameter type(s) for a **class template**. So must supply a list of **explicit template arguments** inside angle brackets following the
template’s name for compiler to instantiate a specific class from the template. 

```C++
template <typename T> class Blob {
public:
  typedef T value_type;
  typedef typename std::vector<T>::size_type size_type;
  // constructors
  Blob();
  Blob(std::initializer_list<T> il);
  // number of elements in the Blob
  size_type size() const { return data->size(); }
  bool empty() const { return data->empty(); }
  // add and remove elements
  void push_back(const T &t) {data->push_back(t);}
  // move version; see § 13.6.3 (p. 548)
  void push_back(T &&t) { data->push_back(std::move(t)); }
  void pop_back();
  // element access
  T& back();
  T& operator[](size_type i); // defined in § 14.5 (p. 566)
private:
  std::shared_ptr<std::vector<T>> data;
  // throws msg if data[i] isn't valid
  void check(size_type i, const std::string &msg) const;
};

Blob<int> ia; // empty Blob<int>
Blob<int> ia2 = {0,1,2,3,4}; // Blob<int> with five elements
Blob<string> names; // Blob that holds strings
Blob<double> prices;// different element typ
```

the name of a class template is not the name of a type, but **inside the scope of the class template** itself, we can <font color=red>use the name of the template without arguments, as if template name was the class type to be instantiated.</font> 

```C++
template <typename T> class BlobPtr
public:
  BlobPtr(): curr(0) { }
  BlobPtr(Blob<T> &a, size_t sz = 0):wptr(a.data), curr(sz) { }
  T& operator*() const
  { auto p = check(curr, "dereference past end");
    return (*p)[curr]; // (*p) is the vector to which this object points
  }
  // prefix increment and decrement operators
  BlobPtr& operator++(); // BlobPtr<T>& operator++();
  BlobPtr& operator--(); // BlobPtr<T>& operator--();
private:
  // check returns a shared_ptr to the vector if the check succeeds
  std::shared_ptr<std::vector<T>>
  check(std::size_t, const std::string&) const;
  // store a weak_ptr, which means the underlying vector might be destroyed
  std::weak_ptr<std::vector<T>> wptr;
  std::size_t curr; // current position within the array
};
```

### Template Type Aliases 

cannot define a ***typedef*** that refers to a template, but can define **template type alias** as a synonym for a family of classes

```C++
template<typename T> using twin = pair<T, T>;
twin<string> authors; // authors is a pair<string, string>
```



### Class-Template Member Functions 

a member function <font color=red>(include copy control member functions)</font> defined outside the class template body starts with the **keyword *template*** followed by the class’ template parameter list. 

```C++
template <typename T>
ret-type Blob<T>::member-name(parm-list)

// constructor is the same
template <typename T>
Blob<T>::Blob(): data(std::make_shared<std::vector<T>>()) { }
```

<font color=red>By default, a member function of a class template is instantiated **only if** the program uses that member function, while data member always instantiated.</font>

### Class Templates and Friends 

the class and the friend can independently be templates or not 

A class template that has a non-template friend grants that friend access to all the instantiations of the template.
When the friend is itself a template, the class granting friendship controls whether friendship includes all instantiations of the template or only specific instantiation(s) 

#### One-to-One Friendship 

friendship between a **class template** to another **template** <font color=red>(class or function)</font> that are instantiated with the same type:  

```C++
// forward declarations needed for friend declarations in Blob
template <typename> class BlobPtr;
template <typename> class Blob; // needed for parameters in operator==
template <typename T>
  bool operator==(const Blob<T>&, const Blob<T>&);
template <typename T> class Blob {
  // each instantiation of Blob grants access to the version of
  // BlobPtr and the equality operator instantiated with the same type
  friend class BlobPtr<T>;
  friend bool operator==<T> (const Blob<T>&, const Blob<T>&);
  // other members as in § 12.1.1 (p. 456)
};
```

#### General and Specific Template Friendship 

- <font color=red>forward declaration is necessary to befriend a specific instantiation of a template</font>
- <font color=red>forward declaration is not necessary to befriend all instantiation of a template because template is not used </font>

```C++
// forward declaration necessary to befriend a specific instantiation of a template
template <typename T> class Pal;
class C { // C is an ordinary, nontemplate class
  friend class Pal<C>; // Pal instantiated with class C is a friend to C
  // all instances of Pal2 are friends to C;
  // no forward declaration required when we befriend all instantiations
  template <typename T> friend class Pal2;
};
template <typename T> class C2 { // C2 is itself a class template
  // each instantiation of C2 has the same instance of Pal as a friend
  friend class Pal<T>; // a template declaration for Pal must be in scope
  // all instances of Pal2 are friends of each instance of C2, prior declaration not needed
  template <typename X> friend class Pal2;
  // Pal3 is a nontemplate class that is a friend of every instance of C2
  friend class Pal3; // prior declaration for Pal3 not needed
};
```

#### Befriending the Template’s Own Type Parameter 

<font color=red>Under the new standard, we can make a template type parameter a friend including built-in type </font>

```C++
template <typename Type> class Bar {
friend Type; // grants access to the type used to instantiate Bar
// ...
};
```

### ***static*** Members of Class Templates 

There must be exactly one **definition** of each ***static*** **data member** of a template class, and one distinct object for each instantiation of a class template. As a result, we define a ***static*** **data member** as a template similarly to member functions of that template

```C++
template <typename T>
size_t Foo<T>::ctr = 0; // define and initialize ctr
```

Like any other member function, ***static*** **member function** is instantiated only if it is used in a program. 

###  Using Class Members That Are Types 

Assuming ***T*** is a **template type parameter**, compiler won’t know whether **T::mem** is a **type** or a ***static*** **data member** until instantiation time

<font color=red>By default, the language assumes that a name accessed through the scope operator is not a type. As a result, we must **explicitly** tell the compiler that the name is a **type** using the keyword ***typename*** </font>

```C++
template <typename T>
typename T::value_type top(const T& c)
{
  if (!c.empty())
    return c.back();
  else
    // call default constructor of class T::value_type
    return typename T::value_type();
}
```

## Member Templates 

an ordinary class or a class template may have a member function that is itself a template, called **member templates**.
<font color=red>Member templates may not be virtual. </font>

```C++
template <typename T> class Blob {
  template <typename It> Blob(It b, It e);
  // ...
};

/* when defined outside */ 
template <typename T> // type parameter for the class
template <typename It> // type parameter for the constructor
  Blob<T>::Blob(It b, It e):
    data(std::make_shared<std::vector<T>>(b, e)) { }

int ia[] = {0,1,2,3,4,5,6,7,8,9};
vector<long> vi = {0,1,2,3,4,5,6,7,8,9};
list<const char*> w = {"now", "is", "the", "time"};
// instantiates the Blob<int> class
// and the Blob<int> constructor that has two int* parameters
Blob<int> a1(begin(ia), end(ia));
// instantiates the Blob<int> constructor that has
// two vector<long>::iterator parameters
Blob<int> a2(vi.begin(), vi.end());
// instantiates the Blob<string> class and the Blob<string>
// constructor that has two (list<const char*>::iterator parameters
Blob<string> a3(w.begin(), w.end());
```



## Controlling Instantiations 

two or more separately compiled source files use the **same template with the same template arguments**, will cause overhead of instantiating the same template in each of those files 
avoid this overhead through an **explicit instantiation**.  

```C++
// instantion declaration and definition
extern template class Blob<string>; // declaration
template int compare(const int&, const int&); // definition
```

the compiler will not generate code for that instantiation in that file after an ***extern*** **template declaration**, so the **extern declaration** must appear before any code that uses that instantiation

An **instantiation definition** for a class template instantiates <font color=red>all the members</font> including ***inline*** member functions because compiler cannot know which member functions the program uses. So an **instantiation definition** can be used **only for** types that can be used with every member function of a class template 

## Name Resolution within a Template 

```C++
// scope of the template definition
extern double foo ( double );
template < class type >
class ScopeRules
{
public:
  void invariant() {
    _member = foo( _val );}
  type type_dependent() {
    return foo( _member );
  }

private:
  int _val;
  type _member;
};
```

<font color=red>the resolution of a non-member name within a template:</font>

1. if the use of the name is not dependent on the template type parameters, the scope of the template declaration determines the resolution of the name.
2. Otherwise, the scope of the template instantiation determines the resolution of the name.

```C++
//scope of the template instantiation
extern int foo( int );
// ...
ScopeRules< int > sr0;

foo(1); // call int foo ( int )
sr0.invariant(); // call double foo ( double )
sr0.type_dependent(); // call double foo ( double )
```

## Function Template Argument Deduction 

Only a very limited number of conversions are automatically applied to function parameters whose type uses a **template type parameter**. But function template can also have parameters that are defined using **ordinary types**. <font color=red>which can be converted as usual</font>

1. top-level ***const*** is ignored
2. **const conversions**: A ***nonconst*** object can be passed to function parameter whose type is a low-level ***const***.
3. **Array- or function-to-pointer conversions**. <font color=red>Occurs when function parameter is not a **reference type**. </font>

```C++
template <typename T> T fobj(T, T); // arguments are copied
template <typename T> T fref(const T&, const T&); // references
string s1("a value");
const string s2("another value");
fobj(s1, s2); // calls fobj(string, string); const is ignored
fref(s1, s2); // calls fref(const string&, const string&)
              // uses premissible conversion to const on s1
int a[10], b[42];
fobj(a, b); // calls f(int*, int*)
/* the parameter is a reference, the arrays are not converted to pointers */
fref(a, b); // error: array types don't match
```

### Function-Template Explicit Arguments 

two situations 

1. impossible for the compiler to deduce the types of the template arguments 

   ```C++
   // T1 cannot be deduced: it doesn't appear in the function parameter list
   template <typename T1, typename T2, typename T3>
   T1 sum(T2, T3);
   // T1 is explicitly specified; T2 and T3 are inferred from the argument types
   auto val3 = sum<long long>(i, lng); // long long sum(int, long)
   
   // poor design: users must explicitly specify all three template parameters
   template <typename T1, typename T2, typename T3>
   T3 alternative_sum(T2, T1);
   // error: can't infer initial template parameters
   auto val3 = alternative_sum<long long>(i, lng);
   // ok: all three parameters are explicitly specified
   auto val2 = alternative_sum<long long, int, long>(i, lng);
   ```

2. user wants to control the template instantiation 

**Explicit template argument(s)** are matched to corresponding template parameter(s) **from left to right** and can use **normal conversions**

```C++
template <typename T>
int compare(const T &v1, const T &v2);

long lng;
compare(lng, 1024); // error: template parameters don't match
compare<long>(lng, 1024); // ok: instantiates compare(long, long)
compare<int>(lng, 1024); // ok: instantiates compare(int, int)
```
#### Trailing Return Types and Type Transformation 

use a **trailing return type** if cannot know the exact type of return object

```C++
// a trailing return lets us declare the return type after the parameter list is seen
template <typename It>
auto fcn(It beg, It end) -> decltype(*beg)
{
  // process the range
  return *beg; // return a reference to an element from the range
}
```

#### Type Transformation Library Template Classes 

library **type transformation**, defined in the ***type_traits*** header, has class template used for so-called **template metaprogramming**. 

The classes have **one template type parameter** and **a (public) type member** named ***type*** that represents a type. That type may be related to the template’s own **template type parameter** in a way that is indicated by the template’s name. If it is not possible (or not necessary) to transform the template’s parameter, the type member is the **template parameter type itself**. 

#### Standard Type Transformation Templates  ***Mod\<T\>***

| For Mod\<T\>, where ***Mod*** is | If T is                                | Then Mod\<T\>::type is |
| -------------------------------- | -------------------------------------- | ---------------------- |
| remove_reference                 | X& or X&& <br>otherwise                | X <br/>T               |
| add_const                        | X&, const X, or function<br/>otherwise | T <br/>const T         |
| add_lvalue_reference             | X& <br/>X&& <br/>otherwise             | T <br/>X& <br/>T&      |
| add_rvalue_reference             | X& or X&& <br/>otherwise               | T <br/>T&&             |
| remove_pointer                   | X\* <br/>otherwisw                     | X<br/>T                |
| add_pointer                      | X& or X&& <br/>otherwise               | X\*<br/>T\*            |
| make_signed                      | unsigned X <br/>otherwise              | X<br/>T                |
| make_unsigned                    | signed type <br/>otherwise             | unsigned T<br/>T       |
| remove_extent                    | X[n] <br/>otherwise                    | X<br/>T                |
| remove_all_extents               | X\[n1\]\[n2\]\... <br/>otherwise       | X<br/>T                |



```C++
// must use typename to use a type member of a template parameter; see § 16.1.3 (p. 670)
template <typename It>
auto fcn2(It beg, It end) ->
typename remove_reference<decltype(*beg)>::type
{
  // process the range
  return *beg; // return a copy of an element from the range
}
```

### Function Pointers and Argument Deduction 

compiler uses the type of the function pointer to deduce the template argument(s) and <font color=red>won’t compile if not possible to identify a unique instantiation for the argument.</font>

```C++
template <typename T> int compare(const T&, const T&);
// pf1 points to the instantiation int compare(const int&, const int&)
int (*pf1)(const int&, const int&) = compare;

// overloaded versions of func; each takes a different function pointer type
void func(int(*)(const string&, const string&));
void func(int(*)(const int&, const int&));
func(compare); // error: which instantiation of compare?
// ok: explicitly specify which version of compare to instantiate
func(compare<int>); // passing compare(const int&, const int&)
```

### Template Argument Deduction and References 

when the function’s parameter is a reference to a template type parameter 

1. normal reference binding rules apply 
2. ***consts*** are low level, not top level 

```C++
template <typename T> void f1(T&); // argument must be an lvalue
// calls to f1 use the referred-to type of the argument as the template parameter type
f1(i); // i is an int; template parameter T is int
f1(ci); // ci is a const int; template parameter T is const int
f1(5); // error: argument to a & parameter must be an lvalue

template <typename T> void f2(const T&); // can take an rvalue
// parameter in f2 is const &; const in the argument is irrelevant
// in each of these three calls, f2's function parameter is inferred as const int&
f2(i); // i is an int; template parameter T is int
f2(ci); // ci is a const int, but template parameter T is int
f2(5); // a const & parameter can be bound to an rvalue; T is int
```

#### Reference Collapsing and Rvalue Reference Parameters 

```C++
template <typename T> void f(T&&); // binds to nonconst rvalues
template <typename T> void f(const T&); // binds to lvalues and const rvalues
```

Ordinarily, we cannot (directly) define a reference to a reference. However, it is possible to do so indirectly through a **type alias** or through a **template type parameter**. And **reference collapsing** applies, for a given type X:

1. X& &, X& &&, and X&& & all collapse to type X&  
2. The type X&& && collapses to X&&  


```C++
template <typename T> void f3(T&&);
// void f3<int>(int &&);
f3(42); // argument is an rvalue of type int; template parameter T is int
// void f3<int&>(int& &&);
f3(i); // argument is an lvalue; template parameter T is int&
//void f3<const int&>(const int& &&);
f3(ci); // argument is an lvalue; template parameter T is const int&
```

##### An example std::move

Ordinarily, a ***static_cast*** can perform only otherwise legitimate conversions. However even though we cannot implicitly convert an lvalue to an rvalue reference, we can explicitly cast an lvalue to an rvalue reference using ***static_cast***  

```C++
// for the use of typename in the return type and the cast see § 16.1.3 (p. 670)
// remove_reference is covered in § 16.2.3 (p. 684)
template <typename T>
typename remove_reference<T>::type&& move(T&& t)
{
  // static_cast covered in § 4.11.3 (p. 163)
  return static_cast<typename remove_reference<T>::type&&>(t);
}
```

It is surprisingly hard to write correct code that is when the function template types involved might be plain (nonreference) types or reference types  

```C++
template <typename T> void f3(T&& val)
{
  T t = val; // copy or binding a reference?
  t = fcn(t); // does the assignment change only t or val and t?
  if (val == t) { /* ... */ } // always true if T is a reference type
}
```

In practice, <font color=red>rvalue reference parameters are used in one of two contexts</font>:

1. the template is forwarding its arguments
2. the template is overloaded  

#### Forwarding  

Some functions need to forward one or more of their arguments with their **types unchanged** to another, forwarded-to, function. So need to preserve everything about the forwarded arguments, including whether or not the argument type is **const**, and whether the argument is an **lvalue** or an **rvalue**

A function template parameter that is an rvalue reference to a template type parameter (i.e., **T&&**) preserves the **constness** and **lvalue/rvalue** property of its corresponding argument  

```C++
template <typename F, typename T1, typename T2>
void flip2(F f, T1 &&t1, T2 &&t2)
{
  /* if T1 is rvalue reference, t1 is an lvalue expression, so cannot preserve rvalue when forwarded-to f */
  f(t2, t1);
}
```

Like ***move***, ***forward*** is defined in the ***utility*** header. Unlike ***move***, ***forward*** must be called with an explicit template argument

As with std::move, it’s a good idea not to provide a using declaration for std::forward. § 18.2.3 (p. 798) will explain why.  

```C++
template <typename F, typename T1, typename T2>
void flip(F f, T1 &&t1, T2 &&t2)
{
  /*std::forward return T2 && type no matter t2 is lvalue/rvlue*/
  f(std::forward<T2>(t2), std::forward<T1>(t1));
}
```



## Overloading and Templates  

Function templates can be overloaded by other **templates** or by **non-template** functions  

1. The **candidate functions** for a call include **any function-template instantiation** for which template argument deduction succeeds.
2. As usual, the **viable functions (template and non-template)** are **ranked** by the conversions
3. if there are several functions that provide an equally good match, then:
   1. If there is **only one non-template function** in the set of equally good matches, the non-template function is called.
   2. If there are no non-template functions in the set, but there are multiple function templates, and one of these templates is more specialized than any of the others, the **more specialized function template** is called.
   3. Otherwise, the call is ambiguous  

### Multiple Viable Templates  

When there are several overloaded templates that provide an equally good match for a call, the **most specialized version** is preferred. Because <font color=red>without this rule, the more general template will always be called</font>

```C++
template <typename T> string debug_rep(const T &t);
template <typename T> string debug_rep(T *p);

string s("hi");
cout << debug_rep(s) << endl; // only the first version is viable

/* 
the first version T bound to string*
the second version T bound to string, is an exact match
*/
cout << debug_rep(&s) << endl;

const string *sp = &s;
/* 
the first version T bound to const string*
the second version T bound to const string, is the more specialized template.
*/
cout << debug_rep(sp) << endl;
```



### Non-template and Template Overloads  

When a non-template function provides an equally good match for a call as a function template, the **non-template version** is preferred  

```C++
template <typename T> string debug_rep(const T &t);
string debug_rep(const string &s);

string s("hi");
/* 
the template T bound to string
the nontemplate version is selected because of more specialized
*/
cout << debug_rep(s) << endl;
```

### Overloaded Templates and Conversions  

for pointers to C-style character strings and string literals

```C++
cout << debug_rep("hi world!") << endl; // calls debug_rep(T*)
```

1. debug_rep(const T&), with T bound to char[10]
2. debug_rep(T\*), with T bound to const char* <font color=red>as good an exact match as debug_rep(const T&) but more specialized </font>
3. debug_rep(const string&), which requires a **user-defined conversion** from const char* to string, <font color=red>less good than an exact match  </font>



## Variadic Templates  

A **variadic template** is a **template function or class** that can take a varying number of parameters known as a **parameter pack**.
There are two kinds of **parameter packs**: 

1. **template parameter pack** (***class...*** or ***typename...***) represents zero or more template parameters, 
2. **function parameter pack** represents zero or more function parameters  

***sizeof...*** returns a **constant expression** and does not evaluate its argument  

```C++
template<typename ... Args> void g(Args ... args) {
  cout << sizeof...(Args) << endl; // number of type parameters
  cout << sizeof...(args) << endl; // number of function parameters
}
```



**variadic function** differs from ***initializer_list***, whose arguments must have the same type (or types that are to a common type).   

**variadic functions are often recursive**:

For the last call in the recursion, both function templates provide an equally good match for the call. However, a nonvariadic template is chosen for this call because of more specialized than a variadic template, 

```C++
// function to end the recursion and print the last element
// this function must be declared before the variadic version of print is defined
template<typename T>
ostream &print(ostream &os, const T &t)
{
  return os << t; // no separator after the last element in the pack
}
// this version of print will be called for all but the last element in the pack
template <typename T, typename... Args>
ostream &print(ostream &os, const T &t, const Args&... rest)
{
  os << t << ", "; // print the first argument
  return print(os, rest...); // recursive call; print the other arguments
}
```

### Pack Expansion  

pack expansion is triggered by putting an **ellipsis (. . . )** to the right of the pattern.  

```C++
template <typename T, typename... Args>
ostream &
print(ostream &os, const T &t, const Args&... rest)// expand Args
{
  os << t << ", ";
  return print(os, rest...); // expand rest
}

// call debug_rep on each argument in the call to print
template <typename... Args>
ostream &errorMsg(ostream &os, const Args&... rest)
{
  // print(os, debug_rep(a1), debug_rep(a2), ..., debug_rep(an)
  return print(os, debug_rep(rest)...);
}

// passes the pack to debug_rep; print(os, debug_rep(a1, a2, ..., an))
print(os, debug_rep(rest...)); // error: no matching function to call
```

### Forwarding Parameter Packs  

preserving type information is a two-step process.   

1. define function parameters as **rvalue references** to a template type parameter 
2. use `forward` to preserve the arguments’ original types   

```C++
template <class... Args>
inline
void StrVec::emplace_back(Args&&... args)
{
  chk_n_alloc(); // reallocates the StrVec if necessary
  alloc.construct(first_free++, std::forward<Args>(args)...);
}
```

## Template Specializations  

When we can’t (or don’t want to) use the template version, we can define a specialized version of the **class or function template**.  

A **template specialization** is a separate definition of the template in which one or more template parameters are specified  to have particular types  

**template specialization** must be in the same **namespace** in which the original template is
defined.   

### Function Template Specialization  

To indicate that we are specializing a template, we use the keyword ***template*** followed by an **empty pair of angle brackets (< >)**. The function parameter type(s) must match the corresponding types in a previously declared template:  

```C++
template <typename T> int compare(const T&, const T&);

// special version of compare to handle pointers to character arrays
template <>
int compare(const char* const &p1, const char* const &p2)
{
  return strcmp(p1, p2);
}
```

#### Function Overloading versus Template Specializations  

When defining a **function template specialization**, we are essentially taking over the job of the compiler to supply the definition to use for a specific instantiation of the original template. It is not an overloaded instance of the function name, so do **not affect function matching**  

<font color=red>Templates and their specializations should be declared in the same header file  </font>

### Class Template Specializations  

class template starts with ***template<>***  is **fully specialized**.

```C++
// open the std namespace so we can specialize std::hash
namespace std {
template <> // we're defining a specialization with
struct hash<Sales_data> // the template parameter of Sales_data
{
  // the type used to hash an unordered container must define these types
  typedef size_t result_type;
  typedef Sales_data argument_type; // by default, this type needs ==
  size_t operator()(const Sales_data& s) const;
  // our class uses synthesized copy control and default constructor
};
} // close the std namespace; note: no semicolon after the close curly
```

### Class-Template Partial Specializations  

**class template partial specialization is itself a template**, which specify some, but not all, of the template parameters or some, but not all, aspects of the parameters.  

The template parameter list of a **partial specialization** is a subset of, while **fully specialization** is just one specialization , the parameter list of the original template.

<font color=red>only a class template can be partially specialized. We cannot partially
specialize a function template.</font>

example **remove_reference:**

```C++
template <class T> struct remove_reference {
  typedef T type;
};
// partial specializations that will be used for lvalue and rvalue references
template <class T> struct remove_reference<T&> // lvalue references
{ typedef T type; };
template <class T> struct remove_reference<T&&> // rvalue references
{ typedef T type; };
```

#### Specializing Members but Not the Class   

```C++
template <typename T> struct Foo {
  Foo(const T &t = T()): mem(t) { }
  void Bar() { /* ... */ }
  T mem;
  // other members of Foo
};
template<> // we're specializing a template
void Foo<int>::Bar() // we're specializing the Bar member of Foo<int>
{
// do whatever specialized processing that applies to ints
}

Foo<string> fs; // instantiates Foo<string>::Foo()
fs.Bar(); // instantiates Foo<string>::Bar()
Foo<int> fi; // instantiates Foo<int>::Foo()
fi.Bar(); // uses our specialization of Foo<int>::Bar(
```



# Chapter 12. Exception Handling  

In C++, an exception is raised by **throwing** an expression and select handler which is the one nearest in the call chain that matches the type of the thrown object.  

## Stack Unwinding  

When an exception is thrown, execution of the current function is suspended

1. search for a matching ***catch*** clause begins. If a matching ***catch*** is found, the exception is handled by that ***catch***. When the ***catch*** completes, execution continues at the point immediately after the last ***catch*** clause associated with that ***try*** block  
2. Otherwise, the search continues through the ***catch*** clauses of the enclosing ***trys***. 
3. If no matching ***catch*** is found, the current function is exited, and the search continues in the calling function.
4. If no matching ***catch*** is found, the program calls the library ***terminate*** function, stops execution of the program.  

<font color=red>When a block is exited during **stack unwinding**, the compiler guarantees that objects created in that block are properly destroyed, even if object under construction is only partially constructed </font> 

### Destructors and Exceptions  

During **stack unwinding**, destructors are run on local objects of class type. Because destructors are run automatically, they should not throw. If, during **stack unwinding**, a destructor throws an exception that it does not also catch, the program will be terminated  

## Catching an Exception  

<font color=red>The **exception declaration** in a ***catch*** **clause** looks like a function parameter list with exactly one parameter.</font>

The type of the declaration determines what kinds of exceptions the handler can catch. The type must be a **complete type** and can be an **lvalue reference** but may not be an **rvalue reference**.
### Exception Object  

The compiler uses the **thrown expression** to copy initialize a special object known as the **exception object** whose type is determined by the **static, compile-time type** of that expression.  
When a catch is entered, the parameter in its exception declaration is initialized by the **exception object**. The **exception object** is destroyed after the exception is completely handled

### Finding a Matching Handler  

the selected ***catch*** is the <font color=red>**first one**</font> that matches the exception. As a consequence, in a list of **catch clauses**, <font color=red>the most specialized catch must appear first, exceptions from an inheritance hierarchy must order from most derived type to least derived. </font>

the types of the exception and the **catch declaration** must match exactly with only a few conversions:

1. Conversions from **non-const** to **const**  
2. Conversions from **derived type** to **base type**  
3. Conversions from array/function to pointer

### Rethrow  

A **rethrow** is a throw that is not followed by an expression, the (current) exception object is passed up the chain.  

```C++
throw;
```

An empty throw can appear only in a ***catch*** or in a function called (directly or indirectly) from a ***catch***.  

### The Catch-All Handler  

**catch-all handlers**, have the form ***catch(...)***. A **catch-all clause** matches any type of exception  

A ***catch(...)*** is often used in combination with a ***rethrow*** expression. The catch does whatever local work can be done and then ***rethrows*** the exception  

```C++
void manip() {
  try {
    // actions that cause an exception to be thrown
  }
  catch (...) {
    // work to partially handle the exception
    throw;
  }
}
```

## Function try Blocks and Constructors  

an exception might occur while processing a **constructor initializer**.
Constructor initializers execute before the constructor body is entered. So a **catch** inside the constructor body can’t handle an exception thrown by a **constructor initializer**. To handle an exception from a **constructor initializer**, we must write the ***constructor*** as a **function try block** 

```C++
template <typename T>
Blob<T>::Blob(std::initializer_list<T> il) try :
  data(std::make_shared<std::vector<T>>(il)) {
  /* empty body */
} catch(const std::bad_alloc &e) { handle_out_of_memory(e); }
```

## ***noexcept*** Exception Specification  

<font color=red>The ***noexcept*** specifier must appear on **all of** the declarations and corresponding definition of a function or on none of them. The specifier precedes a trailing return. </font>

We may also specify ***noexcept*** on the declaration and definition of a **function pointer**, though it may not appear in **a typedef or type alias**. 

In a member function the ***noexcept*** specifier follows any **const** or **reference qualifiers**, and it precedes ***final***, ***override***, or ***= 0*** on a virtual function

***noexcept*** has two meanings:

1. It is an **exception specifier** when it follows a function’s parameter list
2. it is an **operator** that is often used as the **bool** argument to a **noexcept exception specifier**.  

### Exception Specifications and Pointers

Although the ***noexcept*** specifier is not part of a function’s type, a pointer to function and the function to which that pointer points must have compatible specifications.  

1. a pointer declared a nonthrowing exception specification, can only point to ***noexcept*** qualified functions. 
2. A pointer that specifies (explicitly or implicitly) that it might throw can point to any function, even if that function has a ***noexcept*** specifier

```C++
// both recoup and pf1 promise not to throw
void (*pf1)(int) noexcept = recoup;
// ok: recoup won't throw; it doesn't matter that pf2 might
void (*pf2)(int) = recoup;
pf1 = alloc; // error: alloc might throw but pf1 said it wouldn't
pf2 = alloc; // ok: both pf2 and alloc might throw
```

<font color=red>So **inherited virtuals** must also have compatible specification with **base virtual function**</font>

### Violating the Exception Specification  

compiler does not check the ***noexcept*** specification at compile time.   

If a ***noexcept*** function does throw, ***terminate*** is called to enforce the promise not to throw at run time  

### Backward Compatibility: Exception Specifications  

A function can specify the keyword ***throw***, in the same place as the ***noexcept*** specifier does in the current language,  followed by a parenthesized list of types that the function might throw 

there is one use of the old scheme that is in widespread use.

```C++
void recoup(int) noexcept; // recoup doesn't throw
void recoup(int) throw(); // equivalent declaration
```

 ### Arguments to the ***noexcept*** Specification  

The ***noexcept*** specifier takes an optional argument that must be convertible to bool:  

```C++
void recoup(int) noexcept(true); // recoup won't throw
void alloc(int) noexcept(false); // alloc can throw
```

### ***noexcept*** Operator  

**noexcept operator** is a unary operator that returns a **bool rvalue constant expression** that indicates whether a given expression might throw. Like ***sizeof***, ***noexcept*** does not evaluate its operand.  

```C++
noexcept(recoup(i)) // true if calling recoup can't throw, false otherwise
```

## Exception Class Hierarchies  

The only operations that the **exception types** define are the **copy constructor**, **copy-assignment operator**, a **virtual destructor**, and a **virtual member** named ***what***.
The ***what*** function returns a **const char*** that points to a null-terminated character array, and is guaranteed not to throw any exceptions.  

The exception, **bad_cast** and **bad_alloc** classes also define a default constructor.
The **runtime_error** and **logic_error** classes do not have a default constructor but do
have constructors that take a C-style character string or a library ***string*** argument, returned by ***what*** member.  

# Chapter 13. Namespaces  

Libraries that put names into the **global namespace** are said to cause **namespace pollution**.
Traditionally, programmers avoided namespace pollution by using very long names for the global entities they defined 

**Namespaces** provide a much more controlled mechanism for preventing name collisions. **Namespaces** partition the global namespace. 
A namespace is a scope  

## Global Namespace  

names declared outside any class, function, or namespace) are defined inside the **global namespace**  

<font color=red>The scope operator can be used to refer to members of the **global namespace**.  </font>

```C++
::member_name
```

## Nested Namespaces  

A nested namespace is a nested scope  

## Inline Namespaces  

An **inline namespace** is defined by preceding the keyword ***namespace*** with the keyword ***inline***. 

The keyword  ***inline*** must appear on the first definition of the namespace. If the namespace is later reopened, the keyword ***inline*** need not be, but may be, repeated.  

```C++
inline namespace FifthEd {
// namespace for the code from the Primer Fifth Edition
}
namespace FifthEd { // implicitly inline
  class Query_base { /* ... * /};
  // other Query-related declarations
}
```

**Inline namespaces** can be only accessed by the name of the **enclosing namespace** and are often used when code changes from one release of an application to the next.

```C++
namespace FourthEd {
  class Item_base { /* ... */};
  class Query_base { /* ... */};
  // other code from the Fourth Edition
}
namespace cplusplus_primer {
#include "FifthEd.h" // cplusplus_primer:: will get inline namespace FifthEd 
#include "FourthEd.h" // can accessed by cplusplus_primer::FourthEd::
}
```

## Unnamed Namespaces  

The use of file ***static*** declarations inherited from C, is deprecated by the C++ standard. File ***static*** should be avoided and **unnamed namespaces** used instead.  

Variables defined in an **unnamed namespace** have ***static*** lifetime, created before their first use and destroyed when the program ends.
Unlike other namespaces, an unnamed namespace may be discontiguous within a given file but does **not span files**. Each file has its own unnamed namespace.  

Names defined in an **unnamed namespace** are in the same scope as the scope at which the namespace is defined

Names defined in an **unnamed namespace** can be used directly, need not scope operator  

## Namespace Definitions  

A namespace definition begins with the keyword **namespace** followed by the namespace name. Following the namespace name is a sequence of declarations and definitions delimited by curly braces. 

**Any declaration that can appear at global scope can be put into a namespace**: classes, variables (with their initializations), functions (with their definitions), templates, and other namespaces:  

Namespaces may be defined at global scope or inside another namespace but not be defined inside a function or a class.  

A namespace scope does not end with a semicolon  

### Namespace Is a Scope  

As is the case for any scope, each name in a namespace must refer to a unique entity within that namespace.   

Names defined in a namespace may be accessed directly by other members of the namespace, including scopes nested within those members. Code outside the namespace must indicate the namespace in which the name is defined:
### Namespaces Can Be Discontiguous  

Unlike other scopes, a namespace can be defined in several parts, allowing us to use separate files to contain multiple, unrelated type  that the namespace defines.  

### Defining Namespace Members  

The namespace name declaration of the name must be in scope, and the definition must specify the namespace to which the name belongs:
```C++
// namespace members defined outside the namespace must use qualified names
cplusplus_primer::Sales_data
cplusplus_primer::operator+(const Sales_data& lhs,
const Sales_data& rhs)
{
  Sales_data ret(lhs);
  // ...
}
```

Although a namespace member can be defined outside its namespace, <font color=red>such definitions must appear in an enclosing namespace and must not in an unrelated namespace </font>

<font color=red>**Template specializations** must be defined in the same namespace that contains the original template  </font>

## Using Namespace Members  

easier way to use namespace members:

1. namespace aliases
2. using declarations
3. using directives  

Ordinarily, headers should define only the names that are part of its interface, not names used in its own implementation. As a result, header files should not contain **using directives** or **using declarations** except inside functions or namespaces

### Namespace Aliases  

A **namespace alias declaration** begins with the keyword ***namespace***, followed by the alias name, followed by the = sign, followed by the original namespace name and a semicolon  

A namespace can have many synonyms, or aliases. All the aliases and the original namespace name can be used interchangeably  

```C++
namespace primer = cplusplus_primer
namespace Qlib = cplusplus_primer::QueryLib;
Qlib::Query q;
```

### using Declarations: A Recap  

A **using declaration** introduces only one namespace member at a time, <font color=red>as if the using declaration declares a local alias for the namespace member.  </font>

A **using declaration** can appear in global, local, namespace, or class scope   

#### Overloading and using Declarations  

<font color=red>a **using declaration** declares a name, not a specific function. So a using declaration for a function name will bring all the versions of that function into the current scope</font>

1. **using declaration** hides existing declarations for that name in the outer scope.   
2. If the **using declaration** introduces a function in a scope that already has a function of the same name with the same parameter list, then the **using declaration** is in **error**.   
3. Otherwise, the **using declaration** defines additional overloaded instances of the given name  

### using Directives  

In general, a namespace might include definitions that cannot appear in a local scope. 

A **using directive** may appear in global, local, or namespace scope. <font color=red>It may not appear in a class scope.</font>
**using directives** make all the names from a specific namespace visible without qualification  

<font color=red>a **using directive** does not declare local aliases. Rather namespace members are treated as if they appeared in the nearest enclosing namespace scope, so the namespace member is added to the overload set:</font> It is possible for names in the namespace to conflict with other names defined in that (enclosing) scope.  Such conflicts are permitted, but to use the name, we must explicitly indicate which version is wanted.   

One place where using directives are useful is in the implementation files of the namespace itself   

```C++
namespace blip {
int i = 16, j = 15, k = 23;
// other declarations
}
int j = 0; // ok: j inside blip is hidden inside a namespace
void manip()
{
  // using directive; the names in blip are ''added'' to the global scope
  using namespace blip; // clash between ::j and blip::j
  // detected only if j is used
  ++i; // sets blip::i to 17
  ++j; // error ambiguous: global j or blip::j?
  ++::j; // ok: sets global j to 1
  ++blip::j; // ok: sets blip::j to 16
  int k = 97; // local k hides blip::k
  ++k; // sets local k to 98
}
```

## Namespaces Lookup and Overload

Name lookup for names used inside a namespace follows the normal lookup rules: The search looks outward through the enclosing scopes  

<font color=red>With the exception of member function definitions inside the class body, enclosing scopes are always searched upward; names must be declared before they can be used  </font>

```C++
namespace A {
  int i;
  int k;
  class C1 {
  public:
    C1(): i(0), j(0) { } // ok: initializes C1::i and C1::j
    int f1() { return k; } // returns A::k
    int f2() { return h; } // error: h is not defined
    int f3();
  private:
    int i; // hides A::i within C1
    int j;
  };
  int h = i; // initialized from A::i
}
// member f3 is defined outside class C1 and outside namespace A
int A::C1::f3() { return h; } // ok: returns A::h
```



### Argument-Dependent Lookup   

an important exception for name-lookup.

When we pass an object/pointers/references of a class type to a function, the compiler searches the namespace in which the argument’s class is defined in addition to the normal scope lookup.  

Any functions in those namespaces that have the same name as the called function are added to the **overload candidate set** even though they are not visible at the point of the call:  

the chances that an application specifically wants to override the behavior of move (and forward) are pretty small. In general case we want the version from the standard library, so we suggest std::move rather than move  

<font color=red>undeclared class or function that is first named in a **friend declaration** is assumed to be a member of the closest enclosing namespace.  </font>

```C++
namespace A {
class C {
  // two friends; neither is declared apart from a friend declaration
  // these functions implicitly are members of namespace A
  friend void f2(); // won't be found, unless otherwise declared
  friend void f(const C&); // found by argument-dependent lookup
  };
}
```



# Chapter 14. Multiple and Virtual Inheritance 

## Multiple Inheritance 

Multiple inheritance is the ability to derive a class from more than one direct base class 

```C++
class Bear : public ZooAnimal 
{
class Panda : public Bear, public Endangered { /* ... */ };
    
// explicitly initialize both base classes
Panda::Panda(std::string name, bool onExhibit)
    : Bear(name, onExhibit, "Panda"),
      Endangered(Endangered::critical) { }
// implicitly uses the Bear default constructor to initialize the Bear subobject
Panda::Panda() : Endangered(Endangered::critical) { }
```

a derived class can inherit its constructors from one or more of its base classes. 

It is an error to inherit the same constructor (i.e., one with the same parameter list) from more than one base class, unless it defines its own version of that constructor 

```C++
struct Base1 {
  Base1() = default;
  Base1(const std::string&);
  Base1(std::shared_ptr<int>);
};
struct Base2 {
  Base2() = default;
  Base2(const std::string&);
  Base2(int);
};
// error: D1 attempts to inherit D1::D1 (const string&) from both base classes
struct D1: public Base1, public Base2 {
  using Base1::Base1; // inherit constructors from Base1
  using Base2::Base2; // inherit constructors from Base2
};

struct D2: public Base1, public Base2 {
  using Base1::Base1; // inherit constructors from Base1
  using Base2::Base2; // inherit constructors from Base2
  // D2 must define its own constructor that takes a string
  D2(const string &s): Base1(s), Base2(s) { }
  D2() = default; // needed once D2 defines its own constructor
};
```

A pointer or reference to any of an object’s (accessible) base classes can be used to point or refer to a derived object.

Converting from derived-class to each base class is equally good 

### As always, name lookup happens before type checking  

As always, **name lookup** happens before **type checking**  

Under **multiple inheritance**, this same lookup happens simultaneously among all the direct base classes. If a name is found through more than one base class, then use of that name is ambiguous. 

It generates an error noting that the call is ambiguous even if the two inherited functions had different parameter lists. The best way to avoid potential ambiguities is to define a version of the function in the derived class that resolves the ambiguity 

## Virtual Inheritance 

a class can inherit from the same base class more than once and will have more than one subobject of that type. It might inherit the same base indirectly from two of its own direct base classes, or it might inherit a particular class directly and indirectly through another of its base classes. 

Such as iostream, we can share the same base-class subobject object by using **virtual inheritance**. The shared base-class subobject is called a **virtual base class** 

```C++
// the order of the keywords public and virtual is not significant
class Raccoon : public virtual ZooAnimal { /* ... */ };
class Bear : virtual public ZooAnimal { /* ... */ };

class Panda : public Bear,
              public Raccoon, public Endangered {
};
```

### Visibility of Virtual Base-Class Members 

assume class B defines a member named x; class D1 inherits virtually from B as does class D2; and class D inherits from D1 and D2 

1. name x defined in either D1 or D2 will hide x in B
2. If x is defined in both D1 and D2, but not defined in D, access to that member is ambiguous 

### Constructors and Virtual Inheritance 

<font color=red>In a virtual derivation, the **virtual base** is initialized by the **most derived constructor** (explicitly initialize or otherwise default constructor or error if  have no default constructor)</font>

```C++
Bear::Bear(std::string name, bool onExhibit):
  ZooAnimal(name, onExhibit, "Bear") { }
Raccoon::Raccoon(std::string name, bool onExhibit): 
  ZooAnimal(name, onExhibit, "Raccoon") { }

Panda::Panda(std::string name, bool onExhibit): 
  ZooAnimal(name, onExhibit, "Panda"),
  Bear(name, onExhibit),
  Raccoon(name, onExhibit),
  Endangered(Endangered::critical),
  sleeping_flag(false) { }
```

The construction order for an object ***Panda*** with a virtual base is slightly modified from the normal order: 

1. The virtual base subparts ***ZooAnimal*** of the object are initialized first, using initializers provided in the constructor for the most derived class or default constructor if explicitly initializer not provided 
2. the direct base subparts are constructed in the order in which they appear in the derivation list. 
   ***Bear*** ***Raccoon*** ***Endangered***
3. ***Panda*** subparts are constructed last

The same order is used in the synthesized copy and move constructors, and an object is destroyed in **reverse order** from which it was constructed 

# Chapter 15. Specialized Tools and Techniques 

## Controlling Memory Allocation 

### ***new*** expression and ***delete*** expression

```C++
new (type ) new-initializer (optional)
new type new-initializer (optional)

// Placement new
new (placement-args ) (type ) new-initializer (optional)
new (placement-args ) type new-initializer (optional)
delete   expression
delete[] expression
```

overloading ***new*** and ***delete*** operators just provide an alternative **operator new/delete function** for ***new*** or ***delete*** expressions. In fact, we cannot redefine the behavior of the ***new*** and ***delete*** expressions. 

**new expression** always executes by 

1. calling an **operator new function** <font color=red>(standard library or user-defined version)</font> to obtain memory
2. constructing an object in that memory. 
3. return a pointer, with type of void*, referred to the newly allocated and constructed object. 

**delete expression** always executes by 

1. destroying an object 
2. calling an **operator delete function** <font color=red>(standard library or user-defined version)</font> to free the memory used by the object 
3. return ***void***

Calling a destructor destroys an object but does not free the memory 

```C++
string *sp = new string("a value"); // allocate and initialize a string
sp->~string();
```

a **new/delete** expression looks up **operator new/delete function**:

1. <font color=red>if the class of the object being allocated (deallocated), including any base classes, has **operator new/delete members**,  use them and hide **operator new/delete functions in outer scope**</font>
2. <font color=red>if no class-specific **operator new/delete function**, look up a **user-defined version** in the global scope and finally use **standard library version** if no user-defined version **matching function signature** is found</font>

<font color=red>use the **scope operator** to force a ***new*** or ***delete*** expression to bypass a **class-specific function** and use the one from the global scope. `:new` will look only in the **global scope** </font>

### operator new function and operator delete function

parameter of **operator new function** and **operator delete function** may not have a default argument 

When defined as members of a class, the operator **new** and **delete** functions functions are implicitly and must be ***static*** because **operator new** are used before the object is constructed and **operator delete** are after it has been destroyed.  <font color=red>There is no need to declare them static explicitly,</font> although it is legal to do so.   

**new expression** calls, passing the number of bytes requested as the first argument of type [std::size_t]

```C++
// these versions might throw an exception
void* operator new(size_t); // allocate an object
void* operator new[](size_t); // allocate an array
```

**placement new expression** calls

```C++
void* operator new(size_t, nothrow_t&) noexcept;
void* operator new[](size_t, nothrow_t&) noexcept;
void* operator new  ( std::size_t count, void* ptr );
void* operator new[]( std::size_t count, void* ptr );
void* operator new  ( std::size_t count, user-defined-args... );
void* operator new[]( std::size_t count, user-defined-args... );
```

following **operator delete function** is called if executing **delete expression** or

```C++
void operator delete(void*) noexcept; // free an object
void operator delete[](void*) noexcept; // free an array
```

following **operator delete function** is called if exception is thrown when executing **new expression** with the matching signature

<font color=red>Like destructors, an operator delete must not throw an exception </font>. The behavior is undefined if terminated by throwing an exception.

```C++
void operator delete(void*, nothrow_t&) noexcept;
void operator delete[](void*, nothrow_t&) noexcept;
void operator delete( void* ptr, void* place ) noexcept;
void operator delete[]( void* ptr, void* place ) noexcept;
void operator delete  ( void* ptr, args... );
void operator delete[]]( void* ptr, args... );
// class-specific only
void T::operator delete  ( void* ptr, std::size_t sz );	
void T::operator delete[]( void* ptr, std::size_t sz );
```

#### placement new expression

```C++
void* operator new(size_t, nothrow_t&) noexcept;
void* operator new[](size_t, nothrow_t&) noexcept;
```

The type ***nothrow_t*** is a struct defined in the ***new*** header. This type has no members. The new header also defines a const object named ***nothrow***,

```C++
// if allocation fails, new returns a null pointer
int *p1 = new int; // if allocation fails, new throws std::bad_alloc
int *p2 = new (nothrow) int; // if allocation fails, new returns a null pointer
```



```C++
/* this function does not allocate any memory, only initializing an object at the given address */
void* operator new  ( std::size_t count, void* ptr );
void* operator new[]( std::size_t count, void* ptr );
void operator delete( void* ptr, void* place ) noexcept;
void operator delete[]( void* ptr, void* place ) noexcept;
```

The standard library version of  above `operator delete function` cannot be replaced and can only be customized as a class-specific placement new/delete

```C++
new (place_address) type
new (place_address) type (initializers)
new (place_address) type [size]
new (place_address) type [size] { braced initializer list }
```

The pointer that we pass to `construct` must point to space allocated by the same `allocator object`. The
pointer that we pass to **placement new ** need not even refer to dynamic memory 



```C++
// class-specific only
void T::operator delete  ( void* ptr, std::size_t sz );	
void T::operator delete[]( void* ptr, std::size_t sz );
```

When **operator delete** or **operator delete[]** is defined as a class member, the function may have a second parameter of type ***size_t*** is initialized with the size in bytes of the object addressed by the first parameter depending on the dynamic type of the object.



### ***malloc*** and ***free*** Functions 

C++ inherits functions named ***malloc*** and ***free*** from C, defined in ***cstdlib***. 

A simple way to write **operator new** and **operator delete** is as follows: 

```C++
void *operator new(size_t size) {
  if (void *mem = malloc(size))
    return mem;
  else
    throw bad_alloc();
}

void operator delete(void *mem) noexcept { free(mem); }
```



## Run-Time Type Identification 

use **RTT operator**s is more error-prone and it's better to use **virtual member function** if possible

**Run-time type identification (RTTI)** is provided through two operators: 

1. **typeid operator**, which returns the type of a given expression 
2. **dynamic_cast** operator, which safely converts a pointer/reference to a base type into a pointer/reference to a derived type 

RTT operators use the dynamic type of the object when applied to pointers or references to types that **have virtual functions**

### the dynamic_cast Operator 

```C++
dynamic_cast<type*>(e) // return 0 if failed
dynamic_cast<type&>(e) // throw an exception of type bad_cast if failed
dynamic_cast<type&&>(e) // throw an exception of type bad_cast if failed
```



```C++
if (Derived *dp = dynamic_cast<Derived*>(bp))
{
  // use the Derived object to which dp points
} else { // bp points at a Base object
  // use the Base object to which bp points
}

void f(const Base &b)
{
  try {
    const Derived &d = dynamic_cast<const Derived&>(b);
    // use the Derived object to which b referred
    } catch (bad_cast) {
  // handle the fact that the cast failed
  }
}
```

### The typeid Operator 

`typeid(expression) ` return a reference to a ***const*** object of a library type named ***type_info*** defined in the ***typeinfo*** heade, or a type publicly derived from ***type_info***.

type conversion

1. top-level ***const*** is ignored
2.  if the expression is a reference, ***typeid*** returns the type to which the reference refers. 
3. array/function conversion to pointer is not done 

Within C++, for minimizing overhead, distinguish a polymorphic class by the presence of one or more declared or inherited virtual functions 
if class type of `expression` is a class without virtual functions, the **typeid operator** indicates the static type, the compiler knows the static type without evaluating the expression. 
Otherwise, **typeid** requires a run-time check, the expression will be evaluated at run-time

#### ***type_info*** Class 

The exact definition of the type_info class varies by compiler 

The class also provides a public virtual destructor 

***type_info*** has no  default constructor, and the copy and move constructors and the assignment operators are all defined as deleted  

### Example: equal for class hierarchy 

```C++
class Base {
  friend bool operator==(const Base&, const Base&);
  public:
    // interface members for Base
  protected:
    virtual bool equal(const Base&) const;
    // data and other implementation members of Base
};
class Derived: public Base {
  public:
    // other interface members for Derived
  protected:
    bool equal(const Base&) const;
    // data and other implementation members of Derived
};

bool operator==(const Base &lhs, const Base &rhs)
{
  // returns false if typeids are different; otherwise makes a virtual call to equal
  return typeid(lhs) == typeid(rhs) && lhs.equal(rhs);
}

bool Base::equal(const Base &rhs) const
{
  // do whatever is required to compare to Base objects
}

bool Derived::equal(const Base &rhs) const
{
  // we know the types are equal, so the cast won't throw
  auto r = dynamic_cast<const Derived&>(rhs);
  // do the work to compare two Derived objects and return the result
}
```



## Enumerations 

**Enumerations are literal types, group together sets of integral constants.**

C++ has two kinds of enumerations: **scoped** and **unscoped** 

**scoped enumerations** defined using the keywords `enum class` (or, equivalently, `enum struct`), followed by the **enumeration name** and a comma-separated list of **enumerators** enclosed in curly braces. A semicolon follows the close curly 

```C++
enum class open_modes {input, output, append};
```

**unscoped enumeration** defined by omitting the ***class*** (or ***struct***) keyword. The enumeration name is optional

```C++
enum color {red, yellow, green}; // unscoped enumeration
// unnamed, unscoped enum
enum {floatPrec = 6, doublePrec = 10, double_doublePrec = 10};
```

### Enumerators 

The enumerator names in a **scoped enumeration** follow normal scoping rules and are inaccessible outside the scope of the enumeration. 
The enumerator names in an **unscoped enumeration** are placed into the same scope as the enumeration itself: 

```C++
enum color {red, yellow, green}; // unscoped enumeration
enum stoplight {red, yellow, green}; // error: redefines enumerators
enum class peppers {red, yellow, green}; // ok: enumerators are hidden
color eyes = green; // ok: enumerators are in scope for an unscoped enumeration
peppers p = green; // error: enumerators from peppers are not in scope
// color::green is in scope but has the wrong type
color hair = color::red; // ok: we can explicitly access the enumerators
peppers p2 = peppers::red; // ok: using red from peppers
```

By default, enumerator values start at 0 and each enumerator has a value 1 greater than the preceding one. 

Enumerators are ***const*** and, if initialized, their initializers must be **constant expressions**

### Enumerations Define New Types 

An enum object may be initialized or assigned only by one of its enumerators or by another object of the same enum type. So <font color=red>an integral value cannot be used to call a function expecting an enum argument: </font>

<font color=red>Objects or enumerators of an unscoped enumeration type are automatically converted to an integral type while those of scoped enumeration cannot</font>

```C++
open_modes om = 2; // error: 2 is not of type open_modes
om = open_modes::input; // ok: input is an enumerator of open_modes
int i = color::red; // ok: unscoped enumerator implicitly converted to int
int j = peppers::red; // error: scoped enumerations are not implicitly converted
```

#### specifying underlying integral types 

Although each ***enum*** defines a unique type, it is represented by one of the built-in integral types, specified by following enum name with a colon and the name of the type we want to use 

```C++
enum intValues : unsigned long long {
  charTyp = 255, shortTyp = 65535, intTyp = 65535,
  longTyp = 4294967295UL,
  long_longTyp = 18446744073709551615ULL
};
```

1. **scoped enumerations**: default underlying integral type is `int`
2. **unscoped enumeration**: **no default underlying type**, but large enough to hold the enumerator values 

#### forward Declarations for Enumerations 

<font color=red>An **enum forward declaration** must specify (implicitly or explicitly) the underlying size of the enum </font>

```C++
// forward declaration of unscoped enum named intValues
enum intValues : unsigned long long; // unscoped must specify a type because no default
enum class open_modes; // scoped enums can use int by default
```



## Pointer to Class Member 

A pointer to member is a pointer that can point to a non-static member of a class identifies a member of a class, not an object of that class 

The type of a pointer to member embodies both the type of a class and the type of a member of that class 

Pointers to **static** members are ordinary pointers because static class members are not part of any object.

### Pointers to Data Members 

the pointer identifies a specific member but does not point to any data. 

```C++
class Screen {
public:
  typedef std::string::size_type pos;
  char get_cursor() const { return contents[cursor]; }
  char get() const;
  char get(pos ht, pos wd) const;
private:
  std::string contents;
  pos cursor;
  pos height, width;
};

// pdata can point to a string member of a const (or non const) Screen object
const string Screen::*pdata;
pdata = &Screen::contents; // error, Screen::contents is private
auto pdata = &Screen::contents; // error, Screen::contents is private

Screen myScreen, *pScreen = &myScreen;
// .* dereferences pdata to fetch the contents member from the object myScreen
auto s = myScreen.*pdata;
// ->* dereferences pdata to fetch contents from the object to which pScreen points
s = pScreen->*pdata;
```

<font color=red>a function that return a pointer to data member can give access to private member</font> and realize read-only for user and read/write inside class

```C++
class Screen {
public:
  // data is a static member that returns a pointer to member
  static const std::string Screen::*data()
    { return &Screen::contents; }
// other members as before
};

// ok, read-only access to private member Screen::contents
const string Screen::*pdata = Screen::data();
```

### Pointers to Member Functions

If the member function is a **const** member or a reference member, we must include the **const** or reference qualifier as well.

```C++
char (Screen::*pmf2)(Screen::pos, Screen::pos) const;
pmf2 = &Screen::get;

// without parentheses around Screen::* p is nonmember function 
// error: nonmember function cannot have a const qualifier
char Screen::*p(Screen::pos, Screen::pos) const;
```

<font color=red>Unlike ordinary function pointers, there is no automatic conversion between a member function and a pointer to that member: </font>

```C++
// pmf points to a Screen member that takes no arguments and returns char
pmf = &Screen::get; // must explicitly use the address-of operator
pmf = Screen::get; // error: no conversion to pointer for member functions
```



```C++
Screen myScreen,*pScreen = &myScreen;
// call the function to which pmf points on the object to which pScreen points
char c1 = (pScreen->*pmf)();
// passes the arguments 0, 0 to the two-parameter version of get on the object myScreen
char c2 = (myScreen.*pmf2)(0, 0);
// Without the parentheses, myScreen.*pmf() is equivalent to 
myScreen.*(pmf())
```

As with any other function pointer, we can use a pointer-to-member function type as the return type or as a parameter type in a function. Like any other parameter, a pointer-to-member parameter can have a default argument: 

#### Pointer-to-Member Function Tables 

One common use for function pointers and for pointers to member functions is to store them in a function table 

```C++
class Screen {
public:
  // other interface and implementation members as before
  Screen& home(); // cursor movement functions
  Screen& forward();
  Screen& back();
  Screen& up();
  Screen& down();
  // Action is a pointer that can be assigned any of the cursor movement members
  using Action = Screen& (Screen::*)();
  // specify which direction to move; enum see § 19.3 (p. 832)
  enum Directions { HOME, FORWARD, BACK, UP, DOWN };
  Screen& move(Directions);
private:
  static Action Menu[]; // function table
};

Screen::Action Screen::Menu[] = { &Screen::home,
								&Screen::forward,
								&Screen::back,
								&Screen::up,
								&Screen::down,
};

Screen& Screen::move(Directions cm)
{
  // run the element indexed by cm on this object
  return (this->*Menu[cm])(); // Menu[cm] points to a member function
}

myScreen.move(Screen::HOME); // invokes myScreen.home
myScreen.move(Screen::DOWN); // invokes myScreen.down
```

#### Using Member Functions as Callable Objects 

a pointer to member is not a callable object; but can obtain a callable from a pointer to member function  using the library template ***function***、***mem_fn*** or ***bind***



## Nested Classes 

A class defined within another class is a nested class, also referred to as a nested type. Nested classes are most often used to define implementation classes 

Normal name lookup rules apply for nested class 

<font color=red>Nested classes are independent classes and are largely unrelated to their enclosing class </font>

<font color=red>A nested class defines a type member in its enclosing class. As with any other member, the enclosing class determines ***public/protected/private*** access to this type </font>



## union: A Space-Saving Class 

A union is a special kind of class. A union may have multiple data members, but at any point in time, only one of the members may have a value 

The amount of storage allocated for a union is at least as much as is needed to contain its largest data member. 

<font color=red>Like any class, a union defines a new type. By default, like structs, members of a union are ***public***. </font>
But a union cannot have a reference type member, may not inherit from another class, may not be inherited. As a result, a union may not have virtual functions 

```C++
// objects of type Token have a single member, which could be of any of the listed types
union Token {
  // members are public by default
  char cval;
  int ival;
  double dval;
};
```

Like the built-in types, by default unions are uninitialized. If an initializer is present, it is used to initialize the first member 

```C++
Token first_token = {'a'}; // initializes the cval member
Token last_token; // uninitialized Token object
Token *pt = new Token; // pointer to an uninitialized Token object
```



### Anonymous unions 

An **anonymous union** is an **unnamed union** that does not include any declarations between the close curly that ends its body and the semicolon that ends the union definition. 

An **anonymous union** cannot have private or protected members, nor can an anonymous union define member functions. 

When we define an **anonymous union** the compiler automatically creates an **unnamed object** of the newly defined union type, and members of the **unnamed object** are directly accessible in the scope where the **anonymous union** is defined 

```C++
union { // anonymous union
  char cval;
  int ival;
  double dval;
}; // defines an unnamed object, whose members we can access directly
cval = 'c'; // assigns a new value to the unnamed, anonymous union object
ival = 42; // that object now holds the value 42
```



### unions with Members of Class Type 

If a union member’s type defines default constructor or one of the copy-control members, the compiler synthesizes the corresponding member of the union as deleted.

If a class has a union member that has a deleted copy-control member, then that corresponding copy-control operation(s) of the class itself will be deleted as well 

#### Using a Class to Manage union Members 

Because of the complexities involved in constructing and destroying members of class type, unions with class-type members ordinarily are embedded inside another class. And usually define a member of a nested, unnamed, unscoped enumeration, known as a discriminant, to keep track of the type of its union member. 

```C++
class Token {
public:
  // copy control needed because our class has a union with a string member
  // defining the move constructor and move-assignment operator is left as an exercise
  Token(): tok(INT), ival{0} { }
  Token(const Token &t): tok(t.tok) { copyUnion(t); }
  Token &operator=(const Token&);
  // if the union holds a string, we must destroy it; see § 19.1.2 (p. 824)
  ~Token() { if (tok == STR) sval.~string(); }
  // assignment operators to set the differing members of the union
  Token &operator=(const std::string&);
  Token &operator=(char);
  Token &operator=(int);
  Token &operator=(double);
private:
  enum {INT, CHAR, DBL, STR} tok; // discriminant
  union { // anonymous union
    char cval;
    int ival;
    double dval;
    std::string sval;
  }; // each Token object has an unnamed member of this unnamed union type
  // check the discriminant and copy the union member as appropriate
  void copyUnion(const Token&);
};

Token &Token::operator=(int i)
{
  if (tok == STR) sval.~string(); // if we have a string, free it
  ival = i; // assign to the appropriate member
  tok = INT; // update the discriminant
  return *this;
}

Token &Token::operator=(const std::string &s)
{
  if (tok == STR) // if we already hold a string, just do an assignment
    sval = s;
  else
    new(&sval) string(s); // otherwise construct a string
  tok = STR; // update the discriminant
  return *this;
}

void Token::copyUnion(const Token &t)
{
  switch (t.tok) {
    case Token::INT: ival = t.ival; break;
    case Token::CHAR: cval = t.cval; break;
    case Token::DBL: dval = t.dval; break;
    // to copy a string, construct it using placement new; see (§ 19.1.2 (p. 824))
    case Token::STR: new(&sval) string(t.sval); break;
  }
}

Token &Token::operator=(const Token &t)
{
  // if this object holds a string and t doesn't, we have to free the old string
  if (tok == STR && t.tok != STR) sval.~string();
  if (tok == STR && t.tok == STR)
    sval = t.sval; // no need to construct a new string
  else
    copyUnion(t); // will construct a string if t.tok is STR
  tok = t.tok;
  return *this;
}
```

## Local Classes 

A class defined inside a function body is called a **local class**. The **local class** could make the **enclosing function a friend**.
A local class defines a type that is visible only in the scope in which it is defined.
Unlike nested classes, local class is severely restricted 

1. All members, including functions, of a local class must be completely defined inside the class body 
2. a local class is not permitted to declare static data members, there being no way to define them 
3. local class can access only type names, static variables, and enumerators defined within the enclosing local scopes, and may not use the ordinary local variables of the function in which the class is defined: 

More typically, a local class defines its members as ***public***. 

```C++
int a, val;
void foo(int val)
{
  static int si;
  enum Loc { a = 1024, b };
  // Bar is local to foo
  struct Bar {
    Loc locVal; // ok: uses a local type name
    int barVal;
    void fooBar(Loc l = a) // ok: default argument is Loc::a
    {
      barVal = val; // error: val is local to foo
      barVal = ::val; // ok: uses a global object
      barVal = si; // ok: uses a static local object
      locVal = b; // ok: uses an enumerator
     }
  };
  // . . .
}
```

### Nested Local Classes 

A class nested in a local class is itself a local class, with all the attendant restrictions 

the definition of nested class inside a local class can appear outside the local-class body, however, must be defined in the same scope as that in which the local class is defined 

```C++
void foo()
{
  class Bar {
  public:
    // ...
    class Nested; // declares class Nested
  };
  // definition of Nested
  class Bar::Nested {
  // ...
  };
}
```

## Inherently Nonportable Features 

C++ defines some machine specific. nonportable features to support low-level programming, 

### Bit-fields 

inherits from C. The memory layout of a bit-field is machine dependent. 

A class can define a (nonstatic) data member as a bit-field 
The address-of operator (&) cannot be applied to a bit-field 

A bit-field must have integral or enumeration type. Ordinarily, use an unsigned type to hold a bit-field, because the behavior of a signed bit-field is implementation defined. 

```C++
typedef unsigned int Bit;
class File {
  Bit mode: 2; // mode has 2 bits
  Bit modified: 1; // modified has 1 bit
  Bit prot_owner: 3; // prot_owner has 3 bits
  Bit prot_group: 3; // prot_group has 3 bits
  Bit prot_world: 3; // prot_world has 3 bits
  // operations and data members of File
public:
  // file modes specified as octal literals; see § 2.1.3 (p. 38)
  enum modes { READ = 01, WRITE = 02, EXECUTE = 03 };
  File &open(modes);
  void close();
  void write();
  bool isRead() const;
  void setWrite();
};
```

Whether and how the bits are packed into the integer is machine dependent 

### ***volatile*** Qualifier 

inherits from C.

The precise meaning of volatile is inherently machine dependent and can be understood only by reading the compiler documentation.

The volatile qualifier is used in much the same way as the const qualifier. 

One important difference between the treatment of const and volatile is that the synthesized copy/move and assignment operators cannot be used to initialize or assign from a volatile object. 

A type can be both const and volatile 

member functions can be defined as volatile. Only volatile member functions may be called on volatile objects. 

```C++
volatile int v; // v is a volatile int
int *volatile vip; // vip is a volatile pointer to int
volatile int *ivp; // ivp is a pointer to volatile int
// vivp is a volatile pointer to volatile int
volatile int *volatile vivp;
int *ip = &v; // error: must use a pointer to volatile
*ivp = &v; // ok: ivp is a pointer to volatile
vivp = &v; // ok: vivp is a volatile pointer to volatile
```



### Language linkage

Every **function type**, every **function name with external linkage**, and every **variable name with external linkage**, has a property called **language linkage**. 

Language linkage encapsulates the set of requirements necessary to link with a program unit written in another programming language: calling convention, name mangling (name decoration) algorithm, etc. 

It is worth noting that the parameter and return types in functions that are shared across languages are often constrained. 

Since language linkage is part of every function type, a pointer to a C function does not have the same type as a pointer to a C++ function 

```C++
void (*pf1)(int); // points to a C++ function
extern "C" void (*pf2)(int); // points to a C function
pf1 = pf2; // error: pf1 and pf2 have different types
```

Only two language linkages are guaranteed to be supported:

1. "C++", the default language linkage.
2. "C", which makes it possible to link with functions written in the C programming language, and to define, in a C++ program, functions that can be called from the units written in C.

**Language specifications** can only appear in **namespace scope**.
The braces of the language specification do not establish a scope.
When language specifications nest, the **innermost specification** is the one that is in effect.

A function can be re-declared without a linkage specification after it was declared with a language specification, the second declaration will reuse the first language linkage. <font color=red>This allows to use linkage specification with function declaration only in header files and source files just include the headers, need not redeclare linkage specification</font>. **The opposite is not true**: if the first declaration has no language linkage, it is assumed "C++", and redeclaring with another language is an error.

```C++
// illustrative linkage directives that might appear in the C++ header <cstring>
// single-statement linkage directive
extern "C" size_t strlen(const char *);
// compound-statement linkage directive
extern "C" {
  int strcmp(const char*, const char*);
  char *strcat(char*, const char*);
}
```

preprocessor support for the same source file to be compiled under both C and C++

```C++
#ifdef __cplusplus
// ok: we're compiling C++
extern "C"
#endif
int strcmp(const char*, const char*);
```

#### Overloaded Functions and Linkage Directives 

The interaction between linkage directives and function overloading depends on whether target language. supports overloaded functions.

While C language does not support function overloading, C linkage directive can be specified for only one function in a set of overloaded functions 

```C++
// the C function can be called from C and C++ programs
// the C++ functions overload that function and are callable from C++
extern "C" double calc(double);
extern SmallInt calc(const SmallInt&);
extern BigNum calc(const BigNum&);
```

