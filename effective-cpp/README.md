# Effective Cpp

notes taken from reading the *Effective C++ Third Edition* by Scott Meyers.

## Introduction

- This book is not an introduction to C++ or a complete reference for it, but a guidance on how to develop better designs,
- Constructors declared **explicit** are usually preferable to non-explicit ones, because they prevent compilers from performing unexpected (often unintended) **type conversions**.

```cpp
class C {
    public:
        explicit C(int x); // not a default constructor
};
```

- Read carefully when you see what appears to be an assignment, because the “=” syntax can also be used to call the copy constructor:
`Widget w3 = w2; // invoke copy constructor!`.

```cpp
class Widget {
public:
    Widget(); // default constructor
    Widget(const Widget& rhs); // copy constructor
    Widget& operator=(const Widget& rhs); // copy assignment operator
};

Widget w1; // invoke default constructor
Widget w2(w1); // invoke copy constructor
w1 = w2; // invoke copy assignment operator
```

- If a new object is being defined (such as w3 in the statement above), a constructor **has to be called**; it can’t be an assignment. - If no new object is being defined (such as in the `w1 = w2` statement above), no constructor can be involved, so it’s an **assignment**.
- For example, consider this:
```cpp
bool hasAcceptableQuality(Widget w);
Widget aWidget;
if (hasAcceptableQuality(aWidget)) ...
```

- The parameter `w` is passed to `hasAcceptableQuality` by **value**, so in the call above, `aWidget` is copied into `w`. The copying is done by `Widget’s` copy constructor.
  -  ➡️ Pass-by-value means “call the copy constructor.”.

## Chapter 1: Accustoming Yourself to C++

### Item 1: View C++ as a federation of languages

- Today’s C++ is a multi-paradigm programming language, one supporting a combination of **procedural**, **object-oriented**, **functional**, **generic**, and **meta-programming** features.
- Rules for effective C++ programming vary, depending on the part of C++ you are using: **C**, **Object-Oriented C++**, **Template C++** and The **STL**.
> For example, pass-by-value is generally more efficient than pass-by-reference for built-in (i.e., C-like) types, but when you move from the C part of C++ to Object-Oriented C++, the existence of user-defined constructors and destructors means that pass-by-reference-to-const is usually better. This is especially the case when working in Template C++, because there, you don’t even know the type of object you’re dealing with. When you cross into the STL, however, you know that iterators and function objects are modeled on pointers in C, so for iterators and function objects in the STL, the old C pass-by-value rule applies again.

### Item 2: Prefer consts, enums, and inlines to #defines

```cpp
#define authorName "Scott Meyers"; // bad
const char * const authorName = "Scott Meyers"; // better
const std::string authorName("Scott Meyers"); // best
```

- C++ requires that you provide a **definition** for anything you use, but **class-specific constants** that are static and of integral type (e.g., integers, chars, bools) are an exception.
- To limit the scope of a constant to a **class**, you must make it a member, and to ensure there’s at most one **copy** of the constant, you must make it a **static** member.

```cpp
class GamePlayer {
private:
    // As long as you don’t take their address,
    // you can declare them and use them without providing a definition
    static const int NumTurns = 5; // constant declaration
    int scores[NumTurns]; // use of constant
...

// Otherwise you should define them in an implementation file:
const int GamePlayer::NumTurns; // definition of NumTurns;
};
```

- ⚠️ `#defines` don’t respect **scope**. Once a macro is defined, it’s in force for the rest of the compilation (until `#undef`).
  - ▶️ `#defines `can't be used for class-specific constants, they also can’t be used to provide any kind of **encapsulation**.
