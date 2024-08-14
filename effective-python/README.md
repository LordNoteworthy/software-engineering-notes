# Effective Python

notes taken from reading the Effective Python: 90 Specific Ways To Write Better Python, Second Edition by *Brett Slatkin*.

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
   - 🤷 Why is it `set` when what it’s doing is getting a value? Why not call it `get_or_set`?
   - ⚠️ The default value passed to `setdefault` is assigned **directly** into the dictionary when the key is missing instead of being copied.
- There are only a few circumstances in which using `setdefault` is the shortest way to handle missing dictionary keys, such as when the default values are cheap to construct, mutable, and there’s no potential for raising exceptions (e.g., list instances). Check `defaultdict` in next item 🦊.

📆 Things to Remember
- There are four common ways to detect and handle missing keys in dictionaries: using `in` expressions, `KeyError` exceptions, the `get` method, and the `setdefault` method.
- The `get` method is best for dictionaries that contain **basic** types like **counters**, and it is preferable along with assignment expressions when creating dictionary values has a high cost or may raise
exceptions.
- When the `setdefault` method of dict seems like the best fit for your problem, you should consider using `defaultdict` instead.

### Item 17: Prefer defaultdict Over setdefault to Handle Missing Items in Internal State

- For example, say that I want to keep track of the cities I’ve visited in countries around the world. Here, I do this by using a dictionary that maps country names to a set instance containing corresponding city names:
   ```python
   visits = {
      'Mexico': {'Tulum', 'Puerto Vallarta'},
      'Japan': {'Hakone'},
   }

   visits.setdefault('France', set()).add('Arles') # Short
   ## OR
   if (japan := visits.get('Japan')) is None: # Long
      visits['Japan'] = japan = set()
   japan.add('Kyoto')
   ```
- What about the situation when you do control creation of the dictionary being accessed?:
   ```python
   class Visits:
      def __init__(self):
         self.data = {}
      def add(self, country, city):
         city_set = self.data.setdefault(country, set())
         city_set.add(city)
   ```
      - 👎 The `setdefault` method is still **confusingly named**, which makes it more difficult for a new reader of the code to immediately understand what’s happening.
      - 👎 And the implementation isn’t efficient because it **constructs a new set instance on every call**, regardless of whether the given country was already present in the data dictionary.
- Luckily, the `defaultdict` class from the `collections` built-in module simplifies this common use case by automatically storing a default value when a key doesn’t exist. All you have to do is provide a function that will return the default value to use each time a key is missing . Here, I rewrite the `Visits` class to use `defaultdict`:
   ```python
   from collections import defaultdict

   class Visits:
   def __init__(self):
      self.data = defaultdict(set)
   def add(self, country, city):
      self.data[country].add(city)
   ```
- 👍 The code can assume that accessing any key in the data dictionary will always result in an existing set instance. No superfluous set instances will be allocated every time, which could be costly if the add method is called a large
number of times.

📆 Things to Remember
- If you’re creating a dictionary to manage an arbitrary set of potential keys, then you should prefer using a `defaultdict` instance from the `collections` built-in module if it suits your problem.
- If a dictionary of arbitrary keys is passed to you, and you **don’t control its creation**, then you should prefer the `get` method to access its items. However, it’s worth considering using the `setdefault` method for the few situations in which it leads to shorter code.

### Item 18: Know How to Construct Key-Dependent Default Values with __missing__

- There are times when neither `setdefault` nor `defaultdict` is the right fit:
  - `setdefault`:
    - The open built-in function to create the file handle is **always called**, even when the path is **already present** in the dictionary:
     ```python
     try:
        handle = pictures.setdefault(path, open(path, 'a+b'))
     except OSError:
        print(f'Failed to open path {path}')
        raise
     ```
     - Exceptions may be raised by the `open` call and need to be handled, but it may not be possible to **differentiate** them from **exceptions** that may be raised by the `setdefault` call on the same line
   - `defaultdict` expects that the function passed to its constructor **doesn’t require** any arguments:
      ```python
      def open_picture(profile_path):
         try:
            return open(profile_path, 'a+b')
         except OSError:
            print(f'Failed to open path {profile_path}')
            raise
      pictures = defaultdict(open_picture)
      handle = pictures[path]
      ```
