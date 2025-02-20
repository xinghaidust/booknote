[TOC]

# [Core Guideline](https://isocpp.github.io/CppCoreGuidelines/)

# Chapter 1. The Art of Software Design  

software design is the most complicated part of software development. Therefore,this chapter explains several software design principles that will help you to stay on the right path  

## Guideline 1: Understand the Importance of Software Design 

The **overall structure** is the design of a project, it plays more central role in the success of a project than any **feature**

<font color=red>**Software design** is the art of managing interdependencies between software components</font>. It aims at minimizing artificial (technical) dependencies and introduces the necessary abstractions and compromises 

### The Three Levels of Software Development

***Software Architecture*** (**architectural patterns** such as client-server architecture, microservices, and so on  、***Software Design*** and ***Implementation Details***

The boundary between ***software design*** and ***software architecture*** is fluid 

***Software Architecture*** represents the overall strategy of software approach, whereas ***Software Design*** is the tactics to make the strategy work

If software architecture and software design are of such importance, why C++ community focus so strongly on **features**:

1. need to develop a sense of idiomatic C++ with a common understanding on which use is good
   and which use is bad 
2. misunderstanding and wrong expectations on features 
3. despite the huge amount of features and their complexity, in comparison to the complexity of software design, the complexity of C++ features is small 

### Idiom 

An idiom is a commonly used language-specific solution for a recurring problem, either an **implementation pattern** or a **design pattern** 

In C++, most idioms fall into the category of **implementation details**, such as **copy-and-swap idiom**、 **RAII idiom** while others fall into the category of **Software Design** such as the **Non-Virtual Interface (NVI) idiom** based on the **Template Method design pattern** and the **Pimpl idiom** based on the **Bridge design pattern**

RAII is **implementation details idiom** because it separates resource management and business logic by means of **encapsulation** rather than by means of **decoupling, i.e., abstraction**

While **abstraction** solves the problems and issues that arise at the **Software Design level**, **encapsulation** solves the problems and issues that arise at the **Implementation Details level**

## Guideline 2: Design for Change 

One of the essential expectations for good software is its ability to change easily.   

###  Separation of Concerns

One of the best and proven solutions to reduce artificial dependencies and simplify change is to separate concerns, just as ***Single-Responsibility Principle (SRP)***:  

> Everything should do just one thing  

***Don’t Repeat Yourself (DRY) principle***, advises to not duplicate some key information in many places

In ***Unified Modeling Language (UML)*** **high level** refers to stable parts of architecture, and **low level** refers to the aspects that change more often.  

<font color=red>**design patterns** are general and practical approach to structure software to separate concerns  </font>

### You Aren’t Gonna Need It

***YAGNI principle (You Aren’t Gonna Need It)***, warns about **overengineering**  

**Separation of concerns** are **tools for better maintainability and simplifying change** but not goals so avoid premature separation of concerns until you have a clear idea about what kind of change to expect and then refactor the code to make the change as easy as possible.  

## Guideline 3: Separate Interfaces to Avoid Artificial Coupling  
**Interface Segregation Principle (ISP) **is a special case of the ***SRP*** focusing on **decoupling interfaces** 

> Clients should not be forced to depend on methods that they do not use.

## Guideline 4: Design for Testability  

software is expected to change, but with the risk of breaking something, And test protects you from accidentally breaking things  

Use a **black box test** instead of a **white box test** which will introduces dependencies between test code and the details of your production code.  

### How to Test a Private Member Function  

1. Calling another member function which triggers the tested private function <font color=red>(**white box test**)</font>
1.  Befriend the test class  <font color=red>(introduce an artificial dependency   )</font>

The true solution is to consider private member functions that need testing to be misplaced and extract it a separate entity  

generally prefer **nonmember non-friend functions** to **member functions** because every member function has full access to every member of a class, therefore **less encapsulated**.  

## Guideline 5: Design for Extension  

***OpenClosed Principle (OCP)***: but <font color=red>no way to prevent modifications on the lower levels  </font>

> Software artifacts (classes, modules, functions, etc.) should be open for extension **(just add new code)**, but closed for modification **(not modify existing code on the same architectural level
> or on higher levels)**.  

**Standard Library** is designed for extensibility by **function overloading**, **templates**, and **template specialization** rather than **base classes**

should also not prematurely design for extension unless you have a good idea about how your code will evolve

# Chapter 2. The Art of Building Abstractions  

**Abstractions** play a vital role to **managing complexity** in **software design and software architecture**.
Building and using abstractions is surprisingly difficult and comes with a lot of subtleties, and therefore feels more like an art than a science. 

This chapter goes into detail about the meaning of abstractions and the art of building them.  

## Guideline 6: Adhere to the Expected Behavior of Abstractions  
<font color=red>**abstractions** represent a set of **requirements and expectations**</font>

***Liskov Substitution Principle*** formulates **IS-A relationship**, i.e., the expectations in an abstraction, must be adhered to in a subtype:

> Subtype Requirement: Let φ(x) be a property provable about objects x of type T. Then φ(y) should be true for objects y of type S where S is a subtype of T.  

In details:

1. Preconditions cannot be strengthened in a subtype  
2. Postconditions cannot be weakened in a subtype  
3. Function return types in a subtype must be covariant
4. Function parameters in a subtype must be contravariant, <font color=red>no direct language support in C++ </font>
5. Invariants of the super type must be preserved in a subtype  

The following code which introduces a dependency in the behavior of the derived types, should always be considered an LSP violation and very bad practice  

```C++
class Base { /*...*/ };
class Derived : public Base { /*...*/ };
class Special : public Base { /*...*/ };
// ... Potentially more derived classes
void f( Base const& b )
{
    if( dynamic_cast<Special const*>(&b) )
    {
    	// ... do something "special," knowing that 'Special' behaves differently
    }
    else
    {
    	// ... do the expected thing
    }
}
```

## Guideline 7: Understand the Similarities Between Base Classes and Concepts  

***Liskov Substitution Principle (LSP)*** not limited to dynamic (runtime) polymorphism and inheritance hierarchies, but also static (compile-time) polymorphism and templated code  

Since ***concepts*** represent an **LSP abstraction**, i.e., a set of requirements and expectations, they are the **static equivalent** of **base classes** from a **semantic point of view** and subject to the ***Interface Segregation Principle (ISP)*** as well  

```C++
class Document
{
public:
    // ...
    virtual ~Document() = default;
    virtual void exportToJSON( /*...*/ ) const = 0;
    virtual void serialize( ByteStream&, /*...*/ ) const = 0;
    // ...
};
void useDocument( Document const& doc )
{
    // ...
    doc.exportToJSON( /*...*/ );
    // ...
}
//==== Code Snippet 2 ====
template< typename T >
concept Document =
requires( T t, ByteStream b ) {
    t.exportToJSON( /*...*/ );
    t.serialize( b, /*...*/ );
};
template< Document T >
void useDocument( T const& doc )
{
    // ...
    doc.exportToJSON( /*...*/ );
    // ...
}
```

## Guideline 8: Understand the Semantic Requirements of Overload Sets  
In addition to concepts, **function overloading** by means of **free functions** is a compile-time abstraction mechanism as well.

free functions perfectly lives up to the spirit of the ***Open-Closed Principle (OCP)***  
free functions are a very elegant technique to separate concerns, fulfilling the ***Single-Responsibility Principle (SRP)***.   
But  it might not always be clear what the expected behavior is, especially for an overload set that is scattered across a big codebase

## Guideline 9: Pay Attention to the Ownership of Abstractions  
***Dependency Inversion Principle (DIP)***

> The most flexible systems are those in which source code dependencies refer only to abstractions, not to concretions  

DIP is fulfilled by properly assigning ownership and by properly grouping the things that truly belong:

1. **abstractions** are owned by the **high level**, not by the **low level**  
2.  **low-level implementation details** depend on **high-level abstractions** and not the opposite

## Guideline 10: Consider Creating an Architectural Document  
**architectural document** is as important as **continuous Integration (CI) environment**、**automated tests** and **static code analysis tools**  

an **architectural document** is to help maintain and communicate the state of the architecture; and help avoid any misunderstandings.  

**architectural document** should primarily reflect the direction you codebase will evolve and a shared understanding of the system design such as the ideas, visions, and essential decisions. It shouldn’t contain the little details that indeed can change very often;  

# Chapter 3. The Purpose of Design Patterns  
## Guideline 11: Understand the Purpose of Design Patterns  
A design pattern:

1. Has a name
2. Carries an intent
3. Introduces an abstraction to decouple software entities   
4. Has been proven  

`std::make_unique()` is a recurring solution for a specific problem. It is a  implementation pattern but not a design pattern.

## Guideline 12: Beware of Design Pattern Misconceptions  
**design patterns** are tool to solve a design problem but not a goal and should not be overused

**design patterns** are not language-specific idioms but the problems they focus may be usual in **specific programming paradigm** such as ***Visitor and Prototype design patterns*** for **object-oriented programming**、***the Curiously Recurring Template Pattern [CRTP] and Expression Templates*** for **functional programming** or **generic programming**  

## Guideline 13: Design Patterns Are Everywhere  
any kind of abstraction and any attempt to decouple likely represents a known design pattern.  

Apply design patterns based on their intent whenever necessary  

## Guideline 14: Use a Design Pattern’s Name to Communicate Intent  
# Chapter 4. The Visitor Design Pattern  
## Guideline 15: Design for the Addition of Types or Operations  

For **static polymorphism** **types and operations** can be extended easily. 

While in **dynamic polymorphism**, one of the design axes (types and operations) needs to be
fixed to be **close set**.

***Acyclic Visitor*** can have both types and operations open sets but usually impractical because of performance 

### A Procedural Solution  

This kind of type-based programming has a long history in C  

**Procedural Solution** treats types as a **closed set** and operations as an **open set**, that is easy to add new operations, but weak for addition of types:

1. extend the enumeration and `switch statement` in the `drawAllShapes()` function  
2. recompile everything which has dependency of enumeration, such as all shape classes and functions on shape classes
   .   

```C++
//---- <Point.h> ----------------------------------------------------------------------------------
struct Point
{
   double x;
   double y;
};


//---- <Shape.h> ----------------------------------------------------------------------------------
enum ShapeType
{
   circle,
   square
};

class Shape
{
 protected:
   explicit Shape( ShapeType type )
      : type_( type )
   {}

 public:
   virtual ~Shape() = default;

   ShapeType getType() const { return type_; }

 private:
   ShapeType type_;
};

//---- <Circle.h> ---------------------------------------------------------------------------------

//#include <Point.h>
//#include <Shape.h>

class Circle : public Shape
{
 public:
   explicit Circle( double radius )
      : Shape( circle )
      , radius_( radius )
   {
      /* Checking that the given radius is valid */
   }

   double radius() const { return radius_; }
   Point  center() const { return center_; }

 private:
   double radius_;
   Point center_{};
};


//---- <DrawCircle.h> -----------------------------------------------------------------------------
class Circle;

void draw( Circle const& circle );


//---- <DrawCircle.cpp> ---------------------------------------------------------------------------

//#include <Circle.h>
//#include <DrawCircle.h>
//#include /* some graphics library */

void draw( Circle const& circle )
{
   // ... Implementing the logic for drawing a circle
}


//---- <Square.h> ---------------------------------------------------------------------------------

//#include <Shape.h>
//#include <Point.h>

class Square : public Shape
{
 public:
   explicit Square( double side )
      : Shape( square )
      , side_( side )
   {
      /* Checking that the given side length is valid */
   }

   double side  () const { return side_; }
   Point  center() const { return center_; }

 private:
   double side_;
   Point center_{};
};


//---- <DrawSquare.h> -----------------------------------------------------------------------------
class Square;

void draw( Square const& square );


//---- <DrawSquare.cpp> ---------------------------------------------------------------------------

//#include <DrawSquare.h>
//#include <Square.h>
//#include /* some graphics library */

void draw( Square const& square )
{
   // ... Implementing the logic for drawing a square
}


//---- <DrawAllShapes.h> --------------------------------------------------------------------------

#include <memory>
#include <vector>
class Shape;

void drawAllShapes( std::vector<std::unique_ptr<Shape>> const& shapes );


//---- <DrawAllShapes.cpp> ------------------------------------------------------------------------

//#include <DrawAllShapes.h>
//#include <Circle.h>
//#include <Square.h>

void drawAllShapes( std::vector<std::unique_ptr<Shape>> const& shapes )
{
   for( auto const& shape : shapes )
   {
      switch( shape->getType() )
      {
         case circle:
            draw( static_cast<Circle const&>( *shape ) );
            break;
         case square:
            draw( static_cast<Square const&>( *shape ) );
            break;
      }
   }
}
```

### An Object-Oriented Solution  

**Object-Oriented Solution** treats operations as a **closed set** and types as an **open set**, that is easy to add new types, but weak for addition of operations:

1. add a virtual function in base class
2.  implement the new function for all derived classes, even though the function might never be called
3. recompile all derived classes and functions on these classes

```C++
//---- <Point.h> ----------------------------------------------------------------------------------
struct Point
{
   double x;
   double y;
};


//---- <Shape.h> ----------------------------------------------------------------------------------
class Shape
{
 public:
   Shape() = default;

   virtual ~Shape() = default;

   virtual void draw() const = 0;
};


//---- <Circle.h> ---------------------------------------------------------------------------------
//#include <Point.h>
//#include <Shape.h>

class Circle : public Shape
{
 public:
   explicit Circle( double radius )
      : radius_( radius )
   {
      /* Checking that the given radius is valid */
   }

   double radius() const { return radius_; }
   Point  center() const { return center_; }

   void draw() const override;

 private:
   double radius_;
   Point center_{};
};


//---- <Circle.cpp> -------------------------------------------------------------------------------
//#include <Circle.h>
//#include /* some graphics library */

void Circle::draw() const
{
   // ... Implementing the logic for drawing a circle
}


//---- <Square.h> ---------------------------------------------------------------------------------
//#include <Point.h>
//#include <Shape.h>

class Square : public Shape
{
 public:
   explicit Square( double side )
      : side_( side )
   {
      /* Checking that the given side length is valid */
   }

   double side  () const { return side_; }
   Point  center() const { return center_; }

   void draw() const override;

 private:
   double side_;
   Point center_{};
};


//---- <Square.cpp> -------------------------------------------------------------------------------
//#include <Square.h>
//#include /* some graphics library */

void Square::draw() const
{
   // ... Implementing the logic for drawing a square
}


//---- <DrawAllShapes.h> --------------------------------------------------------------------------
#include <memory>
#include <vector>
class Shape;

void drawAllShapes( std::vector<std::unique_ptr<Shape>> const& shapes );


//---- <DrawAllShapes.cpp> ------------------------------------------------------------------------
//#include <DrawAllShapes.h>
//#include <Shape.h>

void drawAllShapes( std::vector<std::unique_ptr<Shape>> const& shapes )
{
   for ( auto const& shape : shapes )
   {
      shape->draw();
   }
}

```

## Guideline 16: Use Visitor to Extend Operations  
### The Visitor Design Pattern Explained  

**Visitor Design Pattern**

> Intent: “Represent an operation to be performed on the  elements of an object structure.
> Visitor lets you define a new operation without changing the classes of the elements on
> which it operates  

```C++
.//---- <ShapeVisitor.h> ---------------------------------------------------------------------------

class Circle;
class Square;

class ShapeVisitor
{
 public:
   virtual ~ShapeVisitor() = default;

   virtual void visit( Circle const& /*, ...*/ ) const = 0;
   virtual void visit( Square const& /*, ...*/ ) const = 0;
   // Possibly more visit() functions, one for each concrete shape
};


//---- <Shape.h> ----------------------------------------------------------------------------------
//#include <ShapeVisitor.h>

class Shape
{
 public:
   virtual ~Shape() = default;
   virtual void accept( ShapeVisitor const& v ) = 0;
};


//---- <Circle.h> ---------------------------------------------------------------------------------
//#include <Shape.h>

class Circle : public Shape
{
 public:
   explicit Circle( double radius )
      : radius_( radius )
   {
      /* Checking that the given radius is valid */
   }

   void accept( ShapeVisitor const& v ) override { v.visit(*this); }

   double radius() const { return radius_; }

 private:
   double radius_;
};

//---- <Square.h> ---------------------------------------------------------------------------------

//#include <Shape.h>

class Square : public Shape
{
 public:
   explicit Square( double side )
      : side_( side )
   {
      /* Checking that the given side length is valid */
   }

   void accept( ShapeVisitor const& v ) override { v.visit(*this); }

   double side() const { return side_; }

 private:
   double side_;
};


//---- <Draw.h> -----------------------------------------------------------------------------------

//#include <Circle.h>
//#include <ShapeVisitor.h>
//#include <Square.h>

class Draw : public ShapeVisitor
{
 public:
   void visit( Circle const& c /*, ...*/ ) const override
   {
      // ... Implementing the logic for drawing a Circle
   }

   void visit( Square const& s /*, ...*/ ) const override
   {
      // ... Implementing the logic for drawing a square
   }

   // Possibly more visit() functions, one for each concrete shape
};


//---- <DrawAllShapes.h> --------------------------------------------------------------------------

#include <memory>
#include <vector>
class Shape;

void drawAllShapes( std::vector<std::unique_ptr<Shape>> const& shapes );


//---- <DrawAllShapes.cpp> ------------------------------------------------------------------------
//#include <DrawAllShapes.h>
//#include <Draw.h>
//#include <Shape.h>

void drawAllShapes( std::vector<std::unique_ptr<Shape>> const& shapes )
{
   for( auto const& shape : shapes )
   {
      shape->accept( Draw{} );
   }
}
```

### Shortcomings of the Visitor Design Pattern  

1. low implementation flexibility since parameter list and return type `visit()` cannot change  functions for every derived class would be very similar,  
2. a **closed set of types** and in exchange provides an **open set of operations**. The underlying reason is that <font color=red>there is a **cyclic dependency**</font> `Shape->ShapeVistor->Square/Circle->Shape` also called ***Cyclic Visitor***
3. adding a new virtual function `accept()` to an existing hierarchy  
4. If add another layer of derived classes and forgets to override the `accept()` function, the **visitor** will be applied to the wrong type and no warning because of inheritance

    ```C++
    class CircleWrap : public Circle { using Circle::Circle;};
    CircleWrap(2.3)->accept(Draw()); // calls Draw::visit( Circle const& )
    ```
5. require to call two virtual functions and slow, quite common to OOP in general.  
6. this design pattern to be rather hard to fully understand and maintain  

## Guideline 17: Consider std::variant for Implementing Visitor  
From an architectural point of view, significant advantages compared to the ***classic Visitor*** 

1. there is no cyclic dependency between `std::variant` and its alternatives
2. far more efficient

### std::variant Solution  

```C++
//---- <Point.h> ----------------------------------------------------------------------------------
struct Point
{
   double x;
   double y;
};


//---- <Circle.h> ---------------------------------------------------------------------------------
//#include <Point.h>

class Circle
{
 public:
   explicit Circle( double radius )
      : radius_( radius )
   {
      /* Checking that the given radius is valid */
   }

   double radius() const { return radius_; }
   Point  center() const { return center_; }

 private:
   double radius_;
   Point center_{};
};


//---- <Square.h> ---------------------------------------------------------------------------------
//#include <Point.h>

class Square
{
 public:
   explicit Square( double side )
      : side_( side )
   {
      /* Checking that the given side length is valid */
   }

   double side  () const { return side_; }
   Point  center() const { return center_; }

 private:
   double side_;
   Point center_{};
};


//---- <Shape.h> ----------------------------------------------------------------------------------
//#include <Circle.h>
//#include <Square.h>
#include <variant>

using Shape = std::variant<Circle,Square>;


//---- <Shapes.h> ----------------
#include <vector>

using Shapes = std::vector<Shape>;


//---- <Draw.h> -----------------------------------------------------------------------------------
//#include <Circle.h>
//#include <Square.h>
//#include /* some graphics library */
class Draw
{
 public:
   void operator()( Circle const& c ) const
   {
      /* ... Implementing the logic for drawing a circle ... */
   }
   void operator()( Square const& s ) const
   {
      /* ... Implementing the logic for drawing a square ... */
   }
};


//---- <DrawAllShapes.h> --------------------------------------------------------------------------
//#include <Shapes.h>
void drawAllShapes( Shapes const& shapes );


//---- <DrawAllShapes.cpp> ------------------------------------------------------------------------
//#include <DrawAllShapes.h>
//#include <Draw.h>

void drawAllShapes( Shapes const& shapes )
{
   for( auto const& shape : shapes )
   {
      std::visit( Draw{}, shape );
   }
}
```

### Shortcomings of the std::variant Solution  

1.  similar to the **Visitor design pattern** and **procedural solution**: an **open set of operations** and a **closed set of types** and recompilation if update the variant
2. avoid putting types of very different sizes inside a variant. A solution would be to store pointers, via **Proxy objects**, or by using the ***Bridge design pattern*** with cost of performance  
3. create physical dependencies on the variant with the alternative types

## Guideline 18: Beware the Performance of Acyclic Visitor  
***Acyclic Visitor*** breaks the **cyclic dependency** of the ***Visitor design pattern*** using `dynamic_cast` but has significant performance disadvantages   

```C++
//---- <AbstractVisitor.h> ------------------------------------------------------------------------

class AbstractVisitor
{
 public:
   virtual ~AbstractVisitor() = default;
};


//---- <Visitor.h> --------------------------------------------------------------------------------

template< typename T >
class Visitor
{
 protected:
   virtual ~Visitor() = default;

 public:
   virtual void visit( T const& ) const = 0;
};


//---- <Shape.h> ----------------------------------------------------------------------------------

//#include <AbstractVisitor.h>

class Shape
{
 public:
   virtual ~Shape() = default;

   virtual void accept( AbstractVisitor const& v ) = 0;
};


//---- <Circle.h> ---------------------------------------------------------------------------------

//#include <Shape.h>
//#include <Visitor.h>

class Circle : public Shape
{
 public:
   explicit Circle( double radius )
      : radius_( radius )
   {
      /* Checking that the given radius is valid */
   }

   void accept( AbstractVisitor const& v ) override {
      if( auto const* cv = dynamic_cast<Visitor<Circle> const*>(&v) ) {
         cv->visit(*this);
      }
   }

   double radius() const { return radius_; }

 private:
   double radius_;
};


//---- <Square.h> ---------------------------------------------------------------------------------

//#include <Shape.h>
//#include <Visitor.h>

class Square : public Shape
{
 public:
   explicit Square( double side )
      : side_( side )
   {
      /* Checking that the given side length is valid */
   }

   void accept( AbstractVisitor const& v ) override {
      if( auto const* sv = dynamic_cast<Visitor<Square> const*>(&v) ) {
         sv->visit(*this);
      }
   }

   double side() const { return side_; }

 private:
   double side_;
};


//---- <Draw.h> -----------------------------------------------------------------------------------

//#include <AbstractVisitor.h>
//#include <Visitor.h>
//#include <Circle.h>
//#include <Square.h>

class Draw : public AbstractVisitor
           , public Visitor<Circle>
           , public Visitor<Square>
{
 public:
   void visit( Circle const& c ) const override
      { /* ... Implementing the logic for drawing a circle ... */ }
   void visit( Square const& s ) const override
      { /* ... Implementing the logic for drawing a square ... */ }
};


//---- <DrawAllShapes.h> --------------------------------------------------------------------------

#include <memory>
#include <vector>
class Shape;

void drawAllShapes( std::vector< std::unique_ptr<Shape> > const& shapes );


//---- <DrawAllShapes.cpp> ------------------------------------------------------------------------

//#include <DrawAllShapes.h>
//#include <Draw.h>
//#include <Shape.h>

void drawAllShapes( std::vector< std::unique_ptr<Shape> > const& shapes )
{
   for( auto const& shape : shapes )
   {
      shape->accept( Draw{} );
   }
}
```

# Chapter 5. The Strategy and Command Design Patterns  
## Guideline 19: Use Strategy to Isolate How Things Are Done  

### The Strategy Design Pattern Explained  

> Intent: “Define a family of algorithms, encapsulate each one, and make them interchangeable. Strategy lets the algorithm vary independently from clients that use it."

```C++
//---- <Shape.h> ----------------------------------------------------------------------------------

class Shape
{
 public:
   virtual ~Shape() = default;

   virtual void draw( /*some arguments*/ ) const = 0;
};


//---- <DrawStrategy.h> ---------------------------------------------------------------------------

template< typename T >
class DrawStrategy
{
 public:
   virtual ~DrawStrategy() = default;
   virtual void draw( T const& ) const = 0;
};


//---- <Circle.h> ---------------------------------------------------------------------------------

//#include <Shape.h>
//#include <DrawStrategy.h>
#include <memory>
#include <utility>

class Circle : public Shape
{
 public:
   using DrawCircleStrategy = DrawStrategy<Circle>;

   explicit Circle( double radius, std::unique_ptr<DrawCircleStrategy> drawer )
      : radius_( radius )
      , drawer_( std::move(drawer) )
   {
      /* Checking that the given radius is valid and that
         the given 'std::unique_ptr' is not a nullptr */
   }

   void draw( /*some arguments*/ ) const override
   {
      drawer_->draw( *this /*, some arguments*/ );
   }

   double radius() const { return radius_; }

 private:
   double radius_;
   std::unique_ptr<DrawCircleStrategy> drawer_;
};


//---- <Square.h> ---------------------------------------------------------------------------------

//#include <Shape.h>
//#include <DrawStrategy.h>
#include <memory>
#include <utility>

class Square : public Shape
{
 public:
   using DrawSquareStrategy = DrawStrategy<Square>;

   explicit Square( double side, std::unique_ptr<DrawSquareStrategy> drawer )
      : side_( side )
      , drawer_( std::move(drawer) )
   {
      /* Checking that the given side length is valid and that
         the given 'std::unique_ptr' is not a nullptr */
   }

   void draw( /*some arguments*/ ) const override
   {
      drawer_->draw( *this /*, some arguments*/ );
   }

   double side() const { return side_; }

 private:
   double side_;
   std::unique_ptr<DrawSquareStrategy> drawer_;
};


//---- <DrawAllShapes.h> --------------------------------------------------------------------------

#include <memory>
#include <vector>
class Shape;

void drawAllShapes( std::vector<std::unique_ptr<Shape>> const& shapes );


//---- <DrawAllShapes.cpp> ------------------------------------------------------------------------

//#include <DrawAllShapes.h>
//#include <Shape.h>

void drawAllShapes( std::vector<std::unique_ptr<Shape>> const& shapes )
{
   for( auto const& shape : shapes )
   {
      shape->draw( /*some arguments*/ );
   }
}


//---- <OpenGLCircleStrategy.h> ----------------

//#include <Circle.h>
//#include <DrawStrategy.h>
//#include /* OpenGL graphics library */

class OpenGLCircleStrategy : public DrawStrategy<Circle>
{
 public:
   explicit OpenGLCircleStrategy( /* Drawing related arguments */ )
   {}

   void draw( Circle const& circle /*, ...*/ ) const override
   {
      // ... Implementing the logic for drawing a circle by means of OpenGL
   }

 private:
   /* Drawing related data members, e.g. colors, textures, ... */
};


//---- <OpenGLSquareStrategy.h> ----------------

//#include <Square.h>
//#include <DrawStrategy.h>
//#include /* OpenGL graphics library */

class OpenGLSquareStrategy : public DrawStrategy<Square>
{
 public:
   explicit OpenGLSquareStrategy( /* Drawing related arguments */ )
   {}

   void draw( Square const& square /*, ...*/ ) const override
   {
      // ... Implementing the logic for drawing a square by means of OpenGL
   }

 private:
   /* Drawing related data members, e.g. colors, textures, ... */
};
```

### Comparison Between Visitor and Strategy  

1. ***Strategy design pattern*** identifies the implementation details of a single function as a **variation point**,  easy to add new types but not easy to add add new operations  
2.  ***Visitor design pattern*** identifies the general addition of operations as the **variation point**, easy to add new operations but not easy to add new types.

### Shortcomings of the Strategy Design Pattern  

reduce the dependencies on a particular implementation detail by introducing an abstraction for that detail but

1. not able to easily add operations.  
2. pay off to identify such variation points early  
3. additional runtime indirection  
4. the major disadvantage is that a single Strategy should deal with either a single operation or a small group of cohesive function, otherwise multiple Strategy base classes and multiple data members  

```C++
/---- <DrawCircleStrategy.h> ----------------
class Circle;
class DrawCircleStrategy
{
public:
    virtual ~DrawCircleStrategy() = default;
    virtual void draw( Circle const& circle, /*some arguments*/ ) const = 0;
};
//---- <SerializeCircleStrategy.h> ----------------
class Circle;
class SerializeCircleStrategy
{
public:
    virtual ~SerializeCircleStrategy() = default;
    virtual void serialize( Circle const& circle, /*somearguments*/ ) const = 0;
};
//---- <Circle.h> ----------------
#include <Shape.h>
#include <DrawCircleStrategy.h>
#include <SerializeCircleStrategy.h>
#include <memory>
#include <utility>
class Circle : public Shape
{
public:
    explicit Circle( double radius
                    , std::unique_ptr<DrawCircleStrategy> drawer
                    , std::unique_ptr<SerializeCircleStrategy> serializer
                    /* potentially more strategy-related arguments
                    */ )
            : radius_( radius )
            , drawer_( std::move(drawer) )
            , serializer_( std::move(serializer) )
            {
            /* Checking that the given radius is valid and that
            the given std::unique_ptrs are not nullptrs */
            }
    void draw( /*some arguments*/ ) const override
    {
    	drawer_->draw( *this, /*some arguments*/ );
    }
    void serialize( /*some arguments*/ ) const override
    {
    	serializer_->serialize( *this, /*some arguments*/ );
    }
    double radius() const { return radius_; }
private:
    double radius_;
    std::unique_ptr<DrawCircleStrategy> drawer_;
    std::unique_ptr<SerializeCircleStrategy> serializer_;
    // ... Potentially more strategy-related data members
};
```

For a situation where you need to extract the details of many operations, it might be better to consider other approaches (***the External Polymorphism design pattern*** or the ***Type Erasure design pattern***) rather than ***Strategy Design Pattern***

### Policy-Based Design  

***policy-based design*** can be considered the **static polymorphism** form of the ***Strategy design pattern***

***concepts*** on template argument just like **virtual function** in base classes

1.  advantage: the performance improvement due to fewer pointer indirections:   
2. You would have one instantiation of Circle for every drawing strategy  
3. lose the opportunity to hide implementation details in a source file  

```C++
//---- <Circle.h> ----------------
#include <Shape.h>
#include <DrawCircleStrategy.h>
#include <memory>
#include <utility>
template< typename DrawCircleStrategy >
class Circle : public Shape
{
public:
    explicit Circle( double radius, DrawCircleStrategy drawer )
 
    {
    /* Checking that the given radius is valid */
    }
    void draw( /*some arguments*/ ) const override
    {
        drawer_( *this, /*some arguments*/ );
    }
    double radius() const { return radius_; }
private:
    double radius_;
    DrawCircleStrategy drawer_; // Could possibly be omitted, if the given
// strategy is presumed to be stateless.
};
```

## Guideline 20: Favor Composition over Inheritance  
Many **design patterns** are enabled by **composition**, not by **inheritance**. Most of the time it is preferable to use composition instead  

**inheritance** is a very powerful feature if used properly. The problem is the “if used properly” part. Inheritance has proven to be hard to use properly and also overused even misused

The aspects of inheritance that are often misunderstood:

1. **Reusability**; Inheritance is not about reusing code in a **base class**; instead, it is about being reused by other code that uses the base class polymorphically. **Real reusability** is created by the **polymorphic use** of a type, not by **polymorphic types**  
2. **Decoupling**: While inheritance decouples software entities, it also creates coupling between pure virtual functions in base class and all derived classes, just as in ***classic Visitor***  

## Guideline 21: Use Command to Isolate What Things Are Done  
### The Command Design Pattern Explained  

***The Command design pattern*** focuses on the abstraction and isolation of work packages that (most often) are executed once and (usually) immediately.

> Intent: “Encapsulate a request as an object, thereby letting you parameterize clients with different requests, queue or log requests, and support undoable operations.”  

```C++
//---- <CalculatorCommand.h> ----------------------------------------------------------------------

class CalculatorCommand
{
 public:
   virtual ~CalculatorCommand() = default;

   virtual int execute( int i ) const = 0;
   virtual int undo( int i ) const = 0;
};


//---- <Add.h> ------------------------------------------------------------------------------------

//#include <CalculatorCommand.h>

class Add : public CalculatorCommand
{
 public:
   explicit Add( int operand ) : operand_(operand) {}

   int execute( int i ) const override
   {
      return i + operand_;
   }
   int undo( int i ) const override
   {
      return i - operand_;
   }

 private:
   int operand_{};
};


//---- <Subtract.h> -------------------------------------------------------------------------------

//#include <CalculatorCommand.h>

class Subtract : public CalculatorCommand
{
 public:
   explicit Subtract( int operand ) : operand_(operand) {}

   int execute( int i ) const override
   {
      return i - operand_;
   }
   int undo( int i ) const override
   {
      return i + operand_;
   }

 private:
   int operand_{};
};


//---- <Calculator.h> -----------------------------------------------------------------------------

//#include <CalculatorCommand.h>
#include <memory>
#include <stack>

class Calculator
{
 public:
   void compute( std::unique_ptr<CalculatorCommand> command );
   void undoLast();

   int result() const;
   void clear();

 private:
   using CommandStack = std::stack<std::unique_ptr<CalculatorCommand>>;

   int current_{};
   CommandStack stack_;
};


//---- <Calculator.cpp> ----------------

//#include <Calculator.h>
#include <utility>

void Calculator::compute( std::unique_ptr<CalculatorCommand> command )
{
   current_ = command->execute( current_ );
   stack_.push( std::move(command) );
}

void Calculator::undoLast()
{
   if( stack_.empty() ) return;

   auto command = std::move(stack_.top());
   stack_.pop();

   current_ = command->undo(current_);
}

int Calculator::result() const
{
   return current_;
}

void Calculator::clear()
{
   current_ = 0;
   CommandStack{}.swap( stack_ );  // Clearing the stack
}

```

### The Command Design Pattern Versus the Strategy Design Pattern  
From either a structural point of view or an implementation point of view, there is no difference between ***Strategy design pattern*** and ***Command design pattern***  

The difference lies entirely in the intent of the two design patterns

1. ***the Strategy design pattern*** specifies how something should be done
2. ***the Command design pattern*** specifies what should be done  

illustrated by `std::partition` and `std::for_each`

```C++
namespace std {
    // UnaryPredicate is Strategy
    template< typename ForwardIt, typename UnaryPredicate >
    constexpr ForwardIt
    	partition( ForwardIt first, ForwardIt last, UnaryPredicate p // 
    );
    // UnaryFunction is Command 
    template< typename InputIt, typename UnaryFunction >
    constexpr UnaryFunction
    	for_each( InputIt first, InputIt last, UnaryFunction f );
} // namespace std
```

### Shortcomings of the Command Design Pattern  
same as [Strategy design pattern](#Shortcomings of the Strategy Design Pattern)

## Guideline 22: Prefer Value Semantics over Reference Semantics  
### The Shortcomings of the GoF Style: Reference Semantics  
The design patterns collected by the Gang of Four and presented in their book in 1994  were introduced as object-oriented design patterns  

This pure OO style is referred to as the ***GoF style*** which is outdated way of doing things in C++, 

1. performance  
2. ***GoF style*** falls into ***reference semantics*** (or sometimes also ***pointer semantics***)  

### Reference Semantics Examples

**references**, and especially **pointers**, make our life so much harder and should avoid.  

```C++
void print( std::span<int> s )
{
   std::cout << " (";
   for( int i : s ) {
      std::cout << ' ' << i;
   }
   std::cout << " )\n";
}

int main()
{
   std::vector<int> v{ 1, 2, 3, 4 };

   // std::span<int const> const s{v}; // s represents a const
   std::span<int> const s{ v };

   s[2] = 99;  // Works!

   // Prints ( 1 2 99 4 );
   print( s );

   v = { 5, 6, 7, 8, 9 }; // (possibly) performed a reallocation and thus changed the address of its first elem
   s[2] = 99;  // s becomes wild pointers

   // Prints ?
   print( s );

   return EXIT_SUCCESS;
}


int main()
{
   std::vector<int> vec{ 1, -3, 27, 42, 4, -8, 22, 42, 37, 4, 18, 9 };
   auto const pos = std::max_element( begin(vec), end(vec) );

   // *pos becomes 4 after delete the first 42
   vec.erase( std::remove( begin(vec), end(vec), *pos ), end(vec) );

   print( vec ); // // ( 1 -3 27 4 -8 22 42 37 18 9 )

   return EXIT_SUCCESS;
}
```

### The Modern C++ Philosophy: Value Semantics  

***value semantics*** makes your code easier to understand and might even have **better performance** by fewer indirect access of ***reference semantics*** and much **better memory layout** and **access pattern**   

`std::string` prior to C++11 is implemented using **copy-on-write**, proven to be a pessimization in a multithreaded world.    

1. values are easier to reason about than pointers and references  
2. value change happens locally, <font color=red>an advantage that compilers heavily exploit for optimization efforts</font>
3. values don’t make us think about ownership    
4. easier to think about threading issues  

<font color=red>**expensive copy operations** can be alleviated by ***copy elision***, ***move semantics***, and well…pass-by-reference  </font>

### Value Semantics: Use `std::optional` deal with error

```C++
int to_int( std::string_view ); // throw an exception, not good for preferences
bool to_int( std::string_view s, int& );
std::unique_ptr<int> to_int( std::string_view );
// std::optional, equivalent to the pointer approach, but no cost of dynamic memory allocation, and lifetime management
std::optional<int> to_int( std::string_view );
```



## Guideline 23: Prefer a Value-Based Implementation of Strategy and Command  
### Prefer to Use Value Semantics to Implement Design Patterns  

### std::function is based on value semantics  

```C++
void foo( int i )
{
   std::cout << "foo: " << i << '\n';
}

int main()
{
   // Create a default std::function instance. Calling it results
   // in a std::bad_function_call exception
   std::function<void(int)> f{};

   f = []( int i ){  // Assigning a callable to 'f'
      std::cout << "lambda: " << i << '\n';
   };

   f(1);  // Calling 'f' with the integer '1'

   auto g = f;  // Copying 'f' into 'g'

   f = foo;  // Assigning a different callable to 'f'

   f(2);  // Calling 'f' with the integer '2'
   g(3);  // Calling 'g' with the integer '3'

   return EXIT_SUCCESS;
}
```

### Refactoring using value semantics

```C++
//---- <Shape.h> ----------------------------------------------------------------------------------

class Shape
{
 public:
   virtual ~Shape() = default;
   virtual void draw( /*some arguments*/ ) const = 0;
};


//---- <Circle.h> ---------------------------------------------------------------------------------

//#include <Shape.h>
#include <functional>
#include <utility>

class Circle : public Shape
{
 public:
   using DrawStrategy = std::function<void(Circle const& /*, ...*/)>;

   explicit Circle( double radius, DrawStrategy drawer )
      : radius_( radius )
      , drawer_( std::move(drawer) )
   {
      /* Checking that the given radius is valid and that
         the given 'std::function' instance is not empty */
   }

   void draw( /*some arguments*/ ) const override
   {
      drawer_( *this /*, some arguments*/ );
   }

   double radius() const { return radius_; }

 private:
   double radius_;
   DrawStrategy drawer_;
};


//---- <Square.h> ---------------------------------------------------------------------------------

//#include <Shape.h>
#include <functional>
#include <utility>

class Square : public Shape
{
 public:
   using DrawStrategy = std::function<void(Square const& /*, ...*/)>;

   explicit Square( double side, DrawStrategy drawer )
      : side_( side )
      , drawer_( std::move(drawer) )
   {
      /* Checking that the given side length is valid and that
         the given 'std::function' instance is not empty */
   }

   void draw( /*some arguments*/ ) const override
   {
      drawer_( *this /*, some arguments*/ );
   }

   double side() const { return side_; }

 private:
   double side_;
   DrawStrategy drawer_;
};


//---- <OpenGLCircleStrategy.h> -------------------------------------------------------------------

//#include <Circle.h>
//#include /* OpenGL graphics library */

class OpenGLCircleStrategy
{
 public:
   explicit OpenGLCircleStrategy( /* Drawing related arguments */ )
   {}

   void operator()( Circle const& circle /*, ...*/ ) const
   {
      // ... Implementing the logic for drawing a circle by means of OpenGL
   }

 private:
   /* Drawing related data members, e.g. colors, textures, ... */
};


//---- <OpenGLSquareStrategy.h> -------------------------------------------------------------------

//#include <Square.h>
//#include /* OpenGL graphics library */

class OpenGLSquareStrategy
{
 public:
   explicit OpenGLSquareStrategy( /* Drawing related arguments */ )
   {}

   void operator()( Square const& square /*, ...*/ ) const
   {
      // ... Implementing the logic for drawing a square by means of OpenGL
   }

 private:
   /* Drawing related data members, e.g. colors, textures, ... */
};
```

### Shortcomings of the std::function Solution  
1. potential performance disadvantage of standard library implementation  `std::function`
2. `std::function` can replace only a single virtual function  

# Chapter 6. The Adapter, Observer, and CRTP Design Patterns  
## Guideline 24: Use Adapters to Standardize Interfaces  
***The Adapter design pattern*** focus on standardizing interfaces and helping nonintrusively add functionality into an **existing inheritance hierarchy**  

> Intent: “Convert the interface of a class into another interface clients expect. Adapter
> lets classes work together that couldn’t otherwise because of incompatible interfaces.”  

### Object Adapters Versus Class Adapters 

**object adapter** build on **composition**, store an instance of the wrapped type or a pointer to the base class of hierarchy wrapped type

**class adapter** inherits from wrapped type, is preferred only when:

1. have to override a **virtual function**.
2. need access to a ***protected*** member function 
3. require the adapted type to be constructed before another base class 
4. need to share a **common virtual base class** or override the construction of a virtual base class 
5. need draw significant advantage from the ***Empty Base Optimization (EBO)*** 

### Example of Stack using inheritance and template

adapt the interface of a container to the stack operations 

```C++
//---- <Stack.h> ----------------
template<typename T> 
class Stack
{
public:
    virtual ~Stack() = default;
    virtual T& top() = 0;
    virtual bool empty() const = 0;
    virtual size_t size() const = 0;
    virtual void push( T const& value ) = 0;
    virtual void pop() = 0;
};

//---- <VectorStack.h> ----------------
#include <Stack.h>
template< typename T >
class VectorStack : public Stack<T>
{
public:
    T& top() override { return vec_.back(); }
    bool empty() const override { return vec_.empty(); }
    size_t size() const override { return vec_.size(); }
    void push( T const& value ) override { vec_.push_back(value);
    }
    void pop() override { vec_.pop_back(); }
private:
    std::vector<T> vec_;
};
```

realization of C++ Standard Library 

```C++
template< typename T
		, typename Container = std::deque<T> >
class stack;
template< typename T
		, typename Container = std::deque<T> >
class queue;
template< typename T
		, typename Container = std::vector<T>
		, typename Compare = std::less<typename Container::value_type> >
class priority_queue;
```

### Comparison Between Adapter and Strategy 

From a structural point of view, the ***Strategy and Adapter design patterns*** are very similar, but the intent is different 

1. the primary focus of an ***Adapter*** is to standardize interfaces and integrate incompatible functionality into an existing set of conventions 
2. the primary focus of the ***Strategy design pattern*** is to enable the configuration of behavior from
   the outside, building on and providing an expected interface 

### Function Adapters 

<font color=red>Remember that all kinds of **abstractions** represent a set of requirements and thus have to adhere to the ***Liskov Substitution Principle (LSP)***. This is also true for **overload set**</font>

The purpose of the **free functions** `begin()` and `end()` is to adapt the **iterator interface** of any type to the expected **STL iterator interface**. This kind of **function adapter** was called a ***shim*** 

One valuable property of ***shim*** technique is that it is possible to add a **free function** to any type, even to types that you could never adapt 

### Shortcomings of the Adapter Design Pattern 
***Adapter design pattern*** makes it very easy to combine things that do not belong together. Thus, it’s very important that you consider the expected behavior and check for ***LSP violations*** when applying ***Adapter design pattern***

## Guideline 25: Apply Observers as an Abstract Notification Mechanism 
### The Observer Design Pattern Explained 

The intent of the ***Observer design pattern*** is decoupling between the subject and its potentially many observers

> Intent: “Define a one-to-many dependency between objects so that when one object changes state, all its dependents are notified and updated automatically.” 

```C++
//---- <Person.h> ---------------------------------------------------------------------------------

//#include <Observer.h>
#include <string>
#include <set>

class Person
{
 public:
   enum StateChange
   {
      forenameChanged,
      surnameChanged,
      addressChanged
   };

   using PersonObserver = Observer<Person,StateChange>;

   explicit Person( std::string forename, std::string surname )
      : forename_{ std::move(forename) }
      , surname_{ std::move(surname) }
   {}

   bool attach( PersonObserver* observer );
   bool detach( PersonObserver* observer );

   void notify( StateChange property );

   void forename( std::string newForename );
   void surname ( std::string newSurname );
   void address ( std::string newAddress );

   std::string const& forename() const { return forename_; }
   std::string const& surname () const { return surname_; }
   std::string const& address () const { return address_; }

 private:
   std::string forename_;
   std::string surname_;
   std::string address_;

   std::set<PersonObserver*> observers_;
};


//---- <Person.cpp> -------------------------------------------------------------------------------

//#include <Person.h>

void Person::forename( std::string newForename )
{
   forename_ = std::move(newForename);
   notify( forenameChanged );
}

void Person::surname( std::string newSurname )
{
   surname_ = std::move(newSurname);
   notify( surnameChanged );
}

void Person::address( std::string newAddress )
{
   address_ = std::move(newAddress);
   notify( addressChanged );
}

bool Person::attach( PersonObserver* observer )
{
   auto [pos,success] = observers_.insert( observer );
   return success;
}

bool Person::detach( PersonObserver* observer )
{
   return ( observers_.erase( observer ) > 0U );
}

void Person::notify( StateChange property )
{
   // This formulation makes sure detach() operations
   // can be detected during the iteration
   // but not cope with attach() and concuurency
   for( auto iter=begin(observers_); iter!=end(observers_); )
   {
      auto const pos = iter++;
      (*pos)->update(*this,property);
   }
}
```



### A Classic Observer Implementation 

***push oberver*** is given all necessary information by the subject and therefore is not required to pull any information from the subject on its own. This can reduce the coupling to the subject significantly but violate of the ***ISP*** to artificially couple all possible state changes.  

```C++
class Observer
{
public:
// ...
    virtual void update1( /*arguments representing the updated
                        state*/ ) = 0;
    virtual void update2( /*arguments representing the updated
                        state*/ ) = 0;
// ...
};
```

***pull observer*** is passed a reference to the ***subject*** and required to pull the new information from the ***subject*** on their own. This reduces dependency on the number and kinds of arguments at cost of a strong, direct dependency between the classes deriving from ***Observer*** and the ***subject***

```C++
//---- <Observer.h> -------------------------------------------------------------------------------

template< typename Subject, typename StateTag >
class Observer
{
 public:
   virtual ~Observer() = default;

   virtual void update( Subject const& subject, StateTag property ) = 0;
};

```

### An Observer Implementation Based on Value Semantics 

Although technically possible, it is not particularly common to pass around pointers to `std::function`. Therefore, the Observer class, in the form of an ***Adapter*** for `std::function`, adds nothing but just used to refer to observer rather than a copy.

```C++
//---- <Observer.h> -------------------------------------------------------------------------------

#include <functional>

template< typename Subject, typename StateTag >
class Observer
{
 public:
   using OnUpdate = std::function<void(Subject const&,StateTag)>;

   // No virtual destructor necessary

   explicit Observer( OnUpdate onUpdate )
      : onUpdate_{ std::move(onUpdate) }
   {
      // Possibly respond on an invalid/empty std::function instance
   }

   // Non-virtual update function
   void update( Subject const& subject, StateTag property )
   {
      onUpdate_( subject, property );
   }

 private:
   OnUpdate onUpdate_;
};
```

### Shortcomings of the Observer Design Pattern 

1. `std::function` approach works only for a ***pull observer*** with a single `update()` function. Use ***Type Erasure design pattern*** if need ***push observer*** with multiple `update()` functions 
2. there is no pure value-based implementation.  
3. A far bigger disadvantage is the **thread-safe registration and deregistration of observers and handling of event in a multithreaded environment**

overuse of observers can quickly and easily lead to a complex network of interconnections 

## Guideline 26: Use CRTP to Introduce Static Type Categories 

### The CRTP Design Pattern Explained  

***CRTP (Curiously Recurring Template Pattern)*** builds on the common idea of creating an
abstraction using a base class by a compile-time relationship rather than virtual functions

> Intent: “Define a compile-time abstraction for a family of related types.”  

#### details

<font color=red>It is sufficient to use an incomplete type to instantiate a template</font>

```C++
template< typename T >
class DynamicVector
   : public DenseVector< DynamicVector<T> >
{
   ...
}
```

>  A base class destructor should be either public and virtual, or protected and non-virtual.  

<font color=red>To prevent **destructor** to be virtual, declare it to be **protected** and **non-virtual**</font>

```C++
template< typename Derived >
struct DenseVector
{
protected:
	~DenseVector() = default;
}
```

<font color=red>can’t use anything of  `Derived` as long as the derived class is an incomplete type, use `decltype(auto)` </font>

```C++
template< typename Derived >
struct DenseVector
{
    // compiler error
    // no type named 'value_type' in 'DynamicVector<int>'
    using value_type = typename Derived::value_type;
    using iterator = typename Derived::iterator;
    using const_iterator = typename Derived::const_iterator;
	
    value_type& operator[]( size_t index ) { return derived()[index]; }
    value_type const& operator[]( size_t index ) const { return derived()[index]; }
    iterator begin() { return derived().begin(); }
    const_iterator begin() const { return derived().begin(); }
    iterator end() { return derived().end(); }
    const_iterator end() const { return derived().end(); 
};
```

#### full code

```C++
//---- <DenseVector.h> ----------------------------------------------------------------------------

#include <cmath>
#include <numeric>
#include <ostream>

template< typename Derived >
struct DenseVector
{
   constexpr Derived&       derived()       noexcept { return static_cast<Derived&>(*this); }
   constexpr Derived const& derived() const noexcept { return static_cast<Derived const&>(*this); }

   constexpr size_t size() const noexcept { return derived().size(); }

   decltype(auto) operator[]( size_t index )       noexcept { return derived()[index]; }
   decltype(auto) operator[]( size_t index ) const noexcept { return derived()[index]; }

   decltype(auto) begin()       noexcept { return derived().begin(); }
   decltype(auto) begin() const noexcept { return derived().begin(); }
   decltype(auto) end()         noexcept { return derived().end(); }
   decltype(auto) end()   const noexcept { return derived().end(); }
};

template< typename Derived >
std::ostream& operator<<( std::ostream& os, DenseVector<Derived> const& vector )
{
   size_t const size( vector.size() );

   os << "(";
   for( size_t i=0UL; i<size; ++i ) {
      os << " " << vector[i];
   }
   os << " )";

   return os;
}

template< typename Derived >
decltype(auto) l2norm( DenseVector<Derived> const& vector )
{
   using T = typename Derived::value_type;
   return std::sqrt( std::inner_product( std::begin(vector), std::end(vector)
                                       , std::begin(vector), T{} ) );
}


//---- <DynamicVector.h> --------------------------------------------------------------------------

//#include <DenseVector.h>
#include <vector>
#include <initializer_list>

template< typename T >
class DynamicVector
   : public DenseVector< DynamicVector<T> >
{
 public:
   using value_type     = T;
   using iterator       = typename std::vector<T>::iterator;
   using const_iterator = typename std::vector<T>::const_iterator;

   DynamicVector() = default;
   DynamicVector( std::initializer_list<T> init )
      : values_( std::begin(init), std::end(init) )
   {}

   size_t size() const noexcept { return values_.size(); }

   T&       operator[]( size_t index )       noexcept { return values_[index]; }
   T const& operator[]( size_t index ) const noexcept { return values_[index]; }

   iterator       begin()       noexcept { return values_.begin(); }
   const_iterator begin() const noexcept { return values_.begin(); }
   iterator       end()         noexcept { return values_.end(); }
   const_iterator end()   const noexcept { return values_.end(); }

   // ... Many numeric functions

 private:
   std::vector<T> values_;
};


//---- <StaticVector.h> ---------------------------------------------------------------------------

//#include <DenseVector.h>
#include <array>
#include <initializer_list>

template< typename T, size_t Size >
class StaticVector
   : public DenseVector< StaticVector<T,Size> >
{
 public:
   using value_type     = T;
   using iterator       = typename std::array<T,Size>::iterator;
   using const_iterator = typename std::array<T,Size>::const_iterator;

   StaticVector() = default;
   StaticVector( std::initializer_list<T> init )
   {
      std::copy( std::begin(init), std::end(init), std::begin(values_) );
   }

   size_t size() const noexcept { return values_.size(); }

   T&       operator[]( size_t index )       noexcept { return values_[index]; }
   T const& operator[]( size_t index ) const noexcept { return values_[index]; }

   iterator       begin()       noexcept { return values_.begin(); }
   const_iterator begin() const noexcept { return values_.begin(); }
   iterator       end()         noexcept { return values_.end(); }
   const_iterator end()   const noexcept { return values_.end(); }

   // ... Many numeric functions

 private:
   std::array<T,Size> values_;
};
```

### Shortcomings of the CRTP Design Pattern  

At first sight, CRTP most definitely looks like the ultimate solution for all kinds of inheritance hierarchies, however:

1.  lack of a **common base class**, cannot use whenever a common base class is required, for instance, to store different types in a collection  
2. everything work with a CRTP base class becomes a template
    ```C++
    template< typename Derived >
    auto l2norm( DenseVector<Derived> const& vector );
    ```
3. CRTP is an intrusive design pattern  

### The Future of CRTP: A Comparison Between CRTP and C++20 Concepts  

**C++20 concepts** are pretty similar to CRTP but represent an easier, nonintrusive alternative, but still with the drawbacks, such as the lack of a **common base class**, the quick spreading of template code, and the restriction to compile-time polymorphism.  

```C++
//---- <DenseVector.h> ----------------------------------------------------------------------------

#include <cmath>
#include <concepts>
#include <numeric>
#include <ostream>

struct DenseVectorTag {};

template< typename T >
struct IsDenseVector
   : public std::is_base_of<DenseVectorTag,T>
{};

template< typename T >
constexpr bool IsDenseVector_v = IsDenseVector<T>::value;

template< typename T >
concept DenseVector =
   requires ( T t, size_t index ) {
      t.size();
      t[index];
      { t.begin() } -> std::same_as<typename T::iterator>;
      { t.end() } -> std::same_as<typename T::iterator>;
   } &&
   requires ( T const t, size_t index ) {
      t[index];
      { t.begin() } -> std::same_as<typename T::const_iterator>;
      { t.end() } -> std::same_as<typename T::const_iterator>;
   } &&
   IsDenseVector_v<T>;

template< DenseVector VectorT >
std::ostream& operator<<( std::ostream& os, VectorT const& vector )
{
   size_t const size( vector.size() );

   os << "(";
   for( size_t i=0UL; i<size; ++i ) {
      os << " " << vector[i];
   }
   os << " )";

   return os;
}

template< DenseVector VectorT >
decltype(auto) l2norm( VectorT const& vector )
{
   using T = typename VectorT::value_type;
   return std::sqrt( std::inner_product( std::begin(vector), std::end(vector)
                                       , std::begin(vector), T{} ) );
}


//---- <DynamicVector.h> --------------------------------------------------------------------------

//#include <DenseVector.h>
#include <vector>
#include <initializer_list>

template< typename T >
class DynamicVector : private DenseVectorTag
{
 public:
   using value_type     = T;
   using iterator       = typename std::vector<T>::iterator;
   using const_iterator = typename std::vector<T>::const_iterator;

   DynamicVector() = default;
   DynamicVector( std::initializer_list<T> init )
      : values_( std::begin(init), std::end(init) )
   {}

   size_t size() const noexcept { return values_.size(); }

   T&       operator[]( size_t index )       noexcept { return values_[index]; }
   T const& operator[]( size_t index ) const noexcept { return values_[index]; }

   iterator       begin()       noexcept { return values_.begin(); }
   const_iterator begin() const noexcept { return values_.begin(); }
   iterator       end()         noexcept { return values_.end(); }
   const_iterator end()   const noexcept { return values_.end(); }

   // ... Many numeric functions

 private:
   std::vector<T> values_;
};


//---- <StaticVector.h> ---------------------------------------------------------------------------

//#include <DenseVector.h>
#include <array>
#include <initializer_list>

template< typename T, size_t Size >
class StaticVector
{
 public:
   using value_type     = T;
   using iterator       = typename std::array<T,Size>::iterator;
   using const_iterator = typename std::array<T,Size>::const_iterator;

   StaticVector() = default;
   StaticVector( std::initializer_list<T> init )
   {
      std::copy( std::begin(init), std::end(init), std::begin(values_) );
   }

   size_t size() const noexcept { return values_.size(); }

   T&       operator[]( size_t index )       noexcept { return values_[index]; }
   T const& operator[]( size_t index ) const noexcept { return values_[index]; }

   iterator       begin()       noexcept { return values_.begin(); }
   const_iterator begin() const noexcept { return values_.begin(); }
   iterator       end()         noexcept { return values_.end(); }
   const_iterator end()   const noexcept { return values_.end(); }

   // ... Many numeric functions

 private:
   std::array<T,Size> values_;
};

template< typename T, size_t Size >
struct IsDenseVector< StaticVector<T,Size> >
   : public std::true_type
{};
```

## Guideline 27: Use CRTP for Static Mixin Classes  
***CRTP design pattern*** is obsolete by the advent of **C++20 concepts**. 

But CRTP is still unchallenged, as an **implementation pattern** to provide **static mixin functionality** via **mixin classes**   
In this case the CRTP base class is preferred to be a **private base class**, not a **public one**, and doesn’t provide a virtual or protected destructor  

```C++
//---- <Addable.h> --------------------------------------------------------------------------------

template< typename Derived >
struct Addable
{
   friend Derived& operator+=( Derived& lhs, Derived const& rhs ) {
      lhs.get() += rhs.get();
      return lhs;
   }

   friend Derived operator+( Derived const& lhs, Derived const& rhs ) {
      return Derived{ lhs.get() + rhs.get() };
   }
};


//---- <Subtractable.h> ---------------------------------------------------------------------------

template< typename Derived >
struct Subtractable
{
   friend Derived& operator-=( Derived& lhs, Derived const& rhs ) {
      lhs.get() -= rhs.get();
      return lhs;
   }

   friend Derived operator-( Derived const& lhs, Derived const& rhs ) {
      return Derived{ lhs.get() - rhs.get() };
   }
};


//---- <IntegralArithmetic.h> ---------------------------------------------------------------------

//#include <Addable.h>
//#include <Subtractable.h>

template< typename Derived >
struct IntegralArithmetic
   : public Addable<Derived>
   , public Subtractable<Derived>
{};


//---- <Printable.h> ------------------------------------------------------------------------------

#include <iosfwd>

template< typename Derived >
struct Printable
{
   friend std::ostream& operator<<( std::ostream& os, Derived const& d )
   {
      os << d.get();
      return os;
   }
};


//---- <Swappable.h> ------------------------------------------------------------------------------

#include <utility>

template< typename Derived >
struct Swappable
{
   friend void swap( Derived& lhs, Derived& rhs )
   {
      using std::swap;  // Enable ADL
      swap( lhs.get(), rhs.get() );
   }
};


//---- <StrongType.h> -----------------------------------------------------------------------------

#include <utility>

template< typename T, typename Tag, template<typename> class... Skills >
struct StrongType
   : public Skills< StrongType<T,Tag,Skills...> >...
{
 public:
   using value_type = T;

   explicit constexpr StrongType( T const& value ) : value_( value ) {}

   constexpr T&       get()       noexcept { return value_; }
   constexpr T const& get() const noexcept { return value_; }

   void swap( StrongType& other ) {
      using std::swap;
      swap( value_, other.value_ );
   }

 private:
   T value_;
};

template< typename T, typename Tag, template<typename> class... Skills >
std::ostream& operator<<( std::ostream& os, StrongType<T,Tag,Skills...> const& nt )
{
   os << nt.get();
   return os;
}

template< typename T, typename Tag, template<typename> class... Skills >
void swap( StrongType<T,Tag,Skills...>& a, StrongType<T,Tag,Skills...>& b )
{
   a.swap( b );
}


//---- <Distances.h> ------------------------------------------------------------------------------

//#include <IntegralArithmetic.h>
//#include <Printable.h>
//#include <StrongType.h>
//#include <Swappable.h>

template< typename T >
using Meter = StrongType<T,struct MeterTag,IntegralArithmetic,Printable,Swappable>;

template< typename T >
using Kilometer = StrongType<T,struct KilometerTag,IntegralArithmetic,Printable,Swappable>;


//---- <Person.h> ---------------------------------------------------------------------------------

//#include <StrongType.h>
#include <string>

using Surname = StrongType<std::string,struct SurnameTag,Printable,Swappable>;


//---- <Main.cpp> ---------------------------------------------------------------------------------

//#include <Distances.h>
#include <cstdlib>
#include <iostream>

int main()
{
   auto const m1 = Meter<long>{ 100 };
   auto const m2 = Meter<long>{  50 };

   auto const m3 = m1 + m2;  // Compiles and results in 150 meters

   std::cout << "\n m3  = " << m3 << "m\n\n";

   return EXIT_SUCCESS;
}
```

# Chapter 7. The Bridge, Prototype, and External Polymorphism Design Patterns  
## Guideline 28: Build Bridges to Remove Physical Dependencies  

### The Bridge Design Pattern Explained 

The purpose of a ***Bridge*** is to minimize physical dependencies by **encapsulating some implementation details behind an abstraction**. In C++, it acts as a **compilation firewall**, which enables easy change 

> Intent: “Decouple an abstraction from its implementation so that the two can vary independently.” 

The **opaque pointer** ***pimpl*** (**pointer-to-implementation**) represents the Bridge to the encapsulated implementation details  

```C++
//---- <Engine.h> ---------------------------------------------------------------------------------

class Engine
{
 public:
   virtual ~Engine() = default;
   virtual void start() = 0;
   virtual void stop() = 0;
   // ... more engine-specific functions

 private:
   // ...
};


//---- <Car.h> ------------------------------------------------------------------------------------

//#include <Engine.h>
#include <memory>
#include <utility>

class Car
{
 protected:
   explicit Car( std::unique_ptr<Engine> engine )
      : pimpl_{ std::move(engine) }
   {}

 public:
   virtual ~Car() = default;
   virtual void drive() = 0;
   // ... more car-specific functions

 protected:
   // Avoid protected data, provide protected getter of private data intead
   Engine*       getEngine()       { return pimpl_.get(); }
   Engine const* getEngine() const { return pimpl_.get(); }

 private:
   std::unique_ptr<Engine> pimpl_;  // Pointer-to-implementation (pimpl)

   // ... more car-specific data members (wheels, drivetrain, ...)
};


//---- <ElectricEngine.h> -------------------------------------------------------------------------

//#include <Engine.h>

class ElectricEngine : public Engine
{
 public:
   void start() override;
   void stop() override;

 private:
   // ...
};


//---- <ElectricEngine.cpp> -----------------------------------------------------------------------

//#include <ElectricEngine.h>
#include <iostream>

void ElectricEngine::start()
{
   std::cout << "Starting the 'ElectricEngine'...\n";
}

void ElectricEngine::stop()
{
   std::cout << "Stopping the 'ElectricEngine'...\n";
}


//---- <ElectricCar.h> ----------------------------------------------------------------------------

//#include <Car.h>

class ElectricCar : public Car
{
 public:
   ElectricCar();

   void drive();
   // ...

 private:
   // ...
};


//---- <ElectricCar.cpp> --------------------------------------------------------------------------

//#include <ElectricCar.h>
//#include <ElectricEngine.h>
#include <iostream>

ElectricCar::ElectricCar()
   : Car{ std::make_unique<ElectricEngine>( /*engine arguments*/ ) }
   // ... Initialization of the other data members
{}

void ElectricCar::drive()
{
   getEngine()->start();
   std::cout << "Driving the 'ElectricCar'...\n";
   getEngine()->stop();
}

// ... Other 'ElectricCar' member functions, primarily using the 'Engine'
//     abstraction, but potentially also explicitly dealing with an
//     'ElectricEngine'.
```

### The Pimpl Idiom 

The following implementation of `Person` uses the ***Bridge design pattern*** in its simplest, local, nonpolymorphic form called ***the Pimpl idiom***, which is commonly and successfully used in both C and C++ for decades.

```C++
//---- <Person.h> ---------------------------------------------------------------------------------

#include <memory>

class Person
{
 public:
   // ...
   Person();
   ~Person();

   Person( Person const& other );
   Person& operator=( Person const& other );

   Person( Person&& other );
   Person& operator=( Person&& other );

   int year_of_birth() const;
   // ... Many more access functions

 private:
   struct Impl;
   std::unique_ptr<Impl> const pimpl_;
};


//---- <Person.cpp> -------------------------------------------------------------------------------

//#include <Person.h>
#include <string>

struct Person::Impl
{
   std::string forename;
   std::string surname;
   std::string address;
   std::string city;
   std::string country;
   std::string zip;
   int year_of_birth;
   // ... Potentially many more data members
};


Person::Person()
   : pimpl_{ std::make_unique<Impl>() }
{}
// avoid compiler generate the destructor in header, which in turn would require the definition of the destructor of the Impl class.
Person::~Person() = default;

// Since std::unique_ptr cannot be copied, have to implement the copy constructor to preserve the copy semantics 
Person::Person( Person const& other )
   : pimpl_{ std::make_unique<Impl>(*other.pimpl_) }
{}

Person& Person::operator=( Person const& other )
{
   *pimpl_ = *other.pimpl_;
   return *this;
}

Person::Person( Person&& other )
   : pimpl_{ std::make_unique<Impl>(std::move(*other.pimpl_)) }
{}

Person& Person::operator=( Person&& other )
{
   *pimpl_ = std::move(*other.pimpl_);
   return *this;
}

int Person::year_of_birth() const
{
   return pimpl_->year_of_birth;
}

// ... Many more Person member functions
```

### Comparison Between Bridge and Strategy 

For ***Strategy design pattern***, you can configure **implementation detail** by passing from the outside (for instance, via a **constructor** or via a **setter function**). ***Strategy*** falls into the category of a **behavioral design pattern** because it primary focus on the flexible configuration of behavior, i.e., the **reduction of logical dependencies**.

```C++
class DatabaseEngine
{
public:
    virtual ~DatabaseEngine() = default;
    // ... Many database-specific functions
};
class Database
{
public:
    explicit Database( std::unique_ptr<DatabaseEngine> engine );
    // ... Many database-specific functions
private:
    std::unique_ptr<DatabaseEngine> engine_;
};
// The database is unaware of any implementation details and requests them
// via its constructor from outside -> Strategy design pattern
Database::Database( std::unique_ptr<DatabaseEngine> engine )
	: engine_{ std::move(engine) }
{}
```

For ***Bridge design pattern***, a class knows about the implementation details but primarily wants to reduce the physical dependencies on these details. ***Bridge*** falls into the category of **structural design patterns** because it primarily focuses on the **physical dependencies of the implementation details**, not the **logical dependencies**.

```C++
class Database
{
public:
    explicit Database();
    // ...
private:
	std::unique_ptr<DatabaseEngine> pimpl_;
};
// The database knows about the required implementation details,but does
// not want to depend too strongly on it -> Bridge design pattern
Database::Database()
	: pimpl_{ std::make_unique<ConcreteDatabaseEngine>( /*some
arguments*/ ) }
{}
```

### Shortcomings of the Bridge Design Pattern 

some performance overhead involved

1. ***Bridge*** introduces an additional indirection: the ***pimpl*** pointer making all access to the implementation details more expensive. 
2. pay for the virtual function call overhead 
3. pay more due to the lack of **inlining** of even the simplest function accessing data members 
4. pay for an additional dynamic memory allocation whenever you create a new instance of a class implemented  
5. memory overhead caused by introducing the ***pimpl*** pointer.  
6. that the code complexity of internals of a class has increased 

## Guideline 29: Be Aware of Bridge Performance Gains and Losses 
***Bridge Design Pattern*** may introduce a performance penalty, influenced by many factors: access through an indirection, virtual function calls, inlining, dynamic memory allocations, etc.  

**partial Bridge** can have a positive impact on performance when **frequently used data** is stored directly in class and **infrequently used data** are stored inside the ***Impl class*** because the size of class type is consistent with size of cache lines which is unit of **cache-based architectures memory load**. 

## Guideline 30: Apply Prototype for Abstract Copy Operations 

### The Prototype Design Pattern Explained  

The ***Prototype design pattern*** is focused on providing an abstract way of creating copies of some abstract entity.  

> Intent: “Specify the kind of objects to create using a prototypical instance, and create new objects by copying this prototype.”  

```C++
//---- <Animal.h> ---------------------------------------------------------------------------------

#include <memory>

class Animal
{
 public:
   virtual ~Animal() = default;
   virtual void makeSound() const = 0;
   virtual std::unique_ptr<Animal> clone() const = 0; // Prototype design pattern
};


//---- <Sheep.h> ----------------------------------------------------------------------------------

//#include <Animal.h>
#include <string>

class Sheep : public Animal
{
 public:
   explicit Sheep( std::string name ) : name_{ std::move(name) } {}

   void makeSound() const override;
   std::unique_ptr<Animal> clone() const override;  // Prototype design pattern

 private:
   std::string name_;
};


//---- <Sheep.cpp> --------------------------------------------------------------------------------

//#include <Sheep.h>
#include <iostream>

void Sheep::makeSound() const
{
   std::cout << "baa\n";
}

std::unique_ptr<Animal> Sheep::clone() const
{
   return std::make_unique<Sheep>(*this);  // Copy-construct a sheep
}

```

### Shortcomings of the Prototype Design Pattern  
There is no **value semantics** solution for the ***Prototype design pattern***

disadvantage is the performance drawbacks resulting from pointer indirections and memory allocations.  

## Guideline 31: Use External Polymorphism for Nonintrusive Runtime Polymorphism  
### The External Polymorphism Design Pattern Explained  

Its intent is to enable the **polymorphic treatment of nonpolymorphic types** (types without a single virtual function).  

> Intent: “Allow C++ classes unrelated by inheritance and/or having no virtual methods to be treated polymorphically. These unrelated classes can be treated in a common manner
> by software that uses them.”  

```C++
//---- <Circle.h> ---------------------------------------------------------------------------------

class Circle
{
 public:
   explicit Circle( double radius )
      : radius_( radius )
   {
      /* Checking that the given radius is valid */
   }

   double radius() const { return radius_; }
   /* Several more getters and circle-specific utility functions */

 private:
   double radius_;
   /* Several more data members */
};


//---- <Square.h> ---------------------------------------------------------------------------------

class Square
{
 public:
   explicit Square( double side )
      : side_( side )
   {
      /* Checking that the given side length is valid */
   }

   double side() const { return side_; }
   /* Several more getters and square-specific utility functions */

 private:
   double side_;
   /* Several more data members */
};


//---- <Shape.h> ----------------------------------------------------------------------------------

#include <functional>
#include <stdexcept>
#include <utility>

class ShapeConcept
{
 public:
   virtual ~ShapeConcept() = default;

   virtual void draw() const = 0;

   // ... Potentially more polymorphic operations
};


template< typename ShapeT
        , typename DrawStrategy >
class ShapeModel : public ShapeConcept
{
 public:
   explicit ShapeModel( ShapeT shape, DrawStrategy drawer )
      : shape_{ std::move(shape) }
      , drawer_{ std::move(drawer) }
   {}

   void draw() const override { drawer_(shape_); }

 private:
   ShapeT shape_;
   DrawStrategy drawer_;
};


//---- <OpenGLDrawStrategy.h> ---------------------------------------------------------------------

//#include <Circle.h>
//#include <Square.h>
//#include /* OpenGL graphics library headers */

class OpenGLDrawStrategy
{
 public:
   explicit OpenGLDrawStrategy( /* Drawing related arguments */ )
   {}

   void operator()( Circle const& circle ) const
   {
      // ... Implementing the logic for drawing a circle by means of OpenGL
   }
   void operator()( Square const& square ) const
   {
      // ... Implementing the logic for drawing a square by means of OpenGL
   }

 private:
   /* Drawing related data members, e.g., colors, textures, ... */
};

```

### Comparison Between External Polymorphism and Adapter  

1. ***Adapter design pattern*** is focused on **standardizing interfaces** and adapts a type or function to an **existing interface** 
2. ***External Polymorphism design pattern*** creates a new, external hierarchy to abstract from a set of related, nonpolymorphic type  

### Shortcomings of the External Polymorphism Design Pattern    

benefits:

1. gain performance due to fewer runtime indirections  
2. benefits for the reduction of dependencies  
3. types become simpler and nonpolymorphic.  
4. nonintrusively, equip any type with polymorphic behavior  
5. makes it very easy to wrap types that do not fulfill the **semantic expectations**  

only one major disadvantage is: the ***External Polymorphism design pattern*** does not really fulfill the expectations of a clean and simple solution

1. It does not help to reduce pointers, does not reduce the number of manual allocations, does not lower the number of inheritance hierarchies, and does not help to simplify user code  
2. It does not save you from thinking about a proper abstraction.  

# Chapter 8. The Type Erasure Design Pattern  

## Guideline 32: Consider Replacing Inheritance Hierarchies with Type Erasure 
### The History of Type Erasure 

The philosophy of `std::unique_ptr` is to represent nothing but the simplest possible wrapper around a **raw pointer**: it should be as fast as a raw pointer, and have the same size as a **raw pointer**. So `std::unique_ptr` is designed such that for stateless deleters, any size overhead can be avoided 

```C++
template< class T, class Deleter = std::default_delete<T>> class unique_ptr;
```

Since `std::shared_ptr` has to store many more data items in its so-called **control block** (that includes the reference count, the weak count, etc.), it uses **Type Erasure** to literally erase the type of the deleter, removing any kind of possible dependency 

### The Type Erasure Design Pattern Explained 

Type Erasure is nothing but a compound design pattern, combination of three other design patterns: ***External Polymorphism***, ***Bridge*** and (optionally) ***Prototype*** 

> Intent: “Provide a value-based, non-intrusive abstraction for an extendable set of unrelated, potentially non-polymorphic types with the same semantic behavior.” 

```C++
//---- <Circle.h> ---------------------------------------------------------------------------------

class Circle
{
 public:
   explicit Circle( double radius )
      : radius_( radius )
   {}

   double radius() const { return radius_; }
   /* Several more getters and circle-specific utility functions */

 private:
   double radius_;
   /* Several more data members */
};


//---- <Square.h> ---------------------------------------------------------------------------------

class Square
{
 public:
   explicit Square( double side )
      : side_( side )
   {}

   double side() const { return side_; }
   /* Several more getters and square-specific utility functions */

 private:
   double side_;
   /* Several more data members */
};


//---- <Shape.h> ----------------------------------------------------------------------------------

#include <memory>
#include <utility>

namespace detail {

class ShapeConcept  // The External Polymorphism design pattern
{
 public:
   virtual ~ShapeConcept() = default;
   virtual void draw() const = 0;
   virtual std::unique_ptr<ShapeConcept> clone() const = 0;  // The Prototype design pattern
};

template< typename ShapeT
        , typename DrawStrategy >
class OwningShapeModel : public ShapeConcept
{
 public:
   explicit OwningShapeModel( ShapeT shape, DrawStrategy drawer )
      : shape_{ std::move(shape) }
      , drawer_{ std::move(drawer) }
   {}

   void draw() const override { drawer_(shape_); }

   std::unique_ptr<ShapeConcept> clone() const override  // The Prototype design pattern
   {
      return std::make_unique<OwningShapeModel>( *this );
   }

 private:
   ShapeT shape_;
   DrawStrategy drawer_;
};

} // namespace detail


class Shape
{
 public:
   template< typename ShapeT
           , typename DrawStrategy >
   Shape( ShapeT shape, DrawStrategy drawer )
   {
      using Model = detail::OwningShapeModel<ShapeT,DrawStrategy>;
      pimpl_ = std::make_unique<Model>( std::move(shape)
                                      , std::move(drawer) );
   }

   Shape( Shape const& other )
      : pimpl_( other.pimpl_->clone() )
   {}

   Shape& operator=( Shape const& other )
   {
      // Copy-and-Swap Idiom
      Shape copy( other );
      pimpl_.swap( copy.pimpl_ );
      return *this;
   }

   ~Shape() = default;
   Shape( Shape&& ) = default;
   Shape& operator=( Shape&& ) = default;

 private:
   friend void draw( Shape const& shape )
   {
      shape.pimpl_->draw();
   }

   std::unique_ptr<detail::ShapeConcept> pimpl_;  // The Bridge design pattern
};
```

### Shortcomings of the Type Erasure Design Pattern 
1. implementation complexity 
2. much more important and limiting: using ***Type Erasure*** for **binary operations**, such as **equality comparison**, is not straightforward 

```C++
int main()
{
    // ...
    if( shape1 == shape2 ) { /*...*/ } // Does not compile!
    return EXIT_SUCCESS;
}
```

## Guideline 33: Be Aware of the Optimization Potential of Type Erasure 
### Small Buffer Optimization 

optimizing memory allocations because acquiring and freeing dynamic memory can be very slow and nondeterministic. 

```C++
#include <array>
#include <cstddef>
#include <memory>
#include <utility>
template< size_t Capacity, size_t Alignment >
struct InClassStorage
{
    template< typename T, typename... Args >
    T* create( Args&&... args ) const
    {
        static_assert( sizeof(T) <= Capacity, "The given type is too large" );
        static_assert( alignof(T) <= Alignment, "The given type is misaligned" );
        T* memory = const_cast<T*>(reinterpret_cast<T const*> (buffer_.data()));
        return std::construct_at( memory, std::forward<Args>( args  )... );
        // or:
        // void* const memory = static_cast<void*>(buffer_.data());
        // return ::new (memory) T( std::forward<Args>( args )...
        );
    }
    template< typename T >
    void destroy( T* ptr ) const noexcept
    {
        std::destroy_at(ptr);
        // or: ptr->~T();
    }
    alignas(Alignment) std::array<std::byte,Capacity> buffer_;
};

template< typename StoragePolicy >
class Shape
{
public:
    template< typename ShapeT >
    Shape( ShapeT shape )
    {
        using Model = OwningModel<ShapeT>;
        pimpl_ = policy_.template create<Model>( std::move(shape) )
    }
    ~Shape() { policy_.destroy( pimpl_ ); }
    // ... All other member functions, in particular the
    // special members functions, are not shown
private:
    // ...
    [[no_unique_address]] StoragePolicy policy_{};
    Concept* pimpl_{};
};
```

### Manual Implementation of Function Dispatch 

optimization by reducing the two of indirections ( virtual function table and function pointer ) to just one (function pointer).  

```C++
//---- <Circle.h> ---------------------------------------------------------------------------------

class Circle
{
 public:
   explicit Circle( double radius )
      : radius_( radius )
   {}

   double radius() const { return radius_; }
   /* Several more getters and circle-specific utility functions */

 private:
   double radius_;
   /* Several more data members */
};


//---- <Square.h> ---------------------------------------------------------------------------------

class Square
{
 public:
   explicit Square( double side )
      : side_( side )
   {}

   double side() const { return side_; }
   /* Several more getters and square-specific utility functions */

 private:
   double side_;
   /* Several more data members */
};


//---- <Shape.h> ----------------------------------------------------------------------------------

#include <cstddef>
#include <memory>

class Shape
{
 public:
   template< typename ShapeT
           , typename DrawStrategy >
   Shape( ShapeT shape, DrawStrategy drawer )
      : pimpl_(
            new OwningModel<ShapeT,DrawStrategy>( std::move(shape)
                                                , std::move(drawer) )
          , []( void* shapeBytes ){
               using Model = OwningModel<ShapeT,DrawStrategy>;
               auto* const model = static_cast<Model*>(shapeBytes);
               delete model;
            } )
      , draw_(
            []( void* shapeBytes ){
               using Model = OwningModel<ShapeT,DrawStrategy>;
               auto* const model = static_cast<Model*>(shapeBytes);
               (model->drawer_)( model->shape_ );
            } )
      , clone_(
            []( void* shapeBytes ) -> void* {
               using Model = OwningModel<ShapeT,DrawStrategy>;
               auto* const model = static_cast<Model*>(shapeBytes);
               return new Model( *model );
            } )
   {}

   Shape( Shape const& other )
      : pimpl_( other.clone_( other.pimpl_.get() ), other.pimpl_.get_deleter() )
      , draw_ ( other.draw_ )
      , clone_( other.clone_ )
   {}

   Shape& operator=( Shape const& other )
   {
      // Copy-and-Swap Idiom
      using std::swap;
      Shape copy( other );
      swap( pimpl_, copy.pimpl_ );
      swap( draw_, copy.draw_ );
      swap( clone_, copy.clone_ );
      return *this;
   }

   ~Shape() = default;
   Shape( Shape&& ) = default;
   Shape& operator=( Shape&& ) = default;

 private:
   friend void draw( Shape const& shape )
   {
      shape.draw_( shape.pimpl_.get() );
   }

   template< typename ShapeT
           , typename DrawStrategy >
   struct OwningModel
   {
      OwningModel( ShapeT value, DrawStrategy drawer )
         : shape_( std::move(value) )
         , drawer_( std::move(drawer) )
      {}

      ShapeT shape_;
      DrawStrategy drawer_;
   };

   using DestroyOperation = void(void*);
   using DrawOperation    = void(void*);
   using CloneOperation   = void*(void*);

   std::unique_ptr<void,DestroyOperation*> pimpl_;
   DrawOperation*  draw_ { nullptr };
   CloneOperation* clone_{ nullptr };
};
```

## Guideline 34: Be Aware of the Setup Costs of Owning Type Erasure Wrappers 

### The Setup Costs of an Owning Type Erasure Wrapper 

The following is not cheap because **value semantics–based** ***Type Erasure*** have no **common base class**

```C++
#include <cstdlib>
class Circle { /*...*/ }; // Nonpolymorphic geometric primitive
class Shape { /*...*/ }; // Type erasure wrapper class as shown before
void useShape( Shape const& shape )
{
	draw(shape);
}
int main()
{
    Circle circle{ 3.14 };
    auto drawStrategy = []( Circle const& c ){ /*...*/ };
    // Creates a temporary 'Shape' object, involving a copy operation and a memory allocation
    useShape( { circle, drawStrategy } );
    return EXIT_SUCCESS;
}
```

### Nonowning Type Erasure Implementation 

`ShapeConstRef` is a wrapper class around the external hierarchy `ShapeConcept` and `NonOwningShapeModel` just like the `Shape` class wraps external hierarchy `ShapeConcept` and `OwningShapeModel`

functions not defined never participate in overload resolution while functions declared as ***deleted*** do, that is

1. move a class with move operator deleted cause a compile error
2. copy operations would be used instead if move a class which does not define move operator

```C++
//---- <Circle.h> ---------------------------------------------------------------------------------

class Circle
{
 public:
   explicit Circle( double radius )
      : radius_( radius )
   {}

   double radius() const { return radius_; }
   /* Several more getters and circle-specific utility functions */

 private:
   double radius_;
   /* Several more data members */
};


//---- <Square.h> ---------------------------------------------------------------------------------

class Square
{
 public:
   explicit Square( double side )
      : side_( side )
   {}

   double side() const { return side_; }
   /* Several more getters and square-specific utility functions */

 private:
   double side_;
   /* Several more data members */
};


//---- <Shape.h> ----------------------------------------------------------------------------------

#include <array>
#include <cstddef>
#include <memory>
#include <utility>

namespace detail {

class ShapeConcept  // The External Polymorphism design pattern
{
 public:
   virtual ~ShapeConcept() = default;
   virtual void draw() const = 0;
   virtual std::unique_ptr<ShapeConcept> clone() const = 0;  // The Prototype design pattern
   virtual void clone( ShapeConcept* memory ) const = 0;  // The Prototype design pattern
};

template< typename ShapeT, typename DrawStrategy > class NonOwningShapeModel;

template< typename ShapeT
        , typename DrawStrategy >
class OwningShapeModel : public ShapeConcept
{
 public:
   explicit OwningShapeModel( ShapeT shape, DrawStrategy drawer )
      : shape_{ std::move(shape) }
      , drawer_{ std::move(drawer) }
   {}

   void draw() const override { drawer_(shape_); }

   std::unique_ptr<ShapeConcept> clone() const override  // The Prototype design pattern
   {
      return std::make_unique<OwningShapeModel>( *this );
   }

   void clone( ShapeConcept* memory ) const
   {
      using Model = NonOwningShapeModel<ShapeT const,DrawStrategy const>;

      std::construct_at( static_cast<Model*>(memory), shape_, drawer_ );

      // or:
      // auto* ptr =
      //    const_cast<void*>(static_cast<void const volatile*>(memory));
      // ::new (ptr) Model( shape_, drawer_ );
   }

 private:
   ShapeT shape_;
   DrawStrategy drawer_;
};

template< typename ShapeT
        , typename DrawStrategy >
class NonOwningShapeModel : public ShapeConcept
{
 public:
   NonOwningShapeModel( ShapeT& shape, DrawStrategy& drawer )
      : shape_{ std::addressof(shape) }
      , drawer_{ std::addressof(drawer) }
   {}

   void draw() const override { (*drawer_)(*shape_); }

   std::unique_ptr<ShapeConcept> clone() const override
   {
      using Model = OwningShapeModel<ShapeT,DrawStrategy>;
      return std::make_unique<Model>( *shape_, *drawer_ );
   }

   void clone( ShapeConcept* memory ) const override
   {
      std::construct_at( static_cast<NonOwningShapeModel*>(memory), *this );

      // or:
      // auto* ptr = const_cast<void*>(static_cast<void const volatile*>(memory));
      // ::new (ptr) NonOwningShapeModel( *this );
   }

 private:
   ShapeT* shape_{ nullptr };
   DrawStrategy* drawer_{ nullptr };
};

} // namespace detail


class Shape;


class ShapeConstRef
{
 public:
   // Type 'ShapeT' and 'DrawStrategy' are possibly cv qualified;
   // lvalue references prevent references to rvalues
   template< typename ShapeT
           , typename DrawStrategy >
   ShapeConstRef( ShapeT& shape
                , DrawStrategy& drawer )
   {
      using Model =
         detail::NonOwningShapeModel<ShapeT const,DrawStrategy const>;
      static_assert( sizeof(Model) == MODEL_SIZE, "Invalid size detected" );
      static_assert( alignof(Model) == alignof(void*), "Misaligned detected" );

      std::construct_at( static_cast<Model*>(pimpl()), shape, drawer );

      // or:
      // auto* ptr =
      //    const_cast<void*>(static_cast<void const volatile*>(pimpl()));
      // ::new (ptr) Model( shape, drawer );
   }

   ShapeConstRef( Shape& other );
   ShapeConstRef( Shape const& other );

   ShapeConstRef( ShapeConstRef const& other )
   {
      other.pimpl()->clone( pimpl() );
   }

   ShapeConstRef& operator=( ShapeConstRef const& other )
   {
      // Copy-and-swap idiom
      ShapeConstRef copy( other );
      raw_.swap( copy.raw_ );
      return *this;
   }

   ~ShapeConstRef()
   {
      std::destroy_at( pimpl() );
      // or: pimpl()->~ShapeConcept();
   }

   // Move operations explicitly not declared

 private:
   friend void draw( ShapeConstRef const& shape )
   {
      shape.pimpl()->draw();
   }

   detail::ShapeConcept* pimpl()  // The Bridge design pattern
   {
      return reinterpret_cast<detail::ShapeConcept*>( raw_.data() );
   }

   detail::ShapeConcept const* pimpl() const
   {
      return reinterpret_cast<detail::ShapeConcept const*>( raw_.data() );
   }

   // Expected size of a model instantiation:
   //     sizeof(ShapeT*) + sizeof(DrawStrategy*) + sizeof(vptr)
   static constexpr size_t MODEL_SIZE = 3U*sizeof(void*);

   alignas(void*) std::array<std::byte,MODEL_SIZE> raw_;

   friend class Shape;
};


class Shape
{
 public:
   template< typename ShapeT
           , typename DrawStrategy >
   Shape( ShapeT shape, DrawStrategy drawer )
   {
      using Model = detail::OwningShapeModel<ShapeT,DrawStrategy>;
      pimpl_ = std::make_unique<Model>( std::move(shape)
                                      , std::move(drawer) );
   }

   Shape( Shape const& other )
      : pimpl_( other.pimpl_->clone() )
   {}

   Shape( ShapeConstRef const& other )
      : pimpl_{ other.pimpl()->clone() }
   {}

   Shape& operator=( Shape const& other )
   {
      // Copy-and-Swap Idiom
      Shape copy( other );
      pimpl_.swap( copy.pimpl_ );
      return *this;
   }

   ~Shape() = default;
   Shape( Shape&& ) = default;
   Shape& operator=( Shape&& ) = default;

 private:
   friend void draw( Shape const& shape )
   {
      shape.pimpl_->draw();
   }

   std::unique_ptr<detail::ShapeConcept> pimpl_;  // The Bridge design pattern

   friend class ShapeConstRef;
};


ShapeConstRef::ShapeConstRef( Shape& other )
{
   other.pimpl_->clone( pimpl() );
}

ShapeConstRef::ShapeConstRef( Shape const& other )
{
   other.pimpl_->clone( pimpl() );
}
```

# Chapter 9. The Decorator Design Pattern 
## Guideline 35: Use Decorators to Add Customization Hierarchically 
### The Decorator Design Pattern Explained 

The ***Decorator design pattern*** primary focus is the flexible combination of different pieces of functionality through composition, addition of new “responsibilities” is identified as a variation point: 

> Intent: “Attach additional responsibilities to an object dynamically. Decorators provide a flexible alternative to subclassing for extending functionality.” 

### A Classic Implementation of the Decorator Design Pattern  
```C++
//---- <Money.h> --------------------------------------------------------------------------------

#include <cmath>
#include <concepts>
#include <cstdint>
#include <ostream>

struct Money
{
   uint64_t value{};
};

template< typename T >
   requires std::is_arithmetic_v<T>
Money operator*( Money money, T factor )
{
   return Money{ static_cast<uint64_t>( money.value * factor ) };
}

constexpr Money operator+( Money lhs, Money rhs ) noexcept
{
   return Money{ lhs.value + rhs.value };
}

std::ostream& operator<<( std::ostream& os, Money money )
{
   return os << money.value;
}


//---- <Item.h> -----------------------------------------------------------------------------------

//#include <Money.h>

class Item
{
 public:
   virtual ~Item() = default;
   virtual Money price() const = 0;
};


//---- <DecoratedItem.h> --------------------------------------------------------------------------

//#include <Item.h>
#include <memory>
#include <stdexcept>
#include <utility>

class DecoratedItem : public Item
{
 public:
   explicit DecoratedItem( std::unique_ptr<Item> item )
      : item_( std::move(item) )
   {
      if( !item_ ) {
         throw std::invalid_argument( "Invalid item" );
      }
   }

 protected:
   Item&       item()       { return *item_; }
   Item const& item() const { return *item_; }

 private:
   std::unique_ptr<Item> item_;
};


//---- <CppBook.h> --------------------------------------------------------------------------------

//#include <Item.h>
#include <string>
#include <utility>

class CppBook : public Item
{
 public:
   CppBook( std::string title, Money price )
      : title_{ std::move(title) }
      , price_{ price }
   {}

   std::string const& title() const { return title_; }
   Money price() const override { return price_; }

 private:
   std::string title_{};
   Money price_{};
};


//---- <ConferenceTicket.h> -----------------------------------------------------------------------

//#include <Item.h>
#include <string>
#include <utility>

class ConferenceTicket : public Item
{
 public:
   ConferenceTicket( std::string name, Money price )
      : name_{ std::move(name) }
      , price_{ price }
   {}

   std::string const& name() const { return name_; }
   Money price() const override { return price_; }

 private:
   std::string name_{};
   Money price_{};
};


//---- <Discounted.h> -----------------------------------------------------------------------------

//#include <DecoratedItem.h>

class Discounted : public DecoratedItem
{
 public:
   Discounted( double discount, std::unique_ptr<Item> item )
      : DecoratedItem( std::move(item) )
      , factor_( 1.0 - discount )
   {
      if( !std::isfinite(discount) || discount < 0.0 || discount > 1.0 ) {
         throw std::invalid_argument( "Invalid discount" );
      }
   }

   Money price() const override
   {
      return item().price() * factor_;
   }

 private:
   double factor_;
};


//---- <Taxed.h> ----------------------------------------------------------------------------------

//#include <DecoratedItem.h>

class Taxed : public DecoratedItem
{
 public:
   Taxed( double taxRate, std::unique_ptr<Item> item )
      : DecoratedItem( std::move(item) )
      , factor_( 1.0 + taxRate )
   {
      if( !std::isfinite(taxRate) || taxRate < 0.0 ) {
         throw std::invalid_argument( "Invalid tax" );
      }
   }

   Money price() const override
   {
      return item().price() * factor_;
   }

 private:
   double factor_;
};


//---- <Main.cpp> ---------------------------------------------------------------------------------

//#include <ConferenceTicket.h>
//#include <CppBook.h>
//#include <Discounted.h>
//#include <Taxed.h>
#include <cstdlib>
#include <memory>

int main()
{
   // 7% tax: 19*1.07 = 20.33
   std::unique_ptr<Item> item1(
      std::make_unique<Taxed>( 0.07,
         std::make_unique<CppBook>( "Effective C++", Money{19} ) ) );

   // 20% discount, 19% tax: (999*0.8)*1.19 = 951.05
   std::unique_ptr<Item> item2(
      std::make_unique<Taxed>( 0.19,
         std::make_unique<Discounted>( 0.2,
            std::make_unique<ConferenceTicket>( "CppCon", Money{999} ) ) ) );

   Money const totalPrice1 = item1->price();  // Results in 20.33
   Money const totalPrice2 = item2->price();  // Results in 951.05

   // ...

   return EXIT_SUCCESS;
}
```

### Comparison Between Decorator, and Strategy  

***Strategy design pattern*** is focused on removing the dependencies on the implementation details of a specific functionality and enables you to define these details from the outside  

***Decorator design pattern*** is focused on removing the dependency between attachable pieces of implementation  

While ***Strategy*** is **intrusive** and requires the modification of a class, it’s always possible to **nonintrusively** add a ***Decorator***  

### Shortcomings of the Decorator Design Pattern  
1. the flexibility of a ***Decorator*** comes with a price: every level in a given hierarchy adds one level of indirection.  
2. potential danger of combining Decorators in a nonsensical way.  
3. **reference semantics–based** implementation of the ***classic Decorator design pattern***  

## Guideline 36: Understand the Trade-off Between Runtime and Compile Time  
### A Value-Based Compile Time Decorator  

performance improvemen due to the possibility of inlining and no pointer indirections, but with no runtime flexibility and longer compile times and more generated code   

```C++
//---- <Money.h> --------------------------------------------------------------------------------

#include <cmath>
#include <concepts>
#include <cstdint>
#include <ostream>

struct Money
{
   uint64_t value{};
};

template< typename T >
   requires std::is_arithmetic_v<T>
Money operator*( Money money, T factor )
{
   return Money{ static_cast<uint64_t>( money.value * factor ) };
}

constexpr Money operator+( Money lhs, Money rhs ) noexcept
{
   return Money{ lhs.value + rhs.value };
}

std::ostream& operator<<( std::ostream& os, Money money )
{
   return os << money.value;
}


//---- <ConferenceTicket.h> -----------------------------------------------------------------------

//#include <Money.h>
#include <string>
#include <utility>

class ConferenceTicket
{
 public:
   ConferenceTicket( std::string name, Money price )
      : name_{ std::move(name) }
      , price_{ price }
   {}

   std::string const& name() const { return name_; }
   Money price() const { return price_; }

 private:
   std::string name_;
   Money price_;
};


//---- <PricedItem.h> -----------------------------------------------------------------------------

//#include <Money.h>

template< typename T >
concept PricedItem =
   requires ( T item ) {
      { item.price() } -> std::same_as<Money>;
   };


//---- <Discounted.h> -----------------------------------------------------------------------------

//#include <Money.h>
//#include <PricedItem.h>
#include <utility>

template< double discount, PricedItem Item >
class Discounted  // Using composition
{
 public:
   template< typename... Args >
   explicit Discounted( Args&&... args )
      : item_{ std::forward<Args>(args)... }
   {}

   Money price() const {
      return item_.price() * ( 1.0 - discount );
   }

 private:
   Item item_;
};


//---- <Taxed.h> ----------------------------------------------------------------------------------

//#include <Money.h>
//#include <PricedItem.h>
#include <utility>

template< double taxRate, PricedItem Item >
class Taxed : private Item  // Using inheritance
{
 public:
   template< typename... Args >
   explicit Taxed( Args&&... args )
      : Item{ std::forward<Args>(args)... }
   {}

   Money price() const {
      return Item::price() * ( 1.0 + taxRate );
   }
};


//---- <Main.cpp> ---------------------------------------------------------------------------------

//#include <ConferenceTicket.h>
//#include <Discounted.h>
//#include <Taxed.h>
#include <cstdlib>

int main()
{
   // 20% discount, 15% tax: (499*0.8)*1.15 = 459.08
   Taxed<0.15,Discounted<0.2,ConferenceTicket>> item{ "Core C++", Money{499} };

   Money const totalPrice = item.price();  // Results in 459.08
    
   return EXIT_SUCCESS;
}
```

There are only five reasons to prefer **non-public inheritance** to **composition**

1.  If you have to override a virtual function
2. If you need access to a protected member function
3. If you need the adapted type to be constructed before another base class
4. If you need to share a common virtual base class or override the construction of a virtual base class
5. If you can draw significant advantage from the ***Empty Base Optimization (EBO)***  

### A Value-Based Runtime Decorator  

The following `Item` class implements an **owning Type Erasure wrapper**   

```C++
//---- <Money.h> --------------------------------------------------------------------------------

#include <cmath>
#include <concepts>
#include <cstdint>
#include <ostream>

struct Money
{
   uint64_t value{};
};

template< typename T >
   requires std::is_arithmetic_v<T>
Money operator*( Money money, T factor )
{
   return Money{ static_cast<uint64_t>( money.value * factor ) };
}

constexpr Money operator+( Money lhs, Money rhs ) noexcept
{
   return Money{ lhs.value + rhs.value };
}

std::ostream& operator<<( std::ostream& os, Money money )
{
   return os << money.value;
}


//---- <Item.h> ----------------

//#include <Money.h>
#include <memory>
#include <utility>

class Item
{
 public:
   template< typename T >
   Item( T item )
      : pimpl_( std::make_unique<Model<T>>( std::move(item) ) )
   {}

   Item( Item const& item ) : pimpl_( item.pimpl_->clone() ) {}

   Item& operator=( Item const& item )
   {
      pimpl_ = item.pimpl_->clone();
      return *this;
   }

   ~Item() = default;
   Item( Item&& ) = default;
   Item& operator=( Item&& item ) = default;

   Money price() const { return pimpl_->price(); }

 private:
   struct Concept
   {
      virtual ~Concept() = default;
      virtual Money price() const = 0;
      virtual std::unique_ptr<Concept> clone() const = 0;
   };

   template< typename T >
   struct Model : public Concept
   {
      explicit Model( T const& item ) : item_( item ) {}
      explicit Model( T&& item ) : item_( std::move(item) ) {}

      Money price() const override
      {
         return item_.price();
      }

      std::unique_ptr<Concept> clone() const override
      {
         return std::make_unique<Model<T>>(*this);
      }

      T item_;
   };

   std::unique_ptr<Concept> pimpl_;
};


//---- <ConferenceTicket.h> -----------------------------------------------------------------------

//#include <Money.h>
#include <string>
#include <utility>

class ConferenceTicket
{
 public:
   ConferenceTicket( std::string name, Money price )
      : name_{ std::move(name) }
      , price_{ price }
   {}

   std::string const& name() const { return name_; }
   Money price() const { return price_; }

 private:
   std::string name_;
   Money price_;
};


//---- <Discounted.h> -----------------------------------------------------------------------------

//#include <Item.h>
#include <utility>

class Discounted
{
 public:
   Discounted( double discount, Item item )
      : item_( std::move(item) )
      , factor_( 1.0 - discount )
   {}

   Money price() const
   {
      return item_.price() * factor_;
   }

 private:
   Item item_;
   double factor_;
};


//---- <Taxed.h> ----------------------------------------------------------------------------------

//#include <Item.h>
#include <utility>

class Taxed
{
 public:
   Taxed( double taxRate, Item item )
      : item_( std::move(item) )
      , factor_( 1.0 + taxRate )
   {}

   Money price() const
   {
      return item_.price() * factor_;
   }

 private:
   Item item_;
   double factor_;
};


//---- <Main.cpp> ---------------------------------------------------------------------------------

//#include <ConferenceTicket.h>
//#include <Discounted.h>
//#include <Taxed.h>
#include <cstdlib>

int main()
{
   // 20% discount, 15% tax: (499*0.8)*1.15 = 459.08
   Item item( Taxed( 0.15, Discounted(0.2, ConferenceTicket{ "Core C++", Money{499} } ) ) );

   Money const totalPrice = item.price();

   return EXIT_SUCCESS;
}
```

# Chapter 10. The Singleton Pattern  
***Singleton*** is not particularly popular and in many circles has a rather bad reputation  

## Guideline 37: Treat Singleton as an Implementation Pattern, Not a Design Pattern  
### The Singleton Pattern Explained  

> Intent: “Ensure a class has only one instance, and provide a global point of access to it.  

### Meyers’ Singleton  

There are multiple ways to restrict the number of instantiations to exactly one. Definitely one of the most useful and therefore most commonly used forms of Singleton is the ***Meyers’ Singleton***  

```C++
//---- <Database.h> ----------------
class Database final
{
public:
    static Database& instance()
    {
        //  first time control passes through the declaration, the variable is initialized in a thread-safe way
        static Database db; // The one, unique instance
        return db;
    }
    bool write( /*some arguments*/ );
    bool read( /*some arguments*/ ) const;
private:
    // the default constructor is explicitly defined and not defaulted,
    // to avoid create a Database with an empty set of braces, i.e., via value initialization:
    Database() {}
    Database( Database const& ) = delete;
    Database& operator=( Database const& ) = delete;
    Database( Database&& ) = delete;
    Database& operator=( Database&& ) = delete;
    // ... Potentially some data members
};
```

## Guideline 38: Design Singletons for Change and Testability  
One of the primary reasons why people dislike ***Singleton*** is that it often causes artificial dependencies and obstructs testability because there are strong and unfortunately even invisible dependencies from all over the code to the specific implementation details and design choices of the **only one** ***Singleton*** **instance**

### Applying the Strategy Design Pattern  

with this approach you can trivially avoid any dependency on the order of initialization of globals (i.e.,
SIOF), since you can explicitly manage the **initialization order** by creating all Singletons on the stack and in a **single compilation unit**:  

```C++
//---- <PersistenceInterface.h> -------------------------------------------------------------------

class PersistenceInterface
{
 public:
   virtual ~PersistenceInterface() = default;

   bool read( /*some arguments*/ ) const
   {
      return do_read( /*...*/ );
   }
   bool write( /*some arguments*/ )
   {
      return do_write( /*...*/ );
   }

   // ... More database specific functionality

 private:
   virtual bool do_read( /*some arguments*/ ) const = 0;
   virtual bool do_write( /*some arguments*/ ) = 0;
};

PersistenceInterface* get_persistence_interface();
void set_persistence_interface( PersistenceInterface* persistence );

// Declaration of the one 'instance' variable
extern PersistenceInterface* instance;


//---- <Database.h> -------------------------------------------------------------------------------

class Database : public PersistenceInterface
{
 public:
   Database() = default;

   // ... Potentially access to data members

   // Make the class immobile by deleting the copy and move operations
   Database( Database const& ) = delete;
   Database& operator=( Database const& ) = delete;
   Database( Database&& ) = delete;
   Database& operator=( Database&& ) = delete;

 private:
   bool do_read( /*some arguments*/ ) const override
   {
      /* Reading from the database */
      return true;
   }

   bool do_write( /*some arguments*/ ) override
   {
      /* Writing to the database */
      return true;
   }

   // ... More database-specific functionality

   // ... Potentially some data members
};


//---- <PersistenceInterface.cpp> -----------------------------------------------------------------

//#include <Database.h>

// Definition of the one 'instance' variable
PersistenceInterface* instance = nullptr;

PersistenceInterface* get_persistence_interface()
{
   // Local object, initialized by an 'Immediately Invoked Lambda Expression (IILE)'
   static bool init = [](){
      if( !instance ) {
         // create a default instance of the database
         static Database db;
         instance = &db;
      }
      return true;  // or false, as the actual value does not matter.
   }();  // Note the '()' after the lambda expression. This invokes the lambda.

   return instance;
}

void set_persistence_interface( PersistenceInterface* persistence )
{
   instance = persistence;
}


//---- <Widget.h> ---------------------------------------------------------------------------------

//#include <PersistenceInterface.h>

class Widget
{
 public:
   Widget( PersistenceInterface* persistence )  // Dependency injection
      : persistence_(persistence)
   {}

   void doSomething( /*some arguments*/ )
   {
      doSomething( get_persistence_interface() /*, some arguments*/ );
   }

   void doSomething( PersistenceInterface* persistence /*, some arguments*/ )
   {
      // ...
      persistence->read( /*some arguments*/ );
      // ...
   }

 private:
   PersistenceInterface* persistence_{};
};


//---- <CustomPersistence.h> ----------------------------------------------------------------------

//#include <PersistenceInterface.h>
#include <iostream>

class CustomPersistence : public PersistenceInterface
{
 public:
   CustomPersistence() = default;
   CustomPersistence( CustomPersistence const& db ) = default;

   bool do_read( /* some arguments */ ) const override
   {
      /* Reading from the custom persistence system */
      return true;
   }

   bool do_write( /* some arguments */ ) override
   {
      /* Writing to the custom persistence system */
      return true;
   }

   // ... More database specific functionality

 private:
   // ... Potential implementation details and data members
};


//---- <Main.cpp> ---------------------------------------------------------------------------------

//#include <PersistenceInterface.h>
#include <cstdlib>

int main()
{
   CustomPersistence persistence;
   set_persistence_interface( &persistence );

   Widget widget{ get_persistence_interface() };
   widget.doSomething();

   // ...

   return EXIT_SUCCESS;
}
```

# Chapter 11. The Last Guideline  

## Guideline 39: Continue to Learn About Design Patterns  
1. Minimize dependencies 
2. Separate concerns  
3. Prefer composition to inheritance 
4. Prefer a nonintrusive design 
5. Prefer value semantics over reference semantics  