- 🌟 If you don’t want to let people get a pointer or reference to one of your integral constants, an **enum** is a good way to enforce that constraint.
- Macros like this have so many drawbacks, just thinking about them is painful:
```cpp
#define CALL_WITH_MAX(a, b) f((a) > (b) ? (a) : (b))
```
- ⚠️ Whenever you write this kind of macro, you have to remember to **parenthesize** all the arguments in the macro body. Otherwise you can run into trouble when somebody calls the macro with an **expression**.
- You can get all the efficiency of a macro plus all the predictable behavior and type safety of a regular function by using a **template** for an **inline** function.
```cpp
- template<typename T> // because we don’t know what T is, we pass by reference-to const
inline void callWithMax(const T& a, const T& b) {
    f(a > b ? a : b);
}
```

📆 Things to Remember:
- For simple constants, prefer const objects or enums to `#defines`.
- For function-like macros, prefer inline functions to `#defines`.

### Item 3: : Use const whenever possible

- Some programmers list const before the type. Others list it after the type but before the **asterisk**. There is no difference in meaning, so the following functions take the same parameter type:
```cpp
void f1(const Widget *pw); // f1 takes a pointer to a constant Widget object
void f2(Widget const *pw); // so does f2
```
- STL iterators are modeled on **pointers**, so an iterator acts much like a _T* pointer_.
- If you want an iterator that points to something that can’t be modified (i.e., the STL analogue of a const T* pointer), you want a *const_iterator*:
```cpp
...std::vector<int> vec;
const std::vector<int>::iterator iter = // iter acts like a T* const
vec.begin();
*iter = 10; // OK, changes what iter points to
++iter; // error! iter is const
std::vector<int>::const_iterator cIter = // cIter acts like a const T*
vec.begin();
*cIter = 10; // error! *cIter is const
++cIter; // fine, changes cIter
```
- 😲 Many people overlook the fact that member functions differing only in their **constness** can be **overloaded**, but this is an important feature of C++.
- It’s never **legal** to modify the return value of a function that returns a **built-in** type. Even if it were legal, the fact that C++ returns objects by value would mean that a copy of `tb.text[0]` would be modified, not `tb.text[0]` itself, and that’s not the behavior you want.
-  C++’s definition of **member function constness** in a member function is `const` if and only if it doesn’t modify any of the object’s data members (excluding those that are static).
   - :facepalm: Unfortunately, many member functions that don’t act very `const` pass this test !
   - Example: function returns a reference to the object’s internal data.
   - ▶️ a `const` member function might modify some of the bits in the object on which it’s invoked, but only in ways that clients cannot detect.

📆 Things to Remember
   - Declaring something `const` helps compilers detect usage errors. `const` can be applied to objects at any scope, to function parameters and return types, and to member functions as a whole.
   - Compilers enforce **bitwise constness**, but you should program using **logical constness**.
   - When `const` and `non-const` member functions have essentially identical implementations, code duplication can be avoided by having the non-const version call the const version.

### Item 4: Make sure that objects are initialized before they’re used

- There are rules that describe when object initialization is guaranteed to take place and when it isn’t.
- Unfortunately, the rules are **complicated** — too complicated to be worth memorizing.
- ⚠️ it’s important not to confuse **assignment** with **initialization**.
```cpp
ABEntry::ABEntry(const std::string& name, const std::string& address, const std::list<PhoneNumber>& phones)
  // these are now all initializations
  : theName(name), theAddress(address), thePhones(phones), numTimesConsulted(0)
{} // the ctor body is now empty
```
- In this case, for example `theName` is copy-constructed from `name`. For most types, a single call to a **copy constructor** is more efficient —
sometimes much more efficient — than a call to the **default constructor** followed by a call to the **copy assignment operator**.
- **Base classes** are always initialized before **derived** classes, and within a class, **data members** are initialized in the order in which they are **declared** ( true even if they are listed in a different order on the member initialization list 😲!)
- ⚠️the relative order of initialization of **non-local static objects** defined in different translation units is **undefined**.

📆 Things to Remember:
- Manually initialize objects of built-in type, because C++ only sometimes initializes them itself.
- In a constructor, prefer use of the **member initialization** list to assignment inside the body of the constructor. List data members in the initialization list in the **same order** they’re declared in the class.
- Avoid initialization order problems across translation units by replacing non-local static objects with local static objects.