- Solution ? ▶️ You can subclass the `dict` type and implement the `__missing__` special method to add custom logic for handling missing keys:
   ```python
   class Pictures(dict):
      def __missing__(self, key):
         value = open_picture(key)
         self[key] = value
         return value

   pictures = Pictures()
   handle = pictures[path]
   ```

📆 Things to Remember
- The `setdefault` method of `dict` is a bad fit when creating the default value has high computational cost or may raise exceptions.
- The function passed to `defaultdict` must not require any arguments, which makes it impossible to have the default value depend on the key being accessed.
- You can define your own `dict` subclass with a `__missing__` method in order to construct default values that must know which key was being accessed.

## Chapter 3: Functions

### Item 19: Never Unpack More Than Three Variables When Functions Return Multiple Values

- Consider the code below:
   ```python
   def get_stats(numbers):
      minimum = min(numbers)
      maximum = max(numbers)
      count = len(numbers)
      average = sum(numbers) / count

      sorted_numbers = sorted(numbers)
      middle = count // 2
      if count % 2 == 0:
         lower = sorted_numbers[middle - 1]
         upper = sorted_numbers[middle]
         median = (lower + upper) / 2
      else:
         median = sorted_numbers[middle]

      return minimum, maximum, average, median, count

   minimum, maximum, average, median, count = get_stats(lengths)
   ```
- There are two problems with this code.
  - First, all the return values are **numeric**, so it is all too easy to **reorder** them **accidentally** (e.g., swapping `average` and `median`), which can cause bugs that are hard to spot later.
   - Second, the line that calls the function and unpacks the values is **long**, and it likely will need to be wrapped in one of a variety of ways, which hurts **readability**:.

📆 Things to Remember
- You can have functions return multiple values by putting them in a tuple and having the caller take advantage of Python’s **unpacking syntax**.
- Multiple return values from a function can also be unpacked by catch-all **starred expressions**.
- Unpacking into **four** or more variables is **error prone** and should be avoided; instead, return a small class or `namedtuple` instance.

### Item 20: Prefer Raising Exceptions to Returning None

- This misinterpretation of a `False`-equivalent return value is a common mistake in Python code when `None` has special meaning. This is why returning `None` from a function like `careful_divide` is error prone:
   ```python
   def careful_divide(a, b):
      try:
         return a / b
      except ZeroDivisionError:
         return None
   ```
- We should raise exceptions instead and use type annotations:
   ```python
   def careful_divide(a: float, b: float) -> float:
      """Divides a by b.

      Raises:
         ValueError: When the inputs cannot be divided.
      """
      try:
         return a / b
      except ZeroDivisionError as e:
         raise ValueError('Invalid inputs')
   ```

📆 Things to Remember
- Functions that return `None` to indicate special meaning are **error prone** because `None` and other values (e.g., `zero`, the **empty** string) all evaluate to `False` in conditional expressions.
- **Raise exceptions** to indicate special situations instead of returning `None`. Expect the calling code to handle exceptions properly when they’re documented.
- **Type annotations** can be used to make it clear that a function will never return the value `None`, even in special situations.

### Item 21: Know How Closures Interact with Variable Scope

- When you reference a variable in an expression, the Python interpreter traverses the scope to resolve the reference in this order:
  1. The **current function**’s scope.
  2. Any **enclosing** scopes (such as other containing functions).
  3. The scope of the module that contains the code (also called the **global scope**).
  1. The **built-in scope** (that contains functions like `len` and `str`).
- Consider the code below (which suffers from a **scoping bug**):
   ```python
   def sort_priority2(numbers, group):
      found = False # Scope: 'sort_priority2'
      def helper(x):
         if x in group:
            found = True # Scope: 'helper' -- Bad!
            return (0, x)
         return (1,x)
      numbers.sort(key=helper)
      return found
   ```
