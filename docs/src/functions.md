# Functions

## Named functions (`def`)

```text
def name(param1: Type1, param2: Type2) ReturnType {
    // body
}
```

Parameter types are always required. Omitting a type annotation is a `TypeError`. The return
type can be omitted when it is `Void` or when using `->` syntax (inferred from the expression).

A function with a non-`Void` return type must have a `return` on every execution path.
A missing `return` on any path is a `TypeError`:

```oxynium
def f() Int {
    if true {
        return 1
    } else {
        return 2
    }
    // valid: both paths return
}
```

An `if` without an `else` does not count as covering all paths, even when the condition is
`true` — the compiler performs a structural check, not value analysis.

A `while { }` with no condition is treated as always returning (an infinite loop). A
conditional `while condition { }` does not cover the case where the loop never runs:

```oxynium
def f(a: Bool) Int {
    while a {
        return 1
    }
    return 2    // required
}
```

## Arrow syntax

Single-expression body with implicit return:

```oxynium
def add(a: Int, b: Int) Int -> a + b
def square(n: Int) -> n * n          // return type inferred as Int
def greet(name: Str) -> print("Hello, " + name)   // inferred as Void
```

## Default parameters

Parameters may have default values. Parameters with defaults must come **after** all
parameters without defaults:

```oxynium
def f(a: Int, b: Int = 2, c: Int = 3) {
    print(a.Str() + b.Str() + c.Str())
}
f(1)       // prints 123
f(1, 9)    // prints 193
```

```oxynium
def g(a: Int = 1, b: Int) {}   // TypeError: parameters after `a` must have default values
```

The default value's type must match the parameter's type:

```oxynium
def f(a: Int = "") {}    // TypeError: Str default for Int parameter
```

Trailing commas are allowed in parameter lists, but not in call arguments
(`f(1, 2,)` is a `SyntaxError`).

## Recursive functions

```oxynium
def fib(n: Int) Int {
    if n <= 1 -> return n
    return fib(n - 1) + fib(n - 2)
}
print(fib(10).Str())    // 55
```

## The `main` entry point

`main` is an optional entry point. It must return `Void`.

```oxynium
def main() {
    print("running")
}
```

`main` accepts an optional `List<Utf8Str>` argument for command-line arguments. Any other
parameter type is a `TypeError`. When `main` is defined, no executable statements may appear
at the top level.

## External functions (`extern def`)

Declare a function implemented in another compilation unit (C interop). The compiler emits a
call; the linker must provide the implementation.

```oxynium
extern def my_c_function(x: Int) Str;
```

The trailing `;` is optional. Bodies are forbidden. Cannot be named `main`. Declaring an
external function that isn't linked is fine, but calling it produces an `IoError` when the
program is assembled.

Always give non-`extern` functions a body (even an empty `{}`). A `def` without a body is
currently accepted and treated like an external declaration, so it fails with an `IoError`
only once something calls it.
