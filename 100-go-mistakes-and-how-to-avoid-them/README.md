# 100 Go Mistakes and How to Avoid Them

Notes taken from reading the *100 Go Mistakes and How to Avoid Them* by Teiva Harsanyi.

## Chapter 1: Go: Simple to learn but hard to master.

- Google created the Go programming language in 2007 in response to these challenges:
  - Maximizing **agility** and reducing the time to **market** is critical for most organizations
  - Ensure that software engineers are as **productive** as possible when reading, writing, and maintaining code.
- Why does Go not have feature *X*?
  - Because it **doesn’t fit**, because it affects **compilation** speed or clarity of **design**, or because it would make the fundamental system model too **difficult**.
- Judging the quality of a programming language via its number of features is probably not an accurate metric 💯.
- Go's essential characteristics: **Stability**, **Expressivity**, **Compilation**, **Safety**.
- Go is simple but not easy:
  - Example:  Study shows that popular repos such as Docker, gRPC, and Kubernetes contain bugs are caused by inaccurate use of the message-passing paradigm via channels.
  - Although a concept such as channels and goroutines can be simple to learn, it isn’t an easy topic in practice.
- The cost of software bugs in the U.S. alone to be over $2 trillion 😢.

## Chapter 2: Code and project organization

## #1: Unintended variable shadowing

- In Go, a variable name declared in a block can be redeclared in an inner block.
- **Variable shadowing** occurs when a variable name is redeclared in an inner block, but we saw that this practice is prone to mistakes:
```go
var client *http.Client
if tracing {
    client, err := createClientWithTracing()
    if err != nil {
        return err
    }
    log.Println(client)
} else {
    client, err := createDefaultClient()
    if err != nil {
        return err
    }
    log.Println(client)
}
// Use client
```

## #2: Unnecessary nested code

- A critical aspect of **readability** is the number of **nested** levels.
- In general, the more nested levels a function requires, the more complex it is to read and understand:
    ```go
    func join(s1, s2 string, max int) (string, error) {
        if s1 == "" {
            return "", errors.New("s1 is empty")
        } else {
            if s2 == "" {
                return "", errors.New("s2 is empty")
            } else {
                concat, err := concatenate(s1, s2)
                if err != nil {
                    return "", err
                } else {
                    if len(concat) > max {
                        return concat[:max], nil
                    } else {
                        return concat, nil
                    }
                }
            }
        }
    }
    ```
- *Align the happy path to the left; you should quickly be able to scan down one column to see the expected execution flow*.
- When an `if` block returns, we should omit the `else` block in all cases.
- If we encounter a **non happy-path**, we should flip the condition like so:
    ```go
    if s == "" {
        return errors.New("empty string")
    }
    // ...
    ```

## #3: Misusing init functions

- Consider the example below:
    ```go
    var db *sql.DB
    func init() {
        dataSourceName := os.Getenv("MYSQL_DATA_SOURCE_NAME")
        d, err := sql.Open("mysql", dataSourceName)
        if err != nil {
            log.Panic(err)
        }
        err = d.Ping()
        if err != nil {
            log.Panic(err)
        }
        db = d
    }
    ```
-  Let’s describe three main downsides of the code above:
   - It shouldn’t necessarily be **up to the package** itself to decide whether to stop the application. Perhaps a caller might have preferred implementing a retry or using a fallback mechanism. In this case, opening the database within an `init` function prevents client packages from implementing their error-handling logic.
   - If we add tests to this file, the `init` function will be executed before running the test cases, which isn’t necessarily what we want (for example, if we add unit tests on a utility function that doesn’t require this connection to be created). Therefore, the `init` function in this example **complicates writing unit tests**.
   - The last downside is that the example requires assigning the database connection pool to a **global variable**. Global variables have some severe drawbacks; for example:
    - Any functions can alter global variables within the package.
    - Unit tests can be more complicated because a function that depends on a global variable won’t be **isolated anymore**.