- Here, I define the same function again, now using `nonlocal`:
   ```python
   def sort_priority3(numbers, group):
      found = False
      def helper(x):
         nonlocal found # Added
         if x in group:
            found = True
            return (0, x)
         return (1, x)
      numbers.sort(key=helper)
      return found
   ```
- When your usage of `nonlocal` starts getting complicated, it’s better to wrap your state in a helper class:
   ```python
   class Sorter:
      def __init__(self, group):
         self.group = group
         self.found = False
      def __call__(self, x):
         if x in self.group:
            self.found = True
            return (0, x)
         return (1, x)

   sorter = Sorter(group)
   numbers.sort(key=sorter)
   assert sorter.found is True
   ```

📆 Things to Remember
- Closure functions can refer to variables from any of the scopes in which they were defined.
- By default, closures can’t affect enclosing scopes by assigning variables.
- Use the `nonlocal` statement to indicate when a closure can modify a variable in its enclosing scopes.
- Avoid using `nonlocal` statements for anything beyond simple functions.

### Item 22: Reduce Visual Noise with Variable Positional Arguments

- Accepting a variable number of **positional** arguments can make a function call **clearer** and reduce **visual noise**.
- These positional arguments are often called *varargs* for short, or *star args*, in reference to the conventional name for the parameter *args.
```python
def log(message, *values): # The only difference
   if not values:
      print(message)
   else:
      values_str = ', '.join(str(x) for x in values)
      print(f'{message}: {values_str}')

log('My numbers are', 1, 2)
log('Hi there') # Much better
```
- If I already have a **sequence** (like a list) and want to call a variadic function like `log`, I can do this by using the `*` operator.
- ⚠️ There are two problems with accepting a variable number of positional arguments:
  - `varargs` are always turned into a **tuple** before they are passed to a function. This means that if the caller uses the * operator on a **generator**, it will be iterated until it’s exhausted. ▶️ Functions that accept `*args` are best for situations where you know the number of inputs in the argument list will be **reasonably small**.
  - You can’t add new positional arguments to a function in the future without **migrating** every caller. ▶️ Use **keyword** only arguments.

📆 Things to Remember
- Functions can accept a variable number of positional arguments by using *args in the def statement.
- You can use the items from a sequence as the positional arguments for a function with the * operator.
- Using the * operator with a generator may cause a program to run out of memory and crash.
- Adding new positional parameters to functions that accept *args can introduce hard-to-detect bugs.

### Item 23: Provide Optional Behavior with Keyword Arguments

- You can mix and match keyword and positional arguments, positional arguments must be specified before keyword arguments.
- Each argument can be specified only once 🤪.
- If you already have a dictionary, and you want to use its contents to call a function like `remainder`, you can do this by using the `** operator`. This instructs Python to pass the values from the dictionary as the corresponding keyword arguments of the function:
   ```python
   def remainder(number, divisor):
      return number % divisor

   my_kwargs = {
      'number': 20,
      'divisor': 7,
   }
   remainder(**my_kwargs)
   ```
- You can mix the `** operator` with positional arguments or keyword arguments in the function call, as long as no argument is repeated: `remainder(number=20, **my_kwargs)`.
- You can also use the `** operator` multiple times if you know that the dictionaries don’t contain **overlapping** keys: `remainder(**my_kwargs, **other_kwargs)`
- And if you’d like for a function to receive any named keyword argument, you can use the `**kwargs` catch-all parameter to collect those arguments into a dict that you can then process.
- The flexibility of keyword arguments provides three significant benefits:
  - 👍 They make the function call **clearer** to new readers of the code.
  - 👍 They can have **default** values specified in the function definition ▶️  eliminates repetitive code and reduces noise.
  - 👍 They provide a powerful way to extend a function’s parameters while remaining **backward compatible** with existing callers.

