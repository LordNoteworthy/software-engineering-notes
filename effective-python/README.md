# Effective Python

notes taken from reading the Effective Python: 90 Specific Ways To Write Better Python, Second Edition by Brett Slatkin.

## Chapter 1: Pythonic Thinking

- Python programmers prefer to be **explicit**, to choose **simple** over **complex**, and to maximize **readability**.
- Type `import this` into your interpreter to read *The Zen of Python*.

### Item 1: Know Which Version of Python You’re Using

- If you’re still stuck working in a Python 2 codebase, you should consider using helpful tools like `2to3` (preinstalled with Python) and `six` (available as a community package) to
help you make the transition to Python 3.

📆 Things to Remember

- **Python 3** is the most up-to-date and well-supported version of Python, and you should use it for your projects.
- Be sure that the command-line executable for running Python on your system is the version you expect it to be.
- Avoid **Python 2** because it will no longer be maintained after *January 1, 2020*.

### Item 2: Follow the PEP 8 Style Guide

Python Enhancement Proposal #8, otherwise known as **PEP 8**, is the style guide for how to **format** Python code.

<details><summary>⭐ Rules for Whitespace:</summary>

- Use **spaces** instead of **tabs** for indentation.
- Use **four spaces** for each level of syntactically significant indenting.
- Lines should be **79** characters in length or less.
- Continuations of **long expressions** onto additional lines should be indented by **four** extra spaces from their normal indentation level.
- In a file, functions and classes should be separated by **two blank lines**.
- In a class, methods should be separated by **one blank line**.
- In a dictionary, put **no whitespace** between each **key** and **colon**, and put a **single space** before the corresponding value if it fits on the same line.
- Put one and only **one—space** before and after the `=` operator in a variable assignment.
- For type **annotations**, ensure that there is **no separation** between the variable name and the colon, and use a **space** before the type information.
</details>

<details><summary>⭐ Rules for Naming:</summary>

- Functions, variables, and attributes should be in `lowercase_underscore` format.
- Protected instance attributes should be in `_leading_underscore` format.
- Private instance attributes should be in `__double_leading_underscore` format.
- Classes (including exceptions) should be in `CapitalizedWord` format.
- Module-level constants should be in `ALL_CAPS` format.
- Instance methods in classes should use self, which refers to the object, as the name of the first parameter.
- Class methods should use `cls`, which refers to the class, as the name of the first parameter.
</details>

<details><summary>⭐ Expressions and Statements:</summary>

- Use inline negation `(if a is not b)` instead of negation of positive expressions `(if not a is b)`.
- Don’t check for **empty** containers or sequences (like [] or '') by comparing the length to zero (`if len(somelist) == 0`). Use `if not somelist` and assume that empty values will implicitly evaluate to `False`.
- The same thing goes for non-empty containers or sequences (like [1] or 'hi'). The statement if `somelist` is implicitly `True` for nonempty values.
- Avoid **single-line** `if` statements, `for` and `while` loops, and `except` compound statements. Spread these over multiple lines for clarity.
- If you can’t fit an expression on one line, surround it with parentheses and add line breaks and indentation to make it easier to read.
- Prefer surrounding multiline expressions with parentheses over using the \ line continuation character.
</details>

<details><summary>⭐ Imports:</summary>

- Always put import statements (including `from x import y`) at the **top** of a file.
- Always use **absolute** names for modules when importing them, not names **relative** to the current module’s own path. For example, to import the foo module from within the bar package, you should use f`rom bar import foo`, not just `import foo`.
- If you must do relative imports, use the **explicit** syntax `from . import foo`.
- Imports should be in sections in the following **order**: standard library modules, third-party modules, your own modules. Each subsection should have imports in **alphabetical** order.
</details>

📆 Things to Remember

- Always follow the *Python Enhancement Proposal* #8 (PEP 8) style guide when writing Python code.
- Sharing a **common** style with the larger Python community facilitates collaboration with others.
- Using a **consistent** style makes it easier to modify your own code later.

