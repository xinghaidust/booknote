[TOC]

# Chapter 0. Motivation 

# Chapter 1. Compilers, Linkers, and Components 

The compiler is not required to diagnose this violation of **ODR (One Definition Rule)**, but the behavior is undefined.

compiler-dependent: weak and strong symbol rules

The linker incorporates all of .o files supplied into the resulting executable image regardless of whether it is needed, whereas .o files supplied via a **library archive** are incorporated only if needed 

avoid any file-scope or namespace-scope variables requiring runtime initialization  

always better to avoid **mutual dependencies** among any entities — logical or physical -- because of inability to implement, test, and use physically cyclic softwares/libraries hierarchically. If have to, package mutually dependent entities together 

## Declarations, Definitions, and Linkage 

### Declaration vs. Definition 

many definitions are also **self-declaring** while others, such as <font color=red>those residing outside of the namespace in which they must also be separately declared (e.g., int ns::x;), are not</font>

### (Logical) Linkage vs. (Physical) Linking 

Linking refers to the physical act of binding the use of a name to its definition (or meaning)

Linkage is used to determine whether a given declaration and definition refer to the same logical entity 

1. External Linkage — The entity that a declaration of a name denotes can be defined in a different
   translation unit or scope (or both).
2. Internal Linkage — The entity that a declaration of a name denotes can be defined in a different
   scope but not in a separate translation unit.
3. No Linkage — The entity that a declaration of a name denotes cannot be defined in a different
   scope or translation unit. 

bindage, physical meaning of “linkage” :

1. A C++ construct has **internal bindage** if use of a declared name of that kind of construct and its corresponding definition (or meaning) is always effectively bound at compile time 
2. A C++ construct has **external bindage** if a corresponding definition must not appear in more than one translation unit of a program (e.g., in order to avoid multiply-defined-symbol errors at link time). 
3. A C++ construct has **dual bindage** if a corresponding definition can safely appear in more than one translation unit of the same program, such as **inline functions** and **implicitly instantiated function templates**, and may generally be bound at either compile or link time  

**symbol definition** for the entity is placed in a translation unit when static storage duration entity is defined 
**symbol reference** for the entity is placed in a translation unit when static storage duration entity is not bound at compile time

**ODR** permits more than one textual occurrence of the definition of the same entity in a program and the program behaves as if there were just a single physical definition of the entity if

1. the source for each definition appears **at most once** in any **translation unit**,  
2. each occurrence of the definition means the same thing (i.e., has the same token sequence, all the tokens are interpreted in the same way, etc.), achieved through the use of header files  

### Not All Declarations Require a Definition to Be Useful 

By placing just the declaration of a class, rather than always #include-ing its definition, we can often substantially reduce unnecessary compile-time coupling  

### Inline Functions Are a Somewhat Special Case 

If the address of an inline function is needed, that address is required to be unique throughout the program.
Taking the address of a function declared ***inline*** might result in an out-of-line copy of the compiled function definition being placed in the current translation unit’s .o file 

### Function Templates and Explicit Specializations 

similarities and subtle distinctions between function overloading and explicit specialization.
for normal functions, the standard conversion rules apply, whereas for function templates, there must be an exact match 

### extern Templates 

***extern*** **template** declaration merely suppresses **generation of function definitions** into the calling translation unit’s .o file but doesn’t prohibit the caller’s compiler from generating the code ***inline*** if appropriate. 

### Namespaces 

declarations of entities within an **unnamed namespace** have **internal linkage** even if they uses **external bindage**  

use **struct** for scopes limited to a single physical module (component) and to use **namespace** for scopes that span them 

<font color=red>Because a definition using qualified-name syntax (named namespace or class) does not implicitly also serve as a declaration, the compiler will require that there be a corresponding declaration directly within the named scope and report an error if inconsistent</font>

