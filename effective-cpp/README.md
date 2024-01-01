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

## Item 1: View C++ as a federation of languages.

- Today’s C++ is a multi-paradigm programming language, one supporting a combination of procedural, object-oriented, functional, generic, and meta-programming features.
- Rules for effective C++ programming vary, depending on the part of C++ you are using: C, Object-Oriented C++, Template C++ and The STL.
>> For example, pass-by-value is generally more efficient than pass-by-reference for built-in (i.e., C-like) types, but when you move from the C part of C++ to Object-Oriented C++, the existence of user-defined constructors and destructors means that pass-by-reference-to-const is usually better. This is especially the case when working in Template C++, because there, you don’t even know the type of object you’re dealing with. When you cross into the STL, however, you know that iterators and function objects are modeled on pointers in C, so for iterators and function objects in the STL, the old C pass-by-value rule applies again.