📆 Things to Remember
- Function arguments can be specified by position or by keyword.
- Keywords make it clear what the purpose of each argument is when it would be confusing with only positional arguments.
- Keyword arguments with default values make it easy to add new behaviors to a function without needing to migrate all existing callers.
- Optional keyword arguments should always be passed by keyword instead of by position.

### Item 24: Use None and Docstrings to Specify Dynamic Default Arguments

- For example, say that I want to load a value encoded as JSON data; if decoding the data fails, I want an empty dictionary to be returned by default:
   ```python
   import json
   def decode(data, default={}):
      try:
         return json.loads(data)
      except ValueError:
         return default
   ```
- The dictionary specified for default will be **shared** by **all calls to decode** because default argument values are evaluated only once (at module load time). This can cause **extremely surprising behavior**:
   ```python
   foo = decode('bad data')
   foo['stuff'] = 5
   bar = decode('also bad')
   bar['meep'] = 1
   print('Foo:', foo)
   print('Bar:', bar)
   >>>
   Foo: {'stuff': 5, 'meep': 1}
   Bar: {'stuff': 5, 'meep': 1}
   ```
- You might expect two different dictionaries, each with a single key and value. But modifying one seems to also modify the other. The culprit is that `foo` and `bar` are both equal to the default parameter. They are the same dictionary object !

📆 Things to Remember
- A default argument value is **evaluated only once**: during function definition at **module load time**. This can cause odd behaviors for dynamic values (like `{}`, `[]`, or `datetime.now()`).
- Use `None` as the default value for any keyword argument that has a dynamic value. Document the actual default behavior in the function’s docstring.
- Using `None` to represent keyword argument default values also works correctly with type annotations.

### Item 25: Enforce Clarity with Keyword-Only and Positional-Only Arguments

- Consider the example below:
   ```python
   def safe_division(number, divisor, ignore_overflow, ignore_zero_division):
      try:
         return number / divisor
      except OverflowError:
         if ignore_overflow:
            return 0
         else:
            raise
      except ZeroDivisionError:
         if ignore_zero_division:
            return float('inf')
         else:
            raise
   ```
- The problem is that it’s easy to **confuse** the **position** of the two *Boolean* arguments that control the exception-ignoring behavior.
- One way to improve the readability of this code is to use keyword arguments:
   ```python
   def safe_division_b(number, divisor,
      ignore_overflow=False, # Changed
      ignore_zero_division=False): # Changed
   ```
- The problem is, since these keyword arguments are **optional** behavior, there’s nothing forcing callers to use keyword arguments for clarity 🤷. With complex functions like this, it’s better to require that callers are clear about their intentions by defining functions with **keyword-only** arguments. These arguments can only be supplied by keyword, **never by position**.
- The `*` symbol in the argument list indicates the end of positional arguments and the beginning of keyword-only arguments:
   ```python
   def safe_division_c(number, divisor, *, # Changed
         ignore_overflow=False,
         ignore_zero_division=False):
   ```
- Callers may specify the first two required arguments (*number* and *divisor*) with a mix of positions and keywords: `safe_division_c(number=2, divisor=5) == 0.4`. Later, I may decide to change the names of these first two arguments because of expanding needs or even just because my style preferences change:
   ```python
   def safe_division_c(numerator, denominator, *, # Changed
         ignore_overflow=False,
         ignore_zero_division=False):
   ```
- Unfortunately, this seemingly superficial change **breaks all the existing callers** that specified the *number* or *divisor* arguments using keywords. Python 3.8 introduces a solution to this problem, called **positional-only** arguments.
- Here, I redefine the `safe_division` function to use positional-only arguments for the **first two required parameters**. The `/` symbol in the argument list indicates where **positional-only arguments end**:
   ```python
   def safe_division_d(numerator, denominator, /, *, # Changed
         ignore_overflow=False,
         ignore_zero_division=False):
   ```
- 🌟 One notable consequence of keyword- and positional-only arguments is that any parameter name **between** the `/` and `*` symbols in the argument list may be passed **either** by **position** or by **keyword**.

