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

### Item 6: Prefer Multiple Assignment Unpacking Over Indexing

- Unpacking works when assigning to lists, sequences, and multiple levels of **arbitrary iterables** within iterables.
   ```python
   favorite_snacks = {
      'salty': ('pretzels', 100),
      'sweet': ('cookies', 180),
      'veggie': ('carrots', 20),
   }

   ((type1, (name1, cals1)),
   (type2, (name2, cals2)),
   (type3, (name3, cals3))) = favorite_snacks.items
   ```
- Unpacking can even be used to **swap** values in place **without** the need to create **temporary** variables: `a[i-1], a[i] = a[i], a[i-1]`.
- Another valuable application of unpacking is in the target list of `for` loops and similar constructs, such as comprehensions and generator expressions:
   ```python
   for rank, (name, calories) in enumerate(snacks, 1):
      print(f'#{rank}: {name} has {calories} calories')
   ```

📆 Things to Remember
- Python has special syntax called **unpacking** for assigning multiple values in a single statement.
- Unpacking is generalized in Python and can be applied to any iterable, including many levels of iterables within iterables.
- 👍 Reduce visual noise and increase code clarity by using unpacking to avoid explicitly **indexing** into sequences.

### Item 7: Prefer enumerate Over range

- Often, you’ll want to iterate over a list and also know the index of the current item in the list:
   ```python
      for i in range(len(flavor_list)):
         flavor = flavor_list[i]
         print(f'{i + 1}: {flavor}')
   ```
- `enumerate` wraps any iterator with a **lazy generator**.  `enumerate` yields **pairs** of the loop **index** and the **next value** from the given iterator.
- Each pair yielded by enumerate can be succinctly **unpacked** in a for statement. The resulting code is much clearer:
   ```python
   for i, flavor in enumerate(flavor_list):
      print(f'{i + 1}: {flavor}')
   ```

📆 Things to Remember
- `enumerate` provides concise syntax for looping over an iterator and getting the index of each item from the iterator as you go.
- Prefer `enumerate` instead of looping over a **range** and **indexing** into a sequence.
- You can supply a second **parameter** to enumerate to specify the number from which to begin counting (zero is the default).

### Item 8: Use zip to Process Iterators in Parallel

- Python provides the zip built-in function. `zip` wraps two or more iterators with a lazy generator. The zip generator **yields tuples** containing the next value from each iterator:
   ```python
   for name, count in zip(names, counts):
      if count > max_count:
         longest_name = name
         max_count = count
   ```
- If you don’t expect the lengths of the lists passed to zip to be **equal**, consider using the `zip_longest` function from the `itertools` built-in module instead:
   ```python
   for name, count in itertools.zip_longest(names, counts):
      print(f'{name}: {count}')
   ```

📆 Things to Remember
- The `zip` built-in function can be used to iterate over **multiple iterators** in parallel.
- `zip` creates a lazy generator that produces tuples, so it can be used on infinitely **long** inputs.
- `zip` truncates its output silently to the **shortest** iterator if you supply it with iterators of different lengths.
- Use the `zip_longest` function from the `itertools` built-in module if you want to use `zip` on iterators of unequal lengths without truncation.

### Item 9: Avoid else Blocks After for and while Loops

- Given all the uses of `else`, `except`, and `finally` in Python, a new programmer might assume that the else part of `for/else `means “*Do this if the loop wasn’t completed*.” In reality, it does exactly the opposite.
- Using a `break` statement in a loop actually **skips** the `else` block:
   ```python
   for i in range(3):
      print('Loop', i)
      if i == 1:
         break
      else:
      print('Else block!')
   ```
- Another surprise is that the `else` block runs immediately if you loop over an **empty** sequence !
- The `else` block also runs when `while` loops are initially `False`.
- ▶️ The rationale for these behaviors is that `else` blocks after loops are useful when using loops to **search for something**.
- 🤷 The **expressivity** you gain from the `else` block doesn’t **outweigh** the **burden** you put on people (including yourself) who want to understand your code in the future. Simple constructs like loops should be self-evident in Python.

📆 Things to Remember
- Python has special syntax that allows `else` blocks to immediately follow `for` and `while` loop interior blocks.
- The `else` block after a **loop** runs only if the loop body did not encounter a **break** statement.
- **Avoid** using `else` blocks after loops because their behavior isn’t intuitive and can be confusing.

### Item 10: Prevent Repetition with Assignment Expressions

- Normal assignment statements are written **a = b** and pronounced `a equals b,` these assignments are written `a := b` and pronounced **a walrus b**.
- This pattern of **fetching** a value, checking to see if it’s **non-zero**, and then **using** it is extremely common in Python.
   ```python
   if count := fresh_fruit.get('lemon', 0): // assign and then evaluate
      make_lemonade(count)
   else:
      out_of_stock()
   ```
