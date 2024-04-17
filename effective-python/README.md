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