```C++
namespace MyUtil {
  extern const float PI;
  void f(double);
  double g();
}

namespace MyUtil {
   int f(int) {} // fail to check the inconsistence
}

const double MyUtil::PI = 3.14; // succeed to check the inconsistence
int MyUtil::g() {} // succeed to check the inconsistence
```

<font color=red>namespace aliases can be used to qualify a name but (unlike the namespace itself) cannot be used to re-open one </font>

## Include Directives and Include Guards 

The quoted form ("") `#include` directive can be convenient and also serves two important purposes: 

1. Facilitating the building of simple programs that are not requiring the user to specify additional command-line arguments (e.g., -I. on Unix platforms) to identify the (current) directory containing
   the header files
2. Avoiding ambiguity when potentially nonunique header-file names are included locally 

For larger development efforts, prefer to angle brackets (< >) as opposed to double quotes ("") 

now-deprecated external (redundant) include guards helped to significantly reduce compile (preprocessing) time 

```C++
// my_box.h
#ifndef INCLUDED_MY_BOX // internal include guard
#define INCLUDED_MY_BOX

#ifndef INCLUDED_MY_POINT // external (redundant) include guard
#include <my_point.h>
#endif

// ...
#endif
```

## From .h /.cpp Pairs to Components 

use header files as a modular mechanism for sharing definitions having internal or dual bindage, such as classes, inline functions, templates, typedefs and preprocessor macros and also use headers for dispensing consistent declarations of shared external-bindage constructs such as functions (or variables) having static storage duration. 

.h /.cpp pairs becomes ***components***, atomic units of physical design, if it satisfies 

1. .cpp file incorporates its corresponding .h file as the first substantive line of code, that enables the compiler to help ensure consistency between interface and implementation (at compile time). 
2. Logical constructs having external linkage defined in a .cpp file are declared in the corresponding .h file. 
3. Logical constructs having external or dual bindage that are declared within the header file of a component, if defined at all, are defined within that component only 
4. Use #include-ed to obtain the needed declaration for all logical constructs having external or dual bindage instead of forward declarations  

C++ preprocessor #include directives alone are sufficient to deduce all actual physical dependencies among the components within a system, provided the system compiles. 

## logical relationships and physical dependency 

logical relationships imply physical dependency when these relationships extend across component boundaries:

1. **Is-A** induces a direct compile-time dependency requires placing a #include of the .h file defining the base class in the (distinct) .h file  
2. **Uses-In-The-Interface** implies a (possibly indirect) physical dependency but might not require #include-ing the used type’s definition in either the .h or the .cpp
3. **Uses-In-The-Implementation** induces a direct compile-time dependency that requires placing a #include of the .h defining the used type in either the .h or the .cpp
4. **Uses-In-Name-Only** indicates logical collaboration, but implies no physical dependency  

the component dependencies in a software subsystem should form a directed acyclic graph (DAG). If not we should eliminate the cyclic physical dependencies. 



# Chapter 2. Packaging and Design Rules 

What we are about to motivate and then present is a fairly small set of architectural design rules that collectively provide a proven physical framework for organizing component-based software (along with non-component-based legacy, open-source, and third-party software) in a way that enables effective design, development, testing, and deployment on an enterprise scale. 

## Physical Aggregate 

An aggregate is a cohesive physical unit of design comprising logical content and should be treated atomically for design purposes 

A **component** is the innermost level of physical aggregation 
A **unit of release** (UOR), which represents a physically (and usually also logically) cohesive collection of source code, is the outermost level of physical aggregation 

While in a UOR  .h files are naturally architecturally significant, .cpp files and their corresponding .o files are not 

### Allowed Dependencies 

The purpose of stating **allowed dependencies** is to be anticipatory, not reactive, that is, express physical design (intent) before any code is written.  

No Cyclic Physical Dependencies! 

### UOR levels

not only hierarchy, but also balance 

three appropriately balanced, architecturally significant levels of physical aggregation have been sufficient to represent very large libraries, more than three can be impractical due to the sheer magnitude of the code involve 

each entity at the second level within the UOR is a **package** and the UOR itself is a **package group**

