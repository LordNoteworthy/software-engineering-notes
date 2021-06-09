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

### Scalability

- Scalability is the term we use to describe a system’s ability to __adapt to increased load__.

#### Describing load

- Load can be described with a few numbers which we call __load parameters__.  Examples:
    - it’s requests per second;
    - ratio of reads to write;
    - the number of simultaneously active users;
    - number of messages in the queue ...

#### Describing performance

- In a batch-processing system such as _Hadoop_, we usually care about __throughput__: the number of records we can process per second, or the total time it takes to run a job on a dataset of a certain size.
- In online systems, __latency__ is usually more important—the time it takes to serve a request, also known as _response time_.
-  In practice, in a system handling a variety of requests, the latency per request can __vary a lot__. We therefore need to think of latency not as a __single number__, but as a __probability distribution__.
- Even in a scenario where you’d think all requests should take the same time, you get variation: random additional latency could be introduced by:
    - a context switch to a background process;
    - the loss of a network packet and TCP retransmission;
    - a garbage collection pause, a page fault forcing a read from disk, or many other things.
- It’s common to see the __average__ response time of a service reported (arithmetic mean).
- However, the mean is not a very good metric if you want to know your “typical” response time, because it is easily __biased by outliers__.
- Usually it is better to use __percentiles__. If you take your list of response times and sort it, from fastest to slowest, then the median is the half-way point: for example, if your median response time is 200 ms, that means half your requests return in less than 200 ms, and half your requests take longer than that.
- This makes the median a __good metric__ if you want to know how long users typically have to wait. The median is also known as __50th percentile__, and sometimes abbreviated as __p50__.
    <p align="center"><img src="assets/response-time.png" width="500px" height="auto"></p>

#### Approaches for coping with load

- good architectures usually involve a pragmatic mixture of approaches (vertical scaling and horizental scaling).
- there is no such a thing as __magic scaling sauce__. The problem may be:
    - the volume of reads, the volume of writes, the volume of data to store;
    - the complexity of the data, the latency requirements, the access patterns;
    - or (usually) some mixture of all of these plus many more issues.
- In an __early-stage startup__ or an unproven product it’s usually more important to be able to __iterate quickly__ on product features, than it is to scale to some __hypothetical future load__.

### Maintainability

- It is well-known that the majority of the cost of software is not in its initial development, but in its ongoing maintenance.
- We will pay particular attention to three design principles for software systems:
    - __Operability__: Make it easy for operations teams to keep the system running smoothly
    - __Simplicity__: Make it easy for new engineers to understand the system, by removing as much complexity as possible from the system. 
    - __Plasticity__: Make it easy for engineers in future to make changes to the system, adapting it for unanticipated use cases as requirements change.  Also known as extensibility, modifiability or malleability.

#### Operability: making life easy for operations

- _Good operations can often work around the limitations of bad (or incomplete) software, but good software cannot run reliably with bad operations._

### Simplicity: managing complexity

- Moseley and Marks  define complexity as _accidental_ if it is not inherent in the problem that the software solves (as seen by the users), but arises only from the implementation.
- One of the best tools we have for removing accidental complexity is __abstraction__. A good abstraction can hide a great deal of implementation detail behind a clean, simple-to-understand
façade.
    - high-level programming languages are abstractions that hide machine code, CPU registers and syscalls.
    - SQL is an abstraction that hides complex on-disk and in-memory data structures, concurrent requests from other clients, and inconsistencies after crashes.

### Plasticity: making change easy

- In terms of organizational processes, _agile_ working patterns provide a framework for adapting to change. The agile community has also developed technical tools and patterns that are helpful when developing software in a frequently-changing environment, such as __test-driven development (TDD)__ and __refactoring__.

## Chapter 2. The Battle of the Data Models

- Data models are perhaps the most important part of developing software, because they have such a profound effect: not only on how the software is written, but also how we think about the problem that we are solving.
- Most applications are built by layering one data model on top of another, each layer hides the complexity of the layers below it by providing a clean data model.

### Rivals of the Relational Model

- The best-known data model today is probably that of _SQL_, based on the relational model proposed by _Edgar Codd_ in 1970.
- data is organized into __relations__ (in SQL: tables), where each relation is an unordered collection of __tuples__ (rows).