## Chapter 2: Constructors, Destructors, and Assignment Operators

### Item 5: Know what functions C++ silently writes and calls.

- When is an empty class not an empty class? When C++ gets through with it. If you don’t declare them yourself, compilers will declare their own versions of a copy constructor, a copy assignment operator, and a destructor. Furthermore, if you declare no constructors at all, compilers will also declare a default constructor for you.
-  As a result, if you write `class Empty{};`, it’s essentially the same as if you’d written this:
```cpp
class Empty {
public:
  Empty() { ... } // default constructor
  Empty(const Empty& rhs) { ... } // copy constructor
  ~Empty() { ... } // destructor — see below // for whether it’s virtual
  Empty& operator=(const Empty& rhs) { ... } // copy assignment operator
};
```
- ⚠️ If you’ve carefully engineered a class to require constructor arguments, you don’t have to worry about compilers **overriding** your decision by blithely adding a constructor that takes no arguments.
- Compilers will refuse to generate an implicit operator= for your class if the resulting code has a reasonable chance of making sense, for example:
  - classes containing **references** members
  - classes containing **const** members
  - **derived** classes that inherit from base classes declaring the **copy** assignment operator **private**.

📆 Things to Remember
- Compilers may implicitly generate a class’s **default** constructor, **copy** constructor, **copy assignment** operator, and **destructor**.

### Item 6: Explicitly disallow the use of compiler-generated functions you do not want

- By declaring a member function explicitly, you prevent compilers from generating their own version, and by making the function private, you keep people from calling it
```cpp
class HomeForSale {
public:
  ...
private:
  // declarations only
  HomeForSale(const HomeForSale&);
  HomeForSale& operator=(const HomeForSale&);
};
```
- It’s possible to move the **link-time** error up to **compile time** (:+1: always a good thing — earlier error detection is better than later) by declaring the copy constructor and copy assignment operator **private** not in `HomeForSale` itself, but in a **base** class specifically designed to prevent copying.
- The base class is simplicity itself:
```cpp
class Uncopyable {
protected:
// allow construction and destruction of derived objects...
  Uncopyable() {}
  ~Uncopyable() {}
private: // ...but prevent copying
  Uncopyable(const Uncopyable&);
  Uncopyable& operator=(const Uncopyable&);
};
```
- To keep `HomeForSale` objects from being copied, all we have to do now is inherit from `Uncopyable`:
```cpp
class HomeForSale: private Uncopyable {
// class no longer declares copy ctor or copy assign. operator
...
};
```

📆 Things to Remember
- To disallow functionality automatically provided by compilers, declare the corresponding member functions **private** and give no implementations. Using a base class like `Uncopyable` is one way to do this.

### Item 7: Declare destructors virtual in polymorphic base classes

- C++ specifies that when a **derived** class object is **deleted** through a pointer to a **base** class with a non-virtual destructor, results are **undefined**.
```cpp
class TimeKeeper {
public:
  TimeKeeper();
  virtual ~TimeKeeper();
...
};

TimeKeeper *ptk = getTimeKeeper();
...
delete ptk; // now behaves correctly
```
- If a class does not contain virtual functions, that often indicates it is not meant to be used as a base class. When a class is not intended to be a base class, making the destructor virtual is usually a bad idea.
```cpp
class SpecialString: public std::string { // bad idea! std::string has a non-virtual destructor
};
```
- The same analysis applies to any class lacking a virtual destructor, including all the *STL* container types (e.g., vector, list, set, tr1::unordered_map, etc.).

📆 Things to Remember
- **Polymorphic** base classes should declare **virtual destructors**. If a class has any virtual functions, it should have a virtual destructor.
- Classes not designed to be base classes (such as STL container types) or not designed to be used polymorphically should not declare virtual destructors.

### Item 8: Prevent exceptions from leaving destructors