There Should Be Nothing Architecturally Significant Larger Than a UOR 

### Architecturally Significant Names Must Be Unique 

The benefit of unique filenames is uniqueness in log message, an assertion, an email, or a tab in a text editor 

## Logical/Physical Coherence 

**logical/physical coherence**: all logical constructs within the collective interface of a physical module or aggregate are implemented directly within that module 

logical and physical coherence along with acyclic physical dependencies is absolutely essential for every level of software.

### Logical and Physical Name Cohesion 

design rules

1. Component files (.h /.cpp) must have the same root name as that of the component itself 
2. Every component must reside within a package 
3. The (all-lowercase) name of each component must begin with the (all-lowercase) name of the package to which it belongs, followed by an underscore (_). 
4. Each logical entity declared within a component must be nested within a namespace having the name of the package in which that entity resides. 
5. Each package namespace must be nested within a unique enterprise-wide namespace. 
6. The name of every logical construct declared at package-namespace scope — other than free operator and aspect functions (such as operator== and swap) — must have a prefix,  
7. Only classes, structs, and free operator functions (and operator-like aspect functions, e.g., swap) are permitted to be declared at package-namespace scope in a component’s .h file 

making all component filenames themselves unique by design is much more robust and flexible than using relative directory paths (e.g., `#include <bts/date.h>`) because a perhaps very large and sparsely populated directory structure has to be replicated when select an arbitrary set of specific components from a vast repository 

> <font color=red>Having modifiable global variables at namespace scope is simply a bad idea. Nesting such variables within a class as static data members and providing only functional access is also generally a bad idea </font>

<font color=red>advantages of using a ***struct*** over a third level of namespace for aggregating related functions into a single utility component:</font>

1. a struct does not permit using directives (or declarations) to import function names into the current namespace 
2. a struct can support private nested data  
3. a struct can be passed as a template parameter 
4. a C-style function in a struct does not participate in ADL, avoiding potentially large overload sets

generally prefer to avoid **private inheritance**, in favor of layering (a.k.a. **composition**), and explicit (inline) function forwarding

# Chapter 3. Physical Design and Factoring 

Iterators often dramatically reduce what would otherwise be considered manifestly primitive operations 

Avoid defining more than one public class within a single component unless there is a compelling engineering reason such as Friendship、Cyclic Dependency、

## Levelization Techniques 

### Escalation 

> Moving mutually dependent functionality higher in the physical hierarchy. 

```c++
// rectangle.h
#include <box.h>
class Rectangle {
  Rectangle(const Box& value);
};

// box.h
#include <rectangle.h>
class Box {
  Box(const Rectangle& value);
};
```

Whether or not to partition functionality at this level of granularity is a matter of engineering judgment.  

```C++
// convertutil.h
// ...
#include <rectangle>
#include <box.h>
#include <point.h>
struct ConvertUtil {
  static Box boxFromRectangle(const Rectangle& value);
  static Rectangle rectangleFromBox(const Box& value);
};

// rectangle.h
#include <point.h>
class Rectangle {
  // ...
};

// box.h
#include <point.h>
class Box {
  // ...
};
```



### Demotion 

> Moving common functionality lower in the physical hierarchy 

```C++
class EventQueue {
  List<Event> d_events;
  // common event info.
  // other stuff
public:
  // ...
};

class Event {
  EventQueue *d_common_p;
  // event-specific info.
public:
  // ...
};
```

factor the common part into a separate class

```C++
class EventQueue {
  List<Event> d_events;
  CommonEventInfo d_info;
  // other stuff
public:
  // ...
};

class Event {
  CommonEventInfo *d_common_p;
  // event-specific info.
public:
  // ...
};

class CommonEventInfo {
  // common event info.
public:
  // ...
};
```



### Opaque Pointers
>  Having an object use another in name only 