📆 Things to Remember
- **Keyword-only** arguments **force** callers to supply certain arguments by keyword (instead of by position), which makes the **intention** of a function call **clearer**. Keyword-only arguments are defined after a single `*` in the argument list.
- **Positional-only** arguments ensure that callers can’t supply certain parameters using keywords, which helps **reduce coupling**. Positional-only arguments are defined before a single `/` in the argument list.
- Parameters between the `/` and `*` characters in the argument list may be supplied by position or keyword, which is the default for Python parameters.

### Item 26: Define Function Decorators with functools.wraps

- Decorators has an unintended side effect. The value returned by the decorator — the function that’s called above — doesn’t
think it’s named `fibonacci`:
```sh
fibonacci = trace(fibonacci) // trace is the decorator function.
print(fibonacci)
>>>
<function trace.<locals>.wrapper at 0x108955dc0>
```
- This behavior is problematic because it undermines tools that do **introspection**, such as **debuggers**.
  - 👎 The *help* built-in function is useless when called on the decorated `fibonacci` function. It should instead print out the docstring.
  - 👎 Object **serializers** break because they can’t determine the **location** of the original function that was decorated.
- 👍 The solution is to use the `wraps` helper function from the `functools` built-in module. This is a decorator that helps you write decorators.
  - When you apply it to the wrapper function, it **copies** all of the important **metadata** about the inner function to the outer function.
  - Beyond these examples, Python functions have many other standard attributes (e.g., `__name__`, `__module__`, `__annotations__`) that must be preserved to maintain the interface of functions in the language.
  - 👍 Using `wraps` ensures that you’ll always get the correct behavior.

📆 Things to Remember
- Decorators in Python are syntax to allow one function to modify another function at runtime.
- Using decorators can cause strange behaviors in tools that do introspection, such as debuggers.
- Use the `wraps` decorator from the `functools` built-in module when you define your own decorators to avoid issues.

## Chapter 4: Comprehensions and Generators

### Item 27: Use Comprehensions Instead of map and filter