- C++ does not like destructors that emit exceptions! ⚠️ yields undefined behavior or premature program termination.
- If an operation may fail by throwing an exception and there may be a need to handle that exception, the exception has to come from some **non-destructor** function:
```cpp
class DBConn {
public:
  void close() {   // create an opportunity to react to problems that may arise:
    db.close();    // giving clients a chance to handle exceptions arising from
    closed = true; // that operation.
  }
  ~DBConn() {
    if (!closed) {
      try {
        db.close(); // close the connection if the client didn’t
      }
      catch (...) { // if closing fails, note that and terminate or swallow
      // make log entry that call to close failed;
      }
    }
  }
}
private:
  DBConnection db;
  bool closed;
};
```

📆 Things to Remember
- Destructors should **never** emit **exceptions**. If functions called in a destructor may throw, the destructor should catch any exceptions, then swallow them or terminate the program.
- If class clients need to be able to react to exceptions thrown during an operation, the class should provide a regular (i.e., non-destructor) function that performs the operation.

### Item 9: Never call virtual functions during construction or destruction

- During base class construction, **virtual** functions never **go down** into **derived** classes. Instead, the object behaves as if it were of the base type.
- ▶️ An object doesn’t become a derived class object until execution of a derived class constructor begins.
- 👨‍🏫 The same reasoning applies during **destruction**.
- There are different ways to approach this problem. One is to turn `logTransaction` into a **non-virtual** function in `Transaction`, then require that derived class constructors pass the necessary log information to the `Transaction` constructor.

📆 Things to Remember
- Don’t call virtual functions during construction or destruction, because such calls will never go to a more derived class than that of the currently executing constructor or destructor.

### Item 10: Item 10: Have assignment operators return a reference to *this

```cpp
class Widget {
public:
  ...
  // return type is a reference to the current class
  Widget& operator=(const Widget& rhs) {
  ...
  return *this; // return the left-hand object
}
...
};
```

- This convention applies to all **assignment** operators, not just the standard form shown above. Hence:
```cpp
class Widget {
  public:
  ...
  Widget& operator+=(const Widget& rhs) // the convention applies to
  {                                     // +=, -=, *=, etc.
    ...
    return *this;
}

  Widget& operator=(int rhs) // it applies even if the
  {                          // operator’s parameter type
  ...                        // is unconventional
  return *this;
  }
...
};
```

📆 Things to Remember
- Have assignment operators return a reference to `*this.`

### Item 11: Handle assignment to self in operator=

- Example of assignment to self:
```cpp
a[i] = a[j]; // assignment to self if i and j have the same value
*px = *py; // assignment to self if px and py happen to point to the same thing
```
- ‼️ In general, code that operates on references or pointers to **multiple objects** of the **same type** needs to consider that the objects might be the same.
- Here’s an implementation of `operator=` that looks reasonable on the surface but is **unsafe** in the presence of assignment to self.

```cpp
Widget&
Widget::operator=(const Widget& rhs) // unsafe impl. of operator=
{
  delete pb; // stop using current bitmap
  pb = new Bitmap(*rhs.pb); // start using a copy of rhs’s bitmap
  return *this; // see Item 10
}
```
- The self-assignment problem here is that inside `operator=`, `*this`  and `rhs` could be the same object.
  - ▶️ At the end of the function, the `Widget` — which should not have been changed by the assignment to self — finds itself holding a pointer to a deleted object!
- Solution:
  - Do an identity test at the top of operator=: `if (this == &rhs) return *this;`. This work, but it is still **exception-unsafe**.
  - A careful **ordering** of statements can yield exception-safe:
```cpp
Widget& Widget::operator=(const Widget& rhs)
{
  Bitmap *pOrig = pb; // remember original pb
  pb = new Bitmap(*rhs.pb); // point pb to a copy of rhs’s bitmap
  delete pOrig; // delete the original pb until we've copied what it points to
  return *this;
}
```
- An alternative to the implementation above that is both exception and self-assignment-safe is to use the technique known as **copy and swap**:
```cpp
class Widget {
...
void swap(Widget& rhs); // exchange *this’s and rhs’s data;
  ... // see Item 29 for details
};
Widget& Widget::operator=(const Widget& rhs)
{
  Widget temp(rhs); // make a copy of rhs’s data
  swap(temp); // swap *this’s data with the copy’s
  return *this;
}
```

📆 Things to Remember
- Make sure `operator=` is **well-behaved** when an object is assigned to itself.
  - Techniques include **comparing** addresses of source and target objects, careful statement ordering, and copy-and-swap.
- Make sure that any function operating on **more than one** object behaves correctly if two or more of the objects are the same.

### Item 12: Copy all parts of an object

- When you declare your own copying functions, and you **later add** a data **member** to the class, you need to make sure that you update the copying functions too.
- One of the most insidious ways this issue can arise is through inheritance:
```cpp
class PriorityCustomer : public Customer {
  public:
    PriorityCustomer(const PriorityCustomer& rhs);
    PriorityCustomer &operator=(const PriorityCustomer& rhs);
  private:
    int priority;
}

PriorityCustomer::PriorityCustomer(const PriorityCustomer& rhs)
    : Customer(rhs), // invoke base class copy ctor <- (might be forgotten)
    priority(rhs.priority) {
  logCall("PriorityCustomer copy constructor");
}

PriorityCustomer&
PriorityCustomer::operator=(const PriorityCustomer& rhs) {
  logCall("PriorityCustomer copy assignment operator");
  Customer::operator=(rhs); // assign base class parts <- (might be forgotten)
  priority = rhs.priority;
  return *this;
}
```

📆 Things to Remember
- Copying functions should be sure to **copy** all of an **object’s data members** and all of its **base class** parts.
- Don’t try to implement one of the copying functions in terms of the other. Instead, put **common functionality** in a third function that both call.

## Resource Chapter 3: Resource Management

### Item 13: Use objects to manage resources

- Consider the function `f` below:
```cpp
void f() {
  Investment *pInv = createInvestment(); // call factory function
  ... // use pInv
  delete pInv; // release object
}
```
- Relying on `f` always getting to its `delete` statement simply isn’t viable because:
  - a premature return statement.
  - if the uses of `createInvestment` and `delete` were in a loop, and the loop was prematurely exited by a break or goto statement.
  - “...” might throw an exception.
  - think about how the code might change over time:
    - somebody might add a `return` or `continue` statement
    - the “...” part of `f` might call a function that never used to throw an exception but suddenly starts doing so after it has been “improved.”
- 👎 Regardless of how the `delete` were to be skipped, we’d **leak** not only the memory containing the investment object but also any **resources** **held** by that object.
- 👍 Solution is to put that resource inside an object whose destructor will **automatically release** the resource when control leaves `f` : `std::auto_ptr<Investment> pInv(createInvestment());`.

📆 Things to Remember
- To prevent resource leaks, use *RAII* (Resource acquisition is initialization) objects that acquire resources in their constructors and release them in their destructors.
- Two commonly useful RAII classes are `tr1::shared_ptr` and `auto_ptr`. `tr1::shared_ptr` is usually the better choice, because its behavior when copied is intuitive. Copying an `auto_ptr` sets it to null.

### Item 14: Think carefully about copying behavior in resource-managing classes.

- Every RAII class author must confront: *what should happen when an RAII object is copied?* Most of the time, you’ll want to choose one of the following possibilities:
  - **Prohibit copying**: likely to be true for a class like `Lock` (contains a mutex). When copying makes no sense for an RAII class, you
