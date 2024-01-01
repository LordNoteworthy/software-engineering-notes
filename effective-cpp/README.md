# Effective Cpp

notes taken from reading the *Effective C++ Third Edition* by Scott Meyers.

## Introduction

- This book is not an introduction to C++ or a complete reference for it, but a guidance on how to develop better designs,
- Constructors declared **explicit** are usually preferable to non-explicit ones, because they prevent compilers from performing unexpected (often unintended) **type conversions**.

```c
class C {
    public:
        explicit C(int x); // not a default constructor
};
```

- Read carefully when you see what appears to be an assignment, because the “=” syntax can also be used to call the copy constructor:
`Widget w3 = w2; // invoke copy constructor!`.

```c
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
```c
bool hasAcceptableQuality(Widget w);
Widget aWidget;
if (hasAcceptableQuality(aWidget)) ...
```

- The parameter `w` is passed to `hasAcceptableQuality` by **value**, so in the call above, `aWidget` is copied into `w`. The copying is done by `Widget’s` copy constructor. ➡️ Pass-by-value means “call the copy constructor.”.

## Chapter 1: Accustoming Yourself to C++

### Item 1: View C++ as a federation of languages.

- Today’s C++ is a multi-paradigm programming language, one supporting a combination of **procedural**, **object-oriented**, **functional**, **generic**, and **meta-programming** features.
- Rules for effective C++ programming vary, depending on the part of C++ you are using: **C**, **Object-Oriented C++**, **Template C++** and The **STL**.
> For example, pass-by-value is generally more efficient than pass-by-reference for built-in (i.e., C-like) types, but when you move from the C part of C++ to Object-Oriented C++, the existence of user-defined constructors and destructors means that pass-by-reference-to-const is usually better. This is especially the case when working in Template C++, because there, you don’t even know the type of object you’re dealing with. When you cross into the STL, however, you know that iterators and function objects are modeled on pointers in C, so for iterators and function objects in the STL, the old C pass-by-value rule applies again.

### Item 2: Prefer consts, enums, and inlines to #defines.

```c
#define authorName "Scott Meyers"; // bad
const char * const authorName = "Scott Meyers"; // better
const std::string authorName("Scott Meyers"); // best
```

- C++ requires that you provide a **definition** for anything you use, but **class-specific constants** that are static and of integral type (e.g., integers, chars, bools) are an exception.
- To limit the scope of a constant to a **class**, you must make it a member, and to ensure there’s at most one **copy** of the constant, you must make it a **static** member.

```c
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
```c
#define CALL_WITH_MAX(a, b) f((a) > (b) ? (a) : (b))
```
- ⚠️ Whenever you write this kind of macro, you have to remember to **parenthesize** all the arguments in the macro body. Otherwise you can run into trouble when somebody calls the macro with an **expression**.
- You can get all the efficiency of a macro plus all the predictable behavior and type safety of a regular function by using a **template** for an **inline** function.
```c
- template<typename T> // because we don’t know what T is, we pass by reference-to const
inline void callWithMax(const T& a, const T& b) {
    f(a > b ? a : b);
}
```

👍 Things to Remember:
- For simple constants, prefer const objects or enums to `#defines`.
- For function-like macros, prefer inline functions to `#defines`.