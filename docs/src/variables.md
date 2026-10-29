# Variables and Constants

## Global constants (`const`)

Declared at the top level of a file (outside any function). Always require an initial value.
Cannot be mutated.

```oxynium
const MAX = 100
const GREETING = "Hello"
const PI_APPROX = 314
```

The value must be a non-negative integer literal or a string literal, so constants are always
`Int` or `Str`. Expressions (including references to other constants), negative numbers and
other types are a `SyntaxError`:

```oxynium
const BASE = 10
const LIMIT = BASE * 5   // SyntaxError: can only have integers or strings as global constants
```

Attempting to redeclare a constant with the same name is a `TypeError`. Declaring a `const`
inside a function is a `SyntaxError`.

## Local variables (`let`)

Declared inside a function body. Immutable by default — the variable cannot be reassigned
after its initial binding.

```oxynium
def main() {
    let x = 42
    let name = "Alice"
    let greeting = "Hello, " + name
}
```

An optional type annotation must match the inferred type:

```oxynium
def main() {
    let x: Int = 42      // valid
    let y: Int = ""      // TypeError: mismatched types
}
```

Variables cannot be declared without an initial value (use `let mut` for that):

```oxynium
def f() {
    let a;        // SyntaxError: missing value
}
```

Redeclaring the same name in the same function is a `TypeError`. There is no shadowing.
This includes `for` loop variables, so two loops in one function can't both use `i`
(see [Scoping rules](#scoping-rules)).

Declaring a `let` at the top level is a `SyntaxError`.

## Mutable local variables (`let mut`)

Allows reassignment after the initial binding. The type is fixed at declaration.

```oxynium
def main() {
    let mut count = 0
    count = count + 1
    count = 99
}
```

Assigning a different type is a `TypeError`:

```oxynium
def main() {
    let mut a = 1
    a = ""    // TypeError: cannot assign Str to Int
}
```

## Empty `let mut` declarations

`let mut` may be declared with a type annotation but no initial value. The variable must be
assigned before it is read:

```oxynium
def f(condition: Bool) Int {
    let mut result: Int
    if condition {
        result = 1
    } else {
        result = 2
    }
    return result    // valid: both branches assign
}
```

Reading a variable that is never assigned before the read is a `TypeError`. An immutable
`let` cannot use the empty declaration form.

> **Warning:** the compiler does not check every path. If only some branches assign the
> variable (e.g. an `if` without an `else`), the program compiles and reads an undefined
> value on the other paths.

## Compound assignment operators

Shorthand for `variable = variable OP value`. The variable must be `mut`.

| Operator | Example  | Equivalent   |
|----------|----------|--------------|
| `+=`     | `a += 3` | `a = a + 3`  |
| `-=`     | `a -= 3` | `a = a - 3`  |
| `*=`     | `a *= 3` | `a = a * 3`  |
| `/=`     | `a /= 3` | `a = a / 3`  |
| `%=`     | `a %= 3` | `a = a % 3`  |

```oxynium
def main() {
    let mut a = 1
    a += 1    // 2
    a -= 1    // 1
    a *= 4    // 4
    a /= 2    // 2
    print(a.Str())
}
```

`+=` also works on `Str` variables (string concatenation):

```oxynium
def main() {
    let mut s = "hello"
    s += " world"
    print(s)    // hello world
}
```

## Scoping rules

Oxynium does **not** have block-level scoping. A variable declared inside an `if`, `while`,
or `for` block is accessible in the enclosing function after that block. Redeclaring a name
from an outer scope inside an inner block is a `TypeError`:

```oxynium
def main() {
    let a = 1
    if true {
        let a = 2    // TypeError: duplicate declaration
    }
}
```

For-loop variables persist after the loop ends. The item variable holds the last item. The
index variable currently holds the number of iterations, not the last index:

```oxynium
def main() {
    for c, i in "abc" {}
    print(i.Str() + c.Str())    // 3c
}
```

Because loop variables are function-scoped, reusing a loop variable name is a `TypeError`:

```oxynium
def main() {
    for i in range(2) {}
    for i in range(2) {}    // TypeError: symbol `i` is already declared
}
```

## Field mutation through immutable variables

The `let` / `let mut` distinction controls whether the *variable binding* can be reassigned.
Fields on a class instance can still be mutated through a non-`mut` variable:

```oxynium
class Point { x: Int, y: Int }

def main() {
    let p = new Point { x: 0, y: 0 }
    p.x = 5    // valid: mutating a field, not reassigning p
}
```