should prohibit it.
  - **Reference-count the underlying resource**: when needed, copying an RAII object should increment the count of the number of objects referring to the resource (using a `tr1::shared_ptr`). However, in the `Lock` example, when we’re done with a `Mutex`, we want to unlock it, not delete it (default behavior for `tr1::shared_ptr`).
  - **Copy the underlying resource**: In that case, copying the resource-managing object should also copy the resource it wraps. That is, copying a resource-managing object performs a **deep copy**, for example, when a string object is copied, a copy is made of both the pointer and the memory it points to.
  - **Transfer ownership of the underlying resource**: Sometimes you may wish to make sure that only **one** RAII object refers to a raw resource and that when the RAII object is copied, ownership of the resource is transferred from the copied object to the copying object (`auto_ptr`).

📆 Things to Remember
- Copying an RAII object entails copying the resource it manages, so the copying behavior of the resource determines the copying behavior of the RAII object.
- Common RAII class copying behaviors are disallowing copying and performing reference counting, but other behaviors are possible.

### Item 15: Provide access to raw resources in resource-managing classes.

- `tr1::shared_ptr` and `auto_ptr` both offer a `get()` to perform an **explicit** conversion, i.e., to return (a copy of) the raw pointer
inside the smart pointer object.
- Like virtually all smart pointer classes, `tr1::shared_ptr` and `auto_ptr` also **overload** the pointer dereferencing operators (operator-> and operator*), and this allows **implicit** conversion to the underlying raw pointers.
- Some programmers might find the need to **explicitly** request conversions **off-putting** enough to avoid using the class. That, in turn,
would increase the chances of leaking fonts 🤷. The alternative is to offer an implicit conversion function:
```cpp
class Font {
  public:
  ...
    operator FontHandle() const { // implicit conversion function
      return f;
    }
...
};
```
- That makes calling into the C API easy and natural, but can increase the chance of errors:
  - ▶️ For example, a client might accidentally create a `FontHandle` when a `Font` object was intended: `FontHandle f2 = f1;`.
- The best design is likely to make interfaces **easy** to use **correctly** and **hard** to use **incorrectly**.

📆 Things to Remember

- APIs often require access to raw resources, so each RAII class should offer a way to get at the resource it manages.
- Access may be via explicit conversion or implicit conversion. In general, **explicit** conversion is **safer**, but **implicit** conversion is more **convenient** for clients.

### Item 16: Use the same form in corresponding uses of new and delete.

- Does the pointer being deleted point to a **single** object or to an **array** of objects?
  - Why ? ▶️ the memory layout for single objects is generally different from the memory layout for arrays.
  - `delete []` includes the size of the array making it easy for `delete` to know **how many destructors** to call.
- Abstain from `typedefs` for array types
```cpp
typedef std::string AddressLines[4];    // a person’s address has 4 lines,
                                        // each of which is a string
  // Because AddressLines is an array, this use of new,
  std::string *pal = new AddressLines;  // note that “new AddressLines”
                                        // returns a string*, just like
                                        // “new string[4]” would
  // must be matched with the array form of delete:
  delete pal; // undefined!
  delete [] pal; // fine
```

📆 Things to Remember
- If you use `[]` in a `new` expression, you must use `[]` in the corresponding `delete` expression.
- If you don’t use `[]` in a new expression, you mustn’t use `[]` in the corresponding `delete` expression.

### Item 17: Store newed objects in smart pointers in standalone statements.

- Consider the following code to call the function `processWidget`:
```cpp
processWidget(std::tr1::shared_ptr<Widget>(new Widget), priority())
```
- ‼️ A **leak** in the call to `processWidget` can arise because an **exception** can intervene between the time a resource is created (via `new Widget`) and the time that resource is turned over to a resource-managing object.
- 👍 Use a separate statement to create the `Widget` and store it in a smart pointer, then pass the smart pointer to `processWidget`:
```cpp
std::tr1::shared_ptr<Widget> pw(new Widget); // store newed object in a smart pointer in a  standalone statement
processWidget(pw, priority()); // this call won’t leak
```

📆 Things to Remember
- Store newed objects in smart pointers in **standalone statements**.
- Failure to do this can lead to subtle resource **leaks** when exceptions are thrown.