### Item 3: Know the Differences Between bytes and str

- `str` instances do not have an associated binary encoding, and bytes instances do not have an associated text encoding.
- To convert Unicode data ▶️ binary data, you must call the `encode` method of `str`.
- To convert binary data ▶️ Unicode data, you must call the `decode` method of `bytes`.
- You can explicitly specify the encoding you want to use for these methods, or accept the system default, which is commonly *UTF-8*.
- When you’re writing Python programs, it’s important to do encoding and decoding of Unicode data at the furthest boundary of your interfaces; this approach is often called the **Unicode sandwich**.

📆 Things to Remember

- `bytes` contains sequences of 8-bit values, and `str` contains sequences of *Unicode* code points.
- Use helper functions to ensure that the inputs you operate on are the type of character sequence that you expect (8-bit values, UTF-8-encoded strings, Unicode code points, etc).
- `bytes` and `str` instances can’t be used together with operators (like >, ==, +, and %).
- If you want to read or write **binary** data to/from a file, always open the file using a **binary** mode (like 'rb' or 'wb').
- If you want to read or write **Unicode** data to/from a file, be careful about your system’s **default text encoding**. Explicitly pass the encoding parameter to open if you want to avoid surprises.

### Item 4: Prefer Interpolated F-Strings Over C-style Format Strings and str.format

- Python has four different ways of formatting strings that are built into the language and standard library:
  1. **% formatting operator**: The syntax for format specifiers comes from C’s `printf` function, which has been inherited by Python.
     - 👎 **Problem #1**: You need to **constantly** check that the two sides of the % operator are in **sync**.
     - 👎 **Problem #2**: They become difficult to read when you need to make small **modifications** to values **before formatting** them into a string.
     - 👎 **Problem #3** :if you want to use the same value in a format string **multiple** times, you have to repeat it in the right side tuple.
     - 🤷 Using dictionaries in formatting expressions solves **problem #1** and **problem #3** but exacerbate **problem #2**.
     - 👎 Using dictionaries in formatting expressions also increases verbosity, which is **problem #4** with C-style formatting expressions in Python.
  2. `format` Built-in and `str.format`:
     - Python 3 introduces a new formatting option: `formatted = format(a, ',.2f')` or you can use placeholders with `{}` (**positional** arguments by default).
     - Like the C-style formatting which required doubling the `%` character to **escape** it, `str.format` needs braces to be escaped.
     - 👎 Does nothing to address **problem #2** from above.
     - 👎 Don’t help reduce the redundancy of repeated keys from **problem #4** above.
  3. **Interpolated Format Strings**:
     - Python 3.6 added interpolated format strings or **f-strings** for short.
     - This new language syntax requires you to prefix format strings with an `f` character.
     - 👍 All of the same options from the new format built-in mini language are available after the colon in the placeholders within an f-string, as is the ability to coerce values to *Unicode* and *repr* strings similar to the `str.format`.
     - 👍 Enable you to put a full Python expression within the placeholder braces, solving **problem #2**.

📆 Things to Remember

- **C-style** format strings that use the % operator suffer from a variety of gotchas and verbosity problems.
- The `str.format` method introduces some useful concepts in its formatting specifiers mini language, but it otherwise repeats the mistakes of C-style format strings and should be avoided.
- **F-strings** are a new syntax for formatting values into strings that solves the biggest problems with C-style format strings.
- F-strings are succinct yet powerful because they allow for **arbitrary Python expressions** to be directly embedded within format specifiers.

### Item 5: Write Helper Functions Instead of Complex Expressions

- What you gain in **readability** always outweighs what **brevity** may have afforded you.

📆 Things to Remember

- Python’s syntax makes it easy to write single-line expressions that are overly complicated and difficult to read.
- Move complex expressions into helper functions, especially if you need to use the same logic **repeatedly**.
- An `if/else` expression provides a more readable alternative to using the `Boolean` operators `or` and `and` in expressions.