- We should be cautious with `init` functions. They can be helpful in some situations, however, such as defining static configuration. Otherwise, and in most cases, we should handle initializations through ad-hoc functions.

## #4: Overusing getters and setters

- Using **getters** and **setters** presents some advantages, including these:
    - They **encapsulate** a behavior associated with getting or setting a field, allowing new functionality to be added later (for example, validating a field, returning a computed value, or wrapping the access to a field around a mutex).
    - They **hide** the **internal representation**, giving us more flexibility in what we expose.
    - They provide a **debugging** interception point for when the property changes at run time, making debugging easier.
- If we fall into these cases or foresee a possible use case while guaranteeing **forward compatibility**, using getters and setters can bring some value. For example, if we use them with a field called `balance`, we should follow these naming conventions:
    - The getter method should be named `Balance` (not `GetBalance`).
    - The setter method should be named `SetBalance`.
- We shouldn’t **overwhelm** our code with getters and setters on structs if they don’t bring any value. We should be pragmatic and strive to find the right balance between efficiency and following idioms that are sometimes considered indisputable in other programming paradigms.

## #5: Interface pollution

- Interface pollution is about **overwhelming** our code with **unnecessary abstractions**, making it harder to understand.
- What makes Go interfaces so different is that they are satisfied **implicitly**. There is no explicit keyword like *implements* to mark that an object `X` implements interface `Y`.
- While designing interfaces, the **granularity** (how many methods the interface contains) is also something to keep in mind:
  - 🌟 *The bigger the interface, the weaker the abstraction*.
  - Examples: `io.Reader`.
- When to use interfaces ?
  - When multiple types implement a **common** behavior. For example, This `Interface` has a strong potential for reusability because it encompasses the common behavior to **sort** any collection that is index-based:
  ```go
  type Interface interface {
    Len() int
    Less(i, j int) bool
    Swap(i, j int)
    }
  ```
  - **Decoupling** our code from an implementation:
    - **Liskov Substitution Principle** (the L in *Robert C. Martin’s* SOLID design principles).
    - If we rely on an **abstraction** instead of a **concrete** implementation, the implementation itself can be replaced with another without even having to change our code. Example:
    ```go
    type customerStorer interface {
        StoreCustomer(Customer) error
    }
    type CustomerService struct {
        storer customerStorer
    }
    func (cs CustomerService) CreateNewCustomer(id string) error {
        customer := Customer{id: id}
        return cs.storer.StoreCustomer(customer)
    }
    ```
    - This gives us more **flexibility** in how we want to test the method:
      - Use the concrete implementation via integration tests.
      - Use a mock (or any kind of test double) via unit tests.
  - **Restrict** a type to a specific behavior for various reasons, such as semantics enforcement:
    ```go
    type Foo struct {
        threshold intConfigGetter
    }
    func NewFoo(threshold intConfigGetter) Foo {
        return Foo{threshold: threshold}
    }
    func (f Foo) Bar() {
        threshold := f.threshold.Get()
        // ...
    }
    ```
    - The configuration getter is **injected** into the `NewFoo` factory method. It doesn’t impact a client of this function because it can still pass an `IntConfig` struct as
it implements `intConfigGetter`. Then, we can only read the configuration in the `Bar` method, not modify it.
- The main caveat when programming meets abstractions is remembering that abstractions should be **discovered**, not **created**.
  - 🌟 *Don’t design with interfaces, discover them*.

## #6: Interface on the producer side