## Chapter 4: Design and Declarations

### Item 18: Make interfaces easy to use correctly and hard to use incorrectly.

- Many client errors can be prevented by the introduction of **new types**. Indeed, the type system is your primary ally in preventing undesirable code from **compiling**.

```cpp
class Date {
  public:
    Date(const Month& m, const Day& d, const Year& y);
...
};
Date d(30, 3, 1995); // error! wrong types
Date d(Day(30), Month(3), Year(1995)); // error! wrong types
Date d(Month(3), Day(30), Year(1995)); // okay,
```
- Another way to prevent likely client errors is to **restrict** what can be done with a **type**.
  - ▶️ A common way to impose restrictions is to add **const**.
- Unless there’s a good reason not to, have your types behave **consistently** with the **built-in types**:
  - :100: Inconsistency imposes mental friction into a developer’s work that no IDE can fully remove !
- 👍 Any interface that requires that clients **remember** to do something is prone to **incorrect use**, because clients can forget to do it.
  - `std::tr1::shared_ptr<Investment> createInvestment();` essentially forces clients to store the return value in a `tr1::shared_ptr`, all but **eliminating** the possibility of **forgetting** to **delete** the underlying `Investment` object when it’s no longer being used.

📆 Things to Remember
- Good interfaces are easy to use correctly and hard to use incorrectly.
- Ways to facilitate correct use include consistency in interfaces and behavioral compatibility with built-in types.
- Ways to prevent errors include **creating new types**, **restricting** operations on types, **constraining** object values, and eliminating **client resource management** responsibilities.
- `tr1::shared_ptr` supports custom deleters. This prevents the *crossDLL* problem, can be used to automatically unlock mutexes.

### Item 19: Treat class design as type design.

- Virtually every class requires that you confront the following questions, the answers to which often lead to constraints on your design:
  - How should objects of your new type be **created** and **destroyed**?
  - How should object **initialization** differ from object **assignment**?
  - What does it mean for objects of your new type to **be passed by value**?
  - What are the **restrictions** on **legal values** for your new type?
  - Does your new type fit into an **inheritance** graph?
  - What kind of **type conversions** are **allowed** for your new type?
  - What **operators** and **functions** make sense for the new type?
  - What **standard functions** should be **disallowed**?
  - Who should have **access** to the **members** of your new type?
  - What is the *undeclared interface* of your new type?
  - How **general** is your new type?
  - Is a **new type** really what you **need**?

📆 Things to Remember
- **Class design is type design**. Before defining a new type, be sure to consider all the issues discussed in this Item.

### Item 20: Prefer pass-by-reference-to-const to pass-by-value.

- Consider the following code:
```cpp
class Person {
  public:
    Person(); // parameters omitted for simplicity
    virtual ~Person(); // see Item 7 for why this is virtual
...
  private:
    std::string name;
    std::string address;
};

class Student: public Person {
  public:
    Student(); // parameters again omitted
    virtual ~Student();
  ...
  private:
    std::string schoolName;
    std::string schoolAddress;
};
```
- Invoking such call `bool validateStudent(Student s);` will result on:
  - the `Student` copy constructor is called to initialize the parameter `s` from *plato*.
  - every time you construct a `Student` object you must also construct two string objects ‼️.
  - and every time you construct a `Student` object you must also construct a `Person` object, so each `Person` construction also entails two more string constructions.
  - `s` is destroyed when `validateStudent` returns.