- Python provides compact syntax for **deriving a new list** from another *sequence* or *iterable*. These expressions are called list comprehensions:
   ```python
   squares = [x**2 for x in a] # List comprehension`.
   ```
- Unless you’re applying a single-argument function, list comprehensions are also **clearer** than the `map` built-in function for simple cases.
  - `map` requires the creation of a `lambda` function for the computation, which is visually noisy:
   ```python
   alt = map(lambda x: x ** 2, a)
   ```
- Unlike `map`, list comprehensions let you easily **filter** items from the input list, removing corresponding outputs from the result:
   ```python
   even_squares = [x**2 for x in a if x % 2 == 0]
   ```
The `filter` built-in function can be used along with `map` to achieve the same outcome, but it is much harder to read:
   ```python
   alt = map(lambda x: x**2, filter(lambda x: x % 2 == 0, a))
   ```
- Dictionaries and sets have their own equivalents of list comprehensions (called **dictionary comprehensions** and **set comprehensions**,
respectively). These make it easy to create other types of derivative data structures when writing algorithms:
   ```python
   even_squares_dict = {x: x**2 for x in a if x % 2 == 0}
   threes_cubed_set = {x**3 for x in a if x % 3 == 0}
   ```
- Achieving the same outcome is possible with `map` and `filter` if you wrap each call with a corresponding constructor. These statements get so long that you have to break them up across multiple lines, which is even **noisier** and should be **avoided**:
   ```python
   alt_dict = dict(map(lambda x: (x, x**2), filter(lambda x: x % 2 == 0, a)))
   alt_set = set(map(lambda x: x**3, filter(lambda x: x % 3 == 0, a)))
   ```

📆 Things to Remember
- List comprehensions are **clearer** than the `map` and `filter` built-in functions because they **don’t require lambda** expressions.
- List comprehensions allow you to easily skip items from the input list, a behavior that map doesn’t support without help from `filter`.
- Dictionaries and sets may also be created using comprehensions.

### Item 28: Avoid More Than Two Control Subexpressions in Comprehensions

- Here is ane example of two `for` subexpressions. These subexpressions run in the order provided, from left to right:
   ```python
   matrix = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
   flat = [x for row in matrix for x in row]
   print(flat)
   [1, 2, 3, 4, 5, 6, 7, 8, 9]
   ```
- Another example involves replicating the **two-level-deep layout** of the input list:
   ```python
   squared = [[x**2 for x in row] for row in matrix]
   print(squared)
   [[1, 4, 9], [16, 25, 36], [49, 64, 81]]
   ```
- If this comprehension included another loop, it would get so long that I’d have to **split** it over **multiple lines**.
  - At this point, the multiline comprehension isn’t much **shorter** than the alternative
- Multiple **conditions** at the same loop level have an implicit **and** expression.
  - For example, say that I want to filter a list of numbers to only even values greater than 4. These two list comprehensions are equivalent:
      ```python
      a = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
      b = [x for x in a if x > 4 if x % 2 == 0]
      c = [x for x in a if x > 4 and x % 2 == 0]
      ```
- Conditions can be specified at each level of looping after the for subexpression.
  - For example, say I want to filter a matrix so the only cells remaining are those divisible by 3 in rows that sum to 10 or higher.
  ```python
   matrix = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
   filtered = [[x for x in row if x % 3 == 0]
               for row in matrix if sum(row) >= 10]
   print(filtered)
   [[6], [9]]
   ```

📆 Things to Remember
- Comprehensions support multiple levels of loops and multiple conditions per loop level.
- Comprehensions with more than two control sub-expressions are very difficult to read and should be avoided.

### Item 29: Avoid Repeated Work in Comprehensions by Using Assignment Expressions

- Consider the example below:
   ```py
   stock = {
      'nails': 125,
      'screws': 35,
      'wingnuts': 8,
      'washers': 24,
   }
   order = ['screws', 'wingnuts', 'clips']
   def get_batches(count, size):
      return count // size

   result = {}
   for name in order:
      count = stock.get(name, 0)
      batches = get_batches(count, 8)
   ```
- Using an assignment expression as part of the comprehension:
   ```py
   found = {name: batches for name in order
      if (batches := get_batches(stock.get(name, 0), 8))}
   ```
- It’s valid syntax to define an assignment expression in the **value** expression for a comprehension.
  - By moving the assignment expression into the condition and then referencing the variable name it defined in the comprehension’s value expression
   ```py
   result = {name: tenth for name, count in stock.items()
      if (tenth := count // 10) > 0}
   ```
- If a comprehension uses the **walrus** operator in the **value** part of the comprehension and **doesn’t** have a **condition**, it’ll **leak** the loop variable into the containing scope.
- This leakage of the loop variable is similar to what happens with a normal for loop 🤷.
- However, similar leakage doesn’t happen for the **loop variables** from comprehensions.

📆 Things to Remember
- Assignment expressions make it possible for comprehensions and generator expressions to reuse the value from one condition elsewhere in the same comprehension, which can improve readability and performance.
- Although it’s possible to use an assignment expression outside of a comprehension or generator expression’s condition, you should avoid doing so.

### Item 30: Consider Generators Instead of Returning Lists

- When called, a **generator** function does not actually run but instead **immediately returns** an **iterator**.
- With each call to the `next` built-in function, the iterator **advances** the generator to its next `yield` expression.

```python
def index_words_iter(text):
   if text:
      yield 0
   for index, letter in enumerate(text):
      if letter == ' ':
         yield index + 1
```

📆 Things to Remember
- Using generators can be **clearer** than the alternative of having a function return a list of accumulated results.
- The iterator returned by a generator produces the set of values passed to yield expressions within the generator function’s body.
- Generators can produce a sequence of outputs for arbitrarily large inputs because their working **memory** doesn’t include all inputs and outputs.