```C++
// my_manager.h
#include <my_employee.h>
#include <vector>
class Manager {
  std::vector<Employee> d_staff;
public:
  int addEmployee(/*...*/);
  const Employee& employee(int id) const;
  int numStaff() const;
// ...
};

// my_employee.h
#include <my_manager.h>
class Employee {
  Manager *d_boss_p;
public:
  Employee(Manager *boss /* , ...*/);
  int numStaff() const;
};

// client.h
#include <my_employee.h>
inline
int askEmployeeNumStaff(const Employee& employee)
{
  return employee.numStaff();
}
```



```C++
// my_manager.h
#include <my_employee.h>
#include <vector>
class Manager {
  std::vector<Employee> d_staff;
public:
  int addEmployee(/*...*/);
  const Employee& employee(int id) const;
  int numStaff() const;
  // ...
};

// my_employee.h
class Manager;
class Employee {
  Manager *d_boss_p;
public:
  Employee(Manager *boss /* , ...*/);
  const Manager *manager() const { return d_boss_p; }
};

// client.h
#include <my_employee.h>
#include <my_manager.h>
inline
int askEmployeeNumStaff(const Employee& employee)
{
  return employee.manager()->numStaff();
}
```



### Dumb Data
>  Using data that indicates a dependency on a peer object, but only in the context of a separate,
> higher-level object 



```C++
// my_node.h
#include <vector>
class Edge;
class Node {
    std::vector<Edge *> d_edges;
public:
    Node();
    void appendEdge(Edge *edge);
    Edge *edge(int edgeIndex);
    int numEdges() const;
    const Edge *edge(int edgeIndex) const;
};

// my_edge.h
class Node;
class Edge {
double d_weight; // edge data
    Node *d_head_p;
    Node *d_tail_p;
public:
    Edge(Node *head, Node *tail);
    void setWeight(double);
    double weight() const;
    const Node *head() const;
    const Node *tail() const;
};

// my_graph.h
#include <my_node.h>
#include <my_edge.h>
#include <list>
class Graph {
    std::list<Node> d_nodes;
    std::list<Edge> d_edges;
public:
    Graph();
    const Node *appendNode();
    Edge *appendEdge(const Node *tail,
                     const Node *head,
                     double weight = 0.0);
    void setWeight(const Edge *edge, double weight);
    int numNodes() const;
    int numEdges() const;
};
```

an alternative to opaque pointers but more type-safe 

```C++
// my_node.h
#include <vector>
class Node {
    std::vector<int> d_edgeIds;
public:
    Node();
    void appendEdge(int edgeId);
    int numEdges() const;
    int edgeId(int edgeIndex) const;
};

// my_edge.h
class Edge {
    double d_weight; // edge data
    int d_headId;
    int d_tailId;
public:
    Edge(int headId, int tailId);
    void setWeight(double);
    double weight() const;
    int headId() const;
    int tailId() const;
};

// my_graph.h
#include <my_node.h>
#include <my_edge.h>
#include <vector>
class Graph {
    std::vector<Node> d_nodes;
    std::vector<Edge> d_edges;
public:
    Graph();
    int appendNode();
    int appendEdge(int tailId, int headId, double weight = 0.0);
    Edge *edge(int edgeId);
    int numNodes() const;
    int numEdges() const;
    const Node *node(int nodeId) const;
    const Edge *edge(int edgeId) const;
};
```



### Redundancy
>  Deliberately avoiding reuse by repeating small amounts of code or data to avoid coupling 

<font color=red>Preferring demotion over redundancy for avoiding dependency </font>

### Callbacks
> Client-supplied functions that enable (typically lower-level) subsystems to perform specific
> tasks in a more global context. 

different flavors of callbacks used for levelization purposes 

1. Data Passing the address of a modifiable object
2. Function Passing the address of a (stateless) function
3. Functor Supplying a (possibly stateful) invocable (“function”) object
4. Protocol Passing the address of a (typically) stateful concrete object
accessed via the interface of a pure abstract base class
5. Concept Supplying a (typically) stateful concrete object satisfying both
structural and semantic requirements 