- In the below example: tt's a lot more **readable** because it’s now clear that `count` is only relevant to the first block of the `if` statement.
   ```python
   if (count := fresh_fruit.get('apple', 0)) >= 4:
      make_cider(count)
   else:
      out_of_stock()
   ```
- It also provides an elegant solution that can feel nearly as versatile as dedicated syntax for `switch/case` statements:
   ```python
   if (count := fresh_fruit.get('banana', 0)) >= 2:
      pieces = slice_bananas(count)
      to_enjoy = make_smoothies(pieces)
   elif (count := fresh_fruit.get('apple', 0)) >= 4:
      to_enjoy = make_cider(count)
   else:
      to_enjoy = 'Nothing'
   ```
- The walrus operator obviates the need for the *loop-and-a-half* idiom by allowing the `fresh_frui`t variable to be reassigned and then conditionally evaluated each time through the `while` loop:
   ```python
   bottles = []
   while fresh_fruit := pick_fruit():
   for fruit, count in fresh_fruit.items():
      batch = make_juice(fruit, count)
      bottles.extend(batch)
   ```

📆 Things to Remember
- Assignment expressions use the **walrus** operator (`:=`) to both assign and evaluate variable names in a single expression, thus reducing **repetition**.
- When an assignment expression is a subexpression of a larger expression, it must be surrounded with **parentheses**.
- Although `switch/case` statements and `do/while` loops are not available in Python, their functionality can be emulated much more clearly by using assignment expressions.

## Chapter 2: Lists and Dictionaries

- Dictionaries (often called an *associative array* or a *hash table*) provide **constant time** performance for **assignments** and **accesses**, which means they are ideal for bookkeeping dynamic information.

### Item 11: Know How to Slice Sequences

