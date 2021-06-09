# Designing Data Intensive Applications

notes taken from reading the _Designing Data Intensive Applications_ book by Martin Kleppmann.


## Chapter 1. Reliable, Scalable and Maintainable Applications

- A data-intensive application is typically built from standard building blocks which provide commonly needed functionality. 
    - Store data so that they, or another application, can find it again later (__databases__),
    - Remember the result of an expensive operation, to speed up reads (__caches__),
    - Allow users to search data by keyword or filter it in various ways (__search indexes__),
    - Send a message to another process, to be handled asynchronously (__message queues__),
    - Observe what is happening, and act on events as they occur (__stream processing__),
    - Periodically crunch a large amount of accumulated data (__batch processing__).

### Thinking About Data Systems

- Even though each category of these systems serves a specific purpose, many new tools for data storage and processing have emerged that are optimized for a variety of different use cases:
    - For example, there are data stores that are also used as __message queues (Redis)__,
    - and there are __message queues with database-like durability guarantees (Kafka)__,
    - => the boundaries between the categories are becoming __blurred__.
- If you are designing a data system or service, a lot of tricky questions arise:
    - How do you ensure that the data remains correct and complete, even when things go wrong internally?
    - How do you provide consistently good performance to clients, even when parts of your system are degraded?
    - How do you scale to handle an increase in load? What does a good API for the service look like?
- We focus on three concerns that are important in most software systems:
    - Reliability
    - Scalability
    - Maintainability

### Reliability

- We understand reliability as meaning, roughly: _continuing to work correctly, even when things go wrong_.
- The things that can go wrong are called __faults__, and systems that anticipate faults and can cope with them are called __fault-tolerant__.
- Note that a fault is not the same as a __failure__ for an overview of the terminology. A __fault__ is usually defined as __one__ component of the system __deviating from its spec__, whereas a __failure__ is when the system as a __whole__ stops providing the required service to the user.

#### Hardware fauls

- Hard disks crash, RAM becomes faulty, the power grid has a blackout, someone unplugs the wrong network cable. Anyone who has worked with __large data centers__ can tell you that these things happen __all the time__ when you have a lot of machines.
- Hard disks are reported as having a __mean time to failure (MTTF)__ of about 10 to 50 years. Thus, on a storage cluster with 10,000 disks, we should expect on average __one disk to die per day__ :smiley:
- There is a move towards systems that can tolerate the __loss of entire machines__, by using software fault-tolerance techniques in preference to hardware redundancy. Such systems also have operational advantages: 
    - A single-server system requires planned downtime if you need to reboot the machine (to apply security patches, for example).
    - Whereas a system that can tolerate machine failure can be patched one node at a time, without downtime of the entire system.

#### Software errors

- Harder to anticipate because they are correlated across nodes, they tend to cause many more system failures than uncorrelated hardware faults. Examples include:
    - A software bug that causes every instance of an application server to crash when given a particular bad
input. For example, consider the leap second on _June 30, 2012_ that caused many applications to hang simultaneously, due to a __bug in the Linux kernel__.
    - A runaway process uses up some shared resource—CPU time, memory, disk space or network bandwidth.
    - A service that the system depends on slows down, becomes unresponsive or starts returning corrupted responses.
    - Cascading failures, where a small fault in one component triggers a fault in another component, which in turn triggers further faults.

### Human errors

- How do we make our system reliable, in spite of unreliable humans?:
    - Design systems in a way that __minimizes opportunities for error__. For example, __well-designed abstractions, APIs and admin interfaces__ make it easy to do “the right thing”, and discourage “the wrong thing”.
    - Provide fully-featured __non-production__ sandbox environments where people can explore and experiment safely, using real data, without affecting real users.
    - Test thoroughly at all levels, from __unit tests__ to whole-system __integration tests__ and __manual tests__.
    - Allow quick and easy recovery from human errors, to minimize the impact in the case of a failure. For example, make it __fast to roll back__ configuration changes, __roll out new code gradually__ (so that any unexpected bugs affect only a small subset of users), and provide tools to recompute data (in case it turns out that the old computation was incorrect).
    - Set up __detailed and clear monitoring and telemetry__, such as performance metrics and error rates.
- How important is reliability? Bugs in business applications cause __lost productivity__ (and legal risks if figures are reported incorrectly), and outages of e-commerce sites can have __huge costs__ in terms of lost revenue and reputation.