- **Producer** side — An interface defined in the same package as the concrete implementation.
- **Consumer** side — An interface defined in an external package where it’s used. <p align="center"><img src="./assets/iface-producer-consumer.png" width="500px" height="auto"></p>
- It’s common to see developers creating interfaces on the producer side, alongside the concrete implementation. This design is perhaps a habit from developers having a *C#* or a *Java* background. But in Go, in most cases this is not what we should do 🤨.
- Let’s discuss the following example:
    ```go
    package store
    // CustomerStorage is a good way 🤷 to decouple the client code from the actual implementation.
    // Or, perhaps we can foresee that it will help clients in creating test doubles.
    type CustomerStorage interface {
        StoreCustomer(customer Customer) error
        GetCustomer(id string) (Customer, error)
        UpdateCustomer(customer Customer) error
        GetAllCustomers() ([]Customer, error)
        GetCustomersWithoutContract() ([]Customer, error)
        GetCustomersWithNegativeBalance() ([]Customer, error)
    }
    ```
- This isn’t a best practice in Go. Why ? It’s not up to the producer to **force a given abstraction** for all the clients. Instead, it’s up to the client to decide whether it needs some form of abstraction and then determine the best abstraction level for its needs. For example:
    ```go
    package client

    type customersGetter interface {
        GetAllCustomers() ([]store.Customer, error)
    }
    ```
- The main point is that the `client` package can now define the most **accurate** abstraction for its need (here, only one method). It relates to the concept of the *Interface Segregation Principle* (the `I` in *SOLID*) ▶️ No client should be forced to depend on methods it doesn’t use.
- An interface should live on the **consumer** side in most cases. However, in particular contexts (for example - in the standard library `encoding, encoding/json, encoding/binary`, when we know — not foresee—that an abstraction will be helpful for consumers), we may want to have it on the **producer** side. If we do, we should strive to keep it as **minimal** as possible, increasing its **reusability** potential and making it more easily **composable**.

## #7: Returning interfaces

- We will consider two packages: `client`, which contains a `Store` interface and `store`, which contains an implementation of `Store`. <p align="center"><img src="./assets/store-client-dependency.png" width="500px" height="auto"></p>
- The `client` package can’t call the `NewInMemoryStore` function anymore; otherwise, there would be a **cyclic dependency**.
- In general, returning an interface restricts **flexibility** because we force all the clients to use one particular type of abstraction.
- *Be conservative in what you do, be liberal in what you accept from others.* If we apply this idiom to Go, it means
    - 👍 Returning structs instead of interfaces.
    - 👍 Accepting interfaces if possible.
- We shouldn’t return interfaces but concrete implementations. Otherwise, it can make our design more complex due to package dependencies and can restrict flexibility because all the clients would have to rely on the same
abstraction.
- Also, we will only be able to use the methods **defined** in the interface, and not the methods defined in the **concrete type**.

## #8: any says nothing

- With Go 1.18, the predeclared type `any` became an alias for an **empty interface {}**.
- In assigning a value to an `any` type, we **lose** all **type information**, which requires a type assertion to get anything useful out of the `i` variable.
    ```go
    func main() {
        var i any
        i = 42
        i = "foo"
        i = struct {
            s string
        }{
            s: "bar",
        }
        i = f
        _ = i
    }
    func f() {}
    ```
- In methods, accepting or returning an `any` type doesn’t **convey meaningful information**:
  - Because there is no **safeguard** at compile time, nothing prevents a caller from calling these methods with whatever data type.
  - Also, the methods lack **expressiveness**. If future developers need to use the parameters of type `any`, they will probably have to dig into the documentation or read the code to understand how to use these methods.
- What are the cases when `any` is helpful?:
  - In the `encoding/json` package. Because we can marshal any type, the `Marshal` function accepts an `any` argument.
  - Another example is in the `database/sql` package. If the query is parameterized (for example, `SELECT * FROM FOO WHERE id = ?`), the parameters could be any kind.

## #9: Being confused about when to use generics