- ▶️ When the copy of the `Student` object is **destroyed**, each constructor call is matched by a destructor call, so the overall cost of passing a `Student` by value is **six constructors** and **six destructors**! 🤷.
- The solution is to use `bool validateStudent(const Student& s);` but remember to declare it **const**, because otherwise callers would have to worry about `validateStudent` making changes to the `Student` they passed in.
- 👍 Passing parameters by reference also avoids the **slicing problem**. When a derived class object is passed (by value) as a base class object, the base class copy constructor is called, and the specialized features that make the object behave like a derived class object are “sliced” off.
- ⚠️ Just because an object is **small** doesn’t mean that calling its copy constructor is **inexpensive**. Many objects — most STL containers among them — contain little more than a pointer, but copying such objects entails **copying everything they point to**. That can be very expensive.
- ⚠️ Even when small objects have inexpensive copy constructors, some compilers refuse to put objects consisting of only a double into a register.
- ⚠️ Being user-defined, their size is subject to change. A type that’s small now may be bigger in a future release, because its internal implementation may change. Things can even change when you switch to a different C++ implementation 👁️.

📆 Things to Remember
- Prefer **pass-by-reference-to-const** over **pass-by-value**. It’s typically more efficient and it avoids the *slicing problem*.
- The rule doesn’t apply to **built-in** types and **STL** iterator and **function object** types. For them, **pass-by-value** is usually appropriate.

### Item 21: Don’t try to return a reference when you must return an object.

- Consider the following code:
```cpp
const Rational& operator*(const Rational& lhs, const Rational& rhs)
{
  Rational *result = new Rational(lhs.n * rhs.n, lhs.d * rhs.d);
  return *result;
}

Rational w, x, y, z;
w = x * y * z; // same as operator*(operator*(x, y), z)
```
- This is a guaranteed resource leak because of hidden pointers, i.e: `x * y`.
- If we use a static variable `static Rational result;` tot return the resulted object, it violates **thread safety**, plus expressions like this: `Rational a, b, c, d; if ((a * b) == (c * d))` will always evaluate to true :-1:.

📆 Things to Remember
- Never return a **pointer** or **reference** to a **local stack** object, a **reference** to a **heap-allocated** object, or a **pointer** or **reference** to a **local static** object if there is a chance that more than one such object will be needed.

### Item 22: Declare data members private.

- If data members aren’t public, the only way for clients to access an object is via **member functions** (▶️ uniform access).
- Hiding data members behind functional interfaces can offer all kinds of implementation **flexibility**.
  - notify other objects when data members are read or written
  - ensure that class invariants are always maintained
  - perform synchronization in threaded environments
  - 👍 reserve the right to change your implementation decisions later ...
- Protected data members are as **unencapsulated** as **public** ones, because in both cases, if the data members are changed, an unknowably large amount of client code is broken 🤷.

📆 Things to Remember
- Declare data members **private**. It gives clients syntactically **uniform access** to data, affords **fine-grained access control**, allows **invariants** to be **enforced**, and offers class authors implementation **flexibility**.
- **Protected** is no more **encapsulated** than public.

### Item 23: Prefer non-member non-friend functions to member functions.

- OO principles dictate that data and the functions that operate on them should be bundled together, and that suggests that the member function is the better choice. Unfortunately, this suggestion is incorrect 🤷.
- OO principles dictate that data should be as **encapsulated** as possible.
  - 👍 The more something is encapsulated, the **fewer** things can **see** it. The fewer things can see it, the greater **flexibility** we have to change it.
  - 👍 The **less** code that can see the data (i.e., access it), the more the data is **encapsulated**.
  - As a coarse-grained measure of how much code can see a piece of data, we can count the **number of functions** that can access that data: ▶️ the **more functions** that can access it, the **less encapsulated** the data !
- To solve this, we can:
  - make `clearBrowser` a static member function of some utility class, As long as it’s not part of (or a friend of) `WebBrowser`.
  - more natural approach would be to make `clearBrowser` a non-member function in the **same namespace** as `WebBrowser`:

```cpp
namespace WebBrowserStuff {
  class WebBrowser { ... };
  void clearBrowser(WebBrowser& wb);
...
}
```
- Namespaces, unlike classes, can be spread across multiple source files.
  - 👍 Putting all convenience functions in multiple header files

📆 Things to Remember
- Prefer non-member non-friend functions to member functions. Doing so increases encapsulation, packaging flexibility, and functional
extensibility.
