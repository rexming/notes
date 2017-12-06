1. A deferred function's arguments are evaluated when the defer statement is evaluated.

In this example, the expression "i" is evaluated when the Println call is deferred. The deferred call will print "0" after the function returns.
```
    func a() {
    i := 0
    defer fmt.Println(i)
    i++
    return
}
```
2. Deferred function calls are executed in Last In First Out order after the surrounding function returns.

This function prints "3210":