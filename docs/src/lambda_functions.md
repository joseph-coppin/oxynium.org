# Anonymous Functions

Anonymous (lambda) functions are first-class values. They can be stored in variables,
passed as arguments, and returned from functions.

## Syntax

```oxynium
def main() {
    let double = fn (x: Int) Int { return x * 2 }
    let triple = fn (x: Int) Int -> x * 3
    let add    = fn (a: Int, b: Int) -> a + b
    let greet  = fn () -> print("hi")
}
```

The return type can be inferred with `->`. An explicit return type is written before `->`:

```oxynium
def main() {
    let f = fn () Int -> 1     // explicit return type
    let g = fn () -> 1         // inferred return type
}
```

Anonymous functions **cannot be immediately invoked**:

```oxynium
(fn () { print("hi") })()    // SyntaxError
```

## Closure restriction

Anonymous functions can only access global constants and top-level definitions. They cannot
close over local variables or parameters of the enclosing function:

```oxynium
def main() {
    let x = 5
    let f = fn () Int { return x }    // UnknownSymbol: x is not accessible
}
```

Global constants are accessible:

```oxynium
const MESSAGE = "hi"
def main() {
    let a = fn () -> print(MESSAGE)
    a()    // prints hi
}
```

Anonymous functions also cannot reference sibling local `fn` values (one local `fn` cannot
call another declared in the same function).

## Higher-order functions

Passing anonymous functions to functions that accept function parameters:

```oxynium
def apply(f: Fn(Int) Int, x: Int) Int -> f(x)

def main() {
    print(apply(fn (n: Int) Int -> n * n, 5).Str())    // 25
}
```

```oxynium
def do_something(f: Fn(Int) Int) Int {
    return f(42)
}

def main() {
    let plus_one = fn (x: Int) Int { return x + 1 }
    print(do_something(plus_one).Str())    // 43
}
```

An anonymous function passed with a mismatched signature is a `TypeError`:

```oxynium
def apply(f: Fn(Int) Int) {}
apply(fn (x: Int) -> "")    // TypeError: Str returned, Int expected
```

## Named functions as first-class values

Named functions can also be stored in variables and reassigned:

```oxynium
def inc(x: Int) Int -> x + 1
def add_two(x: Int) Int -> x + 2

def main() {
    let mut f = inc
    print(f(1).Str())    // 2
    f = add_two
    print(f(1).Str())    // 3
}
```
