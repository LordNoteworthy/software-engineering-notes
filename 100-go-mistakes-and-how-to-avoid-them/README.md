# 100 Go Mistakes and How to Avoid Them

notes taken from reading the *100 Go Mistakes and How to Avoid Them* by Teiva Harsanyi.

## Chapter 1: Go: Simple to learn but hard to master.

- Google created the Go programming language in 2007 in response to these challenges:
  - maximizing agility and reducing the time to market is critical for most organizations
  - ensure that software engineers are as productive as possible when reading, writing, and maintaining code
- Why does Go not have feature *X*?
  - because it doesn’t fit, because it affects compilation speed or clarity of design, or because it would make the fundamental system model too difficult.
- Judging the quality of a programming language via its number of features is probably not an accurate metric 💯.
- Go's essential characteristics: Stability, Expressivity, Compilation, Safety.
- Go is simple but not easy:
  - Example:  Study shows that popular repos such as Docker, gRPC, and Kubernetes contain bugs are caused by inaccurate use of the message-passing paradigm via channels.
  - Although a concept such as channels and goroutines can be simple to learn, it isn’t an easy topic in practice.
-  The cost of software bugs in the U.S. alone to be over $2 trillion 😢.

## Chapter 2: Code and project organization

## #1: Unintended variable shadowing

- In Go, a variable name declared in a block can be redeclared in an inner block.
- Variable shadowing occurs when a variable name is redeclared in an inner block, but we saw that this practice is prone to mistakes:
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

- A critical aspect of readability is the number of nested levels.
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
- When an if block returns, we should omit the else block in all cases.
- If we encounter a non happy-path, we should flip the condition like so:
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
   -   it shouldn’t necessarily be up to the package itself to decide whether to stop the application. Perhaps a caller might have preferred implementing a retry or using a fallback mechanism. In this case, opening the database within an init function prevents client packages from implementing their error-handling logic.
   - If we add tests to this file, the init function will be executed before running the test cases, which isn’t necessarily what we want (for example, if we add unit tests on a utility function that doesn’t require this connection to be created). Therefore, the init function in this example complicates writing unit tests.
   - The last downside is that the example requires assigning the database connection pool to a global variable. Global variables have some severe drawbacks; for example:
    - Any functions can alter global variables within the package.
    - Unit tests can be more complicated because a function that depends on a global variable won’t be isolated anymore.
- We should be cautious with init functions. They can be helpful in some situations, however, such as defining static configuration. Otherwise, and in most cases, we should handle initializations through ad hoc functions.