- Go 1.18 adds generics to the language 🥳.
-  👍 Few common uses where generics are recommended:
    - **Data structures** : We can use generics to **factor out** the element type if we implement a binary tree, a linked list, or a heap, for example.
    - **Functions working with slices, maps, and channels of any type** : A function to merge two channels would work with any `channel` type, for example
      - Hence, we could use type parameters to factor out the channel type:
        ```go
        func merge[T any](ch1, ch2 <-chan T) <-chan T {
        // ...
        }
        ```
    - **Factoring out behaviors instead of types**:  The `sort` package, for example, contains a `sort.Interface` interface with three methods:
        ```go
        type Interface interface {
            Len() int
            Less(i, j int) bool
            Swap(i, j int)
        }
        ```
      - This interface is used by different functions such as `sort.Ints` or `sort.Float64s`. Using type parameters, we could factor out the sorting behavior(for example, by defining a `struct` holding a slice and a comparison function):
        ```go
        type SliceFn[T any] struct {
            S []T
            Compare func(T, T) bool
        }
        func (s SliceFn[T]) Len() int { return len(s.S) }
        func (s SliceFn[T]) Less(i, j int) bool { return s.Compare(s.S[i], s.S[j]) }
        func (s SliceFn[T]) Swap(i, j int) { s.S[i], s.S[j] = s.S[j], s.S[i] }
        ```
      - Then, because the `SliceFn` struct implements `sort.Interface`, we can sort the provided slice using the `sort.Sort(sort.Interface)` function:
        ```go
        s := SliceFn[int]{
            S: []int{3, 2, 1},
            Compare: func(a, b int) bool {
                return a < b
            },
        }
        sort.Sort(s)
        fmt.Println(s.S)
        ```
- 👎 when is it recommended that we not use generics:
    - When **calling a method of the type argument**: Consider a function that receives an `io.Writer` and calls the `Write` method, for example:
        ```go
        func foo[T io.Writer](w T) {
            b := getBytes()
            _, _ = w.Write(b)
        }
        ```
        - In this case, using generics won’t bring any value to our code whatsoever. We should make the `w` argument an `io.Writer` directly.
    - When it makes our code **more complex**: Generics are never mandatory, and as Go developers, we have lived without them for more than a decade. If we’re writing generic functions or structures and we figure out that it doesn’t make our code clearer, we should probably reconsider our decision for that particular use case.

## #10: Not being aware of the possible problems with type embedding

- 👎 The wolloing example is a wrong usage of type embedding. Since `sync.Mutex` is an embedded type, the `Lock` and `Unlock` methods will be **promoted**. Therefore, both methods become **visible** to external clients using `InMem`:
    ```go
    type InMem struct {
        sync.Mutex
        m map[string]int
    }
- We want to write a custom logger that contains an `io.WriteCloser` and exposes two methods, `Write` and `Close`. If `io.WriteCloser` wasn’t **embedded**, we would need to write it like so:
    ```go
    type Logger struct {
        writeCloser io.WriteCloser
    }
    func (l Logger) Write(p []byte) (int, error) {
        return l.writeCloser.Write(p) // Forwards the call to writeCloser
    }
    func (l Logger) Close() error {
        return l.writeCloser.Close() // Forwards the call to writeCloser
    }
    func main() {
        l := Logger{writeCloser: os.Stdout}
        _, _ = l.Write([]byte("foo"))
        _ = l.Close()
    }
    ```
- 👍 Logger `would` have to provide both a `Write` and a `Close` method that would only **forward** the call to `io.WriteCloser`. However, if the field now becomes **embedded**, we can remove these forwarding methods:
    ```go
    type Logger struct {
        io.WriteCloser
    }
    func main() {
        l := Logger{WriteCloser: os.Stdout}
        _, _ = l.Write([]byte("foo"))
        _ = l.Close()
    }
    ```
- If we decide to use type embedding, we need to keep two main constraints in mind:
    - It shouldn’t be used solely as some **syntactic sugar** to simplify accessing a field (such as `Foo.Baz()` instead of `Foo.Bar.Baz()`). If this is the only rationale, let’s not embed the inner type and use a field instead.
    - It shouldn’t promote data (fields) or a behavior (methods) we want to **hide** from the outside: for example, if it allows clients to access a locking behavior (`sync.Mutex`) that should remain **private** to the struct.