#### 1. Data Callbacks 

decoupling  via result-object addresses 

```C++
// my_manager.h
#include <my_employee.h>
#include <vector>
class Manager {
    std::vector<Employee> d_staff;
    int d_numStaff = 0; // New (possibly redundant) data member
public:
    int addEmployee(/*...*/);
    const Employee& employee(int id) const;
};

// my_employee.h
class Employee {
    const int *d_addrNumStaff_p; //
public:
    Employee(const int *addrNumStaff /* , ...*/);
    int numStaff() const { return *d_addrNumStaff_p; }
};

// client.h
#include <my_employee.h>
inline
int askEmployeeNumStaff(const Employee& employee)
{
	return employee.numStaff();
}
```

#### 2. Function Callbacks 

This basic callback technique, originally implemented using C-style functions, is now often much more effectively implemented using modern, modular, and type-safe functors. 

#### 3. Functor Callbacks 

a **functor** is an object and, that can have state and supports one or more overloads of `operator()`

#### 4. Protocol Callbacks 

```C++
// my_dealer.h
class Deck;
class Rules;
class Player;
class Dealer {
public:
    void addPlayer(
    const Player *p,
    int cash);
    void dealCards();
    void shuffleCards();
    const Deck& deck() const;
    const Rules& rules() const;
};

// my_player.h
class Action;
class Dealer;
class Player {
public:
    Player(const char *name);
    int bet(const Dealer&
    game) const;
    Action choice(const
    Dealer& game) const;
    const char *name() const;
};

// my_dealer.cpp
#include <my_dealer.h>
#include <my_player.h>
// ...

// my_player.cpp
#include <my_player.h>
#include <my_dealer.h>
// ...
```

mutually dependency becomes dependency on abstract interface 

```C++
// player.h
// ...
class Action;
class Dealer;
class Player {
public:
    virtual ~Player();
    virtual int bet(const Dealer& game) const = 0;
    virtual Action choice(const Dealer& game) const = 0;
    virtual const char *name() const = 0;
};

// my_dealer.cpp
#include <my_dealer.h>
#include <player.h>
// ...

// my_player.cpp
#include <player.h>
#include <my_dealer.h>
#include <my_player.h>
// ...
```



#### 5. Concept Callbacks 

The union of requirements on a type for it to be viable for use in such a context has come to be known as a **concept**  

Relying on concept with template rather than on an abstract base class immediately gives up any necessary
physical dependency and  runtime overhead of a virtual function call  

### Manager Class
>  Establishing a class that owns and coordinates lower-level objects 

essentially just a practical design strategy 

The classical example is that of a singly linked list `Link<T>` managed by `List<T>`

### Factoring
> Moving independently testable sub-behavior out of the implementations of complex components involved in excessive physical coupling 

### Escalating Encapsulation
> Moving the point at which implementation details are hidden from clients to a higher level
> in the physical hierarchy. 

encapsulating wrapper, sometimes called a facade 

create a multicomponent facade that wraps each of these types individually in separate components and then provide only the subset of their respective functionalities that are desirable to public clients. 

## Architectural Insulation Techniques  

An implementation detail of a component (type, data, or function) that can be altered, added, or removed without forcing clients to rework their code is said to be **encapsulated**  

An implementation detail of a component (type, data, or function) that can be altered, added, or removed without forcing clients to **recompile** is said to be **insulated**  

### 1. The Pure Abstract Interface (“Protocol”) Class
Deriving from (or adapting to) a pure interface, where each concrete implementation is instantiated separately from typical clients that use it
### 2. The Fully Insulating Concrete Wrapper Component
Creating one or more classes within a single component that each have exactly one data member, which is an opaque pointer to an instance of a unique component-local class defined in the implementation (.cpp) file and managed entirely by its corresponding public “wrapper” class
### 3. The Procedural Interface
Providing a collection of free functions that allocate, operate on, and deallocate opaque objects via strongly typed pointers  