- Slicing can be extended to any Python class that implements the `__getitem_`_` and `__setitem__` special methods.
- The result of slicing a list is a whole new list.
- ⚠️ If you assign to a slice with **no start or end indexes**, you replace the entire contents of the list with a copy of what’s **referenced** (instead of allocating a new list).

📆 Things to Remember
- Avoid being **verbose** when slicing: Don’t supply 0 for the start index or the length of the sequence for the end index.
- Slicing is **forgiving of start or end** indexes that are out of bounds which means it’s easy to express slices on the front or back boundaries of a sequence (like a[:20] or a[-20:]).
- Assigning to a list slice **replaces that range** in the original sequence with what’s referenced even if the lengths are different.

### Item 12: Avoid Striding and Slicing in a Single Expression

- In addition to basic slicing, Python has special syntax for the **stride** of a slice in the form `somelist[start : end : stride]`. This lets you take every `nth` item when slicing a sequence.
-  For example, the stride makes it easy to group by even and odd indexes in a list:
   ```python
   x = ['red', 'orange', 'yellow', 'green', 'blue', 'purple']
   odds = x[::2]
   evens = x[1::2]
   print(odds)
   print(evens)
   >>>
   ['red', 'yellow', 'blue']
   ['orange', 'green', 'purple']
   ```
- A common Python trick for **reversing** a byte string is to slice the string with a stride of `-1` (this also works correctly for **Unicode** strings):
   ```python
   x = b'mongoose'
   y = x[::-1]
   ```
- ⚠️ But it will break when Unicode data is encoded as a **UTF-8** byte string !

📆 Things to Remember
- Specifying start, end, and stride in a slice can be extremely **confusing**.
- Prefer using **positive** stride values in slices **without start** or **end** indexes. **Avoid negative** stride values if possible.
- Avoid using start, end, and stride **together** in a **single** slice. If you need all three parameters, consider doing two assignments (one to stride and another to slice) or using `islice` from the `itertools` built-in module.

### Item 13: Prefer Catch-All Unpacking Over Slicing

- Consider the example below:
   ```python
   oldest = car_ages_descending[0]
   second_oldest = car_ages_descending[1]
   others = car_ages_descending[2:]
   print(oldest, second_oldest, others)
   >>>
   20 19 [15, 9, 8, 7, 6, 4, 1, 0]
   ```
   - 👎 Indexing and slicing is **visually noisy**, and also error prone because of **off-by-one** errors.
- To better handle this situation, Python supports **catch-all unpacking** through a *starred expression*:
   ```python
   oldest, second_oldest, *others = car_ages_descending
   ```
- A starred expression may appear in **any position**, so you can get the benefits of catch-all unpacking anytime you need to extract one slice:
  ```python
   oldest, *others, youngest = car_ages_descending
   print(oldest, youngest, others)
   *others, second_youngest, youngest = car_ages_descending
   print(youngest, second_youngest, others)
   >>>
   20 0 [19, 15, 9, 8, 7, 6, 4, 1]
   0 1 [20, 19, 15, 9, 8, 7, 6, 4]
   ```
- You also can’t use **multiple catch-all** expressions in a **single-level** unpacking pattern: `first, *middle, *second_middle, last = [1, 2, 3, 4]` but:
  - It is possible to use multiple starred expressions in an unpacking assignment statement, as long as they’re catch-alls for **different parts** of the multilevel structure being unpacked:
   ```python
   ((loc1, (best1, *rest1)),
   (loc2, (best2, *rest2))) = car_inventory.items()
   ```
- Starred expressions become **list** instances in all cases. If there are **no leftover** items from the sequence being unpacked, the catch-all part will be an **empty list**.
- You can also unpack arbitrary **iterators** with the unpacking syntax:
   ```python
   it = iter(range(1, 3))
   first, second = it
   ```
   - Unpacking with a starred expression makes it easy to process the **first row** — the header — separately from the rest of the iterator’s contents. This is much clearer:
   ```python
   it = generate_csv()
   header, *rows = it
   print('CSV Header:', header)
   print('Row count: ', len(rows))
   >>>
   CSV Header: ('Date', 'Make', 'Model', 'Year', 'Price')
   Row count: 200
   ```

📆 Things to Remember
- **Unpacking assignments** may use a **starred** expression to catch all values that weren’t assigned to the other parts of the unpacking pattern into a list.
- **Starred expressions** may appear in any position, and they will always become a list containing the zero or more values they receive.
- When dividing a list into non-overlapping pieces, **catch-all unpacking** is much **less error prone** than **slicing and indexing**.

### Item 14: Sort by Complex Criteria Using the key Parameter

- Use the **lambda** keyword to define a function for the `key` parameter that enables me to sort the list of `Tool` objects alphabetically by their name:
   ```python
   print('Unsorted:', repr(tools))
   tools.sort(key=lambda x: x.name)
   print('\nSorted: ', tools)
   ```
- Sometimes you may need to use **multiple criteria** for sorting. For example, say that I have a list of power tools and I want to sort them first by weight and then by name.
  - 👍 The simplest solution is to use the *tuple* type.
    - Tuples are **comparable** by default and have a **natural ordering**, meaning that they implement all of the special methods, such as `__lt__`, that are required by the `sort` method.
        ```python
        saw = (5, 'circular saw')
        jackhammer = (40, 'jackhammer')
        assert not (jackhammer < saw) # Matches expectations
        ```
   - Another solution would be to define a key function that returns a tuple containing the two attributes that I want to sort on in order of priority: `power_tools.sort(key=lambda x: (x.weight, x.name))`.
     - One limitation of this method is that the direction of sorting for all criteria **must be the same* (either all in ascending order, or all in descending order) 🤷.
     - For **numerical** values, it’s possible to mix sorting directions by using the *unary minus* operator in the key function: `power_tools.sort(key=lambda x: (-x.weight, x.name))`.
     - For **non numerical** types, call sort **multiple times** on the same list to combine different criteria together. Here, I produce the same sort ordering of weight descending and name ascending as I did above but by using two separate calls to sort:
      ```python
      power_tools.sort(key=lambda x: x.name) # Name ascending
      power_tools.sort(key=lambda x: x.weight, reverse=True) # Weight descending
      ```
      - ⚠️ Make sure that you execute the sorts in the **opposite sequence** of what you want the final list to contain.

📆 Things to Remember
- The `sort` method of the list type can be used to rearrange a list’s contents by the natural ordering of built-in types like *strings*, *integers*, *tuples*, and so on.
- The `sort` method doesn’t work for **objects** unless they **define** a natural ordering using **special methods**, which is uncommon.
- The `key` parameter of the `sort` method can be used to supply a helper function that returns the value to use for sorting in place of each item from the list.
- **Returning a tuple** from the key function allows you to combine multiple sorting criteria together. The *unary minus* operator can be used to **reverse** individual sort orders for types that allow it.
- For types that can’t be negated, you can combine many sorting criteria together by calling the `sort` method **multiple times** using different key functions and reverse values, in the order of lowest rank sort call to highest rank sort call.

### Item 15: Be Cautious When Relying on dict Insertion Ordering

- Python 3.5 and before, iterating over a dict would return keys in arbitrary order.
  - Why ? The dictionary type implemented its hash table algorithm with a combination of the hash built-in function and a **random seed** that was assigned when the Python **interpreter started**.
- The way that dictionaries preserve insertion ordering is now part of the Python (3.7) language specification.
  - `.keys(), .values(), .items(), .popitem()`.
  - The order of **keyword arguments**: `my_func(**kwargs)`.
  - **Object fields** in classes also use the dict type for their instance dictionaries:
      ```python
      a = MyClass()
      for key, value in a.__dict__.items():
         print('%s = %s' % (key, value))
      ```
- ⭐ For a long time the `collections` built-in module has had an `OrderedDict`
class that preserves insertion ordering.
   - Although this class’s behavior is similar to that of the standard dict type (since Python 3.7), the **performance** characteristics of `OrderedDict` are quite different.
   -  If you need to handle a **high rate of key insertions** and `popitem` calls (e.g., to implement a least-recently-used cache), `OrderedDict` may be a better fit !
- ⚠️ Example of a class that conforms to the protocol of a standard dictionary but won't preserve the order:
   ```python
   from collections.abc import MutableMapping

   class SortedDict(MutableMapping):
      def __init__(self): self.data = {}
      def __getitem__(self, key): return self.data[key]
      def __setitem__(self, key, value): self.data[key] = value
      def __delitem__(self, key): del self.data[key]
      def __len__(self): return len(self.data)
      def __iter__(self):
         keys = list(self.data.keys())
         keys.sort()
         for key in keys:
            yield key
   ```

📆 Things to Remember
- Since Python 3.7, you can rely on the fact that **iterating a dict** instance’s contents will occur in the **same order** in which the keys were initially added.
- Python makes it easy to define objects that act like dictionaries but that aren’t dict instances. For these types, you **can’t assume** that insertion ordering will be **preserved**.
- There are three ways to be careful about dictionary-like classes:
  - Write code that doesn’t rely on insertion ordering 🤷.
  - Explicitly **check** for the dict type at **runtime**.
  - Require dict values using **type annotations** and **static analysis**.

### Item 16: Prefer get Over in and KeyError to Handle Missing Dictionary Keys

- Using `in` requires accessing the key two times:
   ```python
   key = 'wheat'
   if key in counters:
      count = counters[key]
   else:
      count = 0
   counters[key] = count + 1
   ```
- Using the exceptions' approach is more efficient because it requires only one access and one assignment:
   ```python
   try:
      count = counters[key]
   except KeyError:
      count = 0
   counters[key] = count + 1
   ```
- This flow of fetching a key that exists or returning a default value is so common that the dict built-in type provides the `get` method to accomplish this task:
   ```python
   count = counters.get(key, 0)
   counters[key] = count + 1
   ```
- `setdefault` is superfluous for cases like this because it requires one access and two assignments:
   ```python
   count = counters.setdefault(key, 0)
   counters[key] = count + 1
   ```
- What if the values of the dictionary are a more **complex** type, like a `list`?:
   ```python
   votes = {
      'baguette': ['Bob', 'Alice'],
      'ciabatta': ['Coco', 'Deb'],
   }
   key = 'brioche'
   who = 'Elmer'
   if key in votes:
      names = votes[key]
   else:
      votes[key] = names = [] # the list is modified by reference.
   names.append(who)
   ```
- Relying on the `in` expression requires two accesses if the key is present, or one access and one assignment if the key is missing.
-  Using the exceptions' approach requires one key access if the key is present, or one key access and one assignment if it’s missing, which makes it more efficient than the in condition:
   ```python
   try:
      names = votes[key]
   except KeyError:
      votes[key] = names = []
   ```
- Same thing for the `get` method:
   ```python
   if (names := votes.get(key)) is None:
      votes[key] = names = []
   names.append(who)
   ```
- The dict type also provides the `setdefault` method to help shorten this pattern even further:
   ```python
   names = votes.setdefault(key, [])
   names.append(who)
   ```
   - 👎 Readability of this approach isn’t ideal.
   - Why is it `set` when what it’s doing is getting a value? Why not call it `get_or_set`? 🤷.
   - ⚠️ The default value passed to `setdefault` is assigned **directly** into the dictionary when the key is missing instead of being copied.
- There are only a few circumstances in which using `setdefault` is the shortest way to handle missing dictionary keys, such as when the default values are cheap to construct, mutable, and there’s no potential for raising exceptions (e.g., list instances). Check `defaultdict` in next item 🦊.

📆 Things to Remember
- There are four common ways to detect and handle missing keys in dictionaries: using `in` expressions, `KeyError` exceptions, the `get` method, and the `setdefault` method.
- The `get` method is best for dictionaries that contain **basic** types like **counters**, and it is preferable along with assignment expressions when creating dictionary values has a high cost or may raise
exceptions.
- When the `setdefault` method of dict seems like the best fit for your problem, you should consider using `defaultdict` instead.