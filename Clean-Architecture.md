# Clean Architecture

notes taken from reading the _Clean Architecture_ book by Robert C.Martin.

## Preface

- the rules of software architecture are the same, they are independent of every other variable:
    - programming language
    - hardware
    - domain
    - application type.
    - ...

## Introduction

- getting some code to work is not that rhard, get it right is  another matter entirely.
- when you get a software right:
    - don't need hordes of programmers to keep it working
    - don't need a massive requirements doc and huge issue tracking systems.
    - require a fraction of human resources to maintain, efforts are minimized, functionality and flexibility are maximized.

## 1. What is design and architecture

- the word __architecture__ is often used in the context of something at a _high level_ that is divorced from the low-level details, whereas __design__ more often seems to imply structures and decisions at a _low level_.
- the low-level details and the high-level structure are all part of the same whole, they form a continuous fabric that defines the shape of the system. You can't have one without the other; indeed __no clear deviding__ lines separates them.
- the measure of __design quality__ is simply the measure of the __effort required__ to meet the needs if the customer.
- developers buy into a familir lie:
    - we can __clean it up later__; we just need to get to __market first__ but things never do get cleaned up later because market pressures never abate.
    - making __messy code__ makes them go __fast__ in the short term and just slows them down in the long term.
- a fact is that making messes is _always_ slower then staying clean, no matter which time scale you are using.
- developers may think that the answer is to start over __from scratch__ and redesign the whole system but that could be another mistake due to their __overconfidence__.

## 2. A tale of two values

- every software system provides two different values to the stakeholders:.
    - __behavior__ or __function__: program should work and behave in a way that makes or saves money for the stakeholders.
    - __architecture__: software system should be easy to change.
- both business managers and developers would tend more to advocate function over architecture, but it is the __wrong__ attitude.
- Business managers and developers fail to separate those features that __urgent__ but __not important__, from those features that truly are __urgent and important__. This failure leads to ignoring the important architecture of the system in favor of the unimportant features of the system.
- If __architecture__ comes last, then the system will become ever __more costly__ to develop, and eventually __change__ wil become practically impossible for part or for all of the system. If that is allowed to happen, it means the software development team did not fight hard enough for what they knew was necessery.

## 3. Paradigm overview

- __structured programming__:
    - shows that the use of __unrestrained__ jumps (`goto` statements) is __harmful__ to program structure.
    - replaced those jumps with more familiar `if/then/else` and `do/while/until` constructs.
    - _Structured programming imposes discipline on direct transfer of control_.
- __object orionted programming__:
    - engineers noticed that the function call stack frame in the ALGOL language could be moved to a heap, thereby allowing local variables declared by a function to exist long after the function returned.
    - the function became a constructor for a class, the local variables became instance variables, and the nested functions became methods. This led inevitably to the discovery of polymorphism through the disciplined use of function pointers.
    - _Object-oriented programming imposes discipline on indirect transfer of control_.
- __functional programming__:
    - only recently begun to be adopted, was the first to be invented.
    - a foundational notion of lambda calculus is immutability—that is, the notion that the values of symbols do not change => __no assignment statement__.
    - most functional languages do, in fact, have some means to alter the value of a variable, but only under very strict discipline.
    - _Functional programming imposes discipline upon assignment_.
- each of these paradigms __removes__ capabilities from the programmer, none of them adds new ones. Each imposes some kind of extra discipline that is negative in its intent. The paradigms tell us __what not to do__, more than they tell us __what to do__. They removed `goto` statements, function pointers, and assignement.

## Structured programming