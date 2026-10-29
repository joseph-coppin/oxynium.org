# Control Flow

## `if` statements

```oxynium
def f(condition: Bool, other_condition: Bool) {
    if condition {
        // body
    }

    if condition {
        // then
    } else {
        // else
    }

    if condition {
        // branch 1
    } else if other_condition {
        // branch 2
    } else {
        // branch 3
    }
}
```

The condition must have type `Bool`. Using an `Int`, `Str`, or class instance as a condition
is a `TypeError`:

```oxynium
if 0 { }      // TypeError: if condition must be of type Bool
```

Parentheses around the condition are not needed, but are allowed, since `(condition)` is
just a parenthesised expression.

### Arrow syntax for single statements

`->` replaces `{ }` when the body is a single statement:

```oxynium
def main() {
    let x = 5
    if x > 0 -> print("positive")
    if x > 0 -> print("positive") else print("not positive")
}
```

## `while` loops

**Conditional while** — runs while the condition is `Bool` true:

```oxynium
def main() {
    let mut i = 0
    while i < 5 {
        print(i.Str(), "")
        i += 1
    }
    // prints 01234
}
```

**Infinite loop** — no condition; must use `break` to exit:

```oxynium
while {
    print("once")
    break
}
```

**Arrow form** — single-statement body:

```oxynium
def main() {
    let mut i = 0
    while i == 0 -> i = 1
}
```

The condition (when present) must be `Bool`. `while 1 {}` is a `TypeError`.

## `break`, `continue`, and `return`

`break` exits the innermost loop. `continue` skips to the next iteration. Both are valid
inside `while` and `for` loops.

```oxynium
def main() {
    let mut i = 0
    while {
        i += 1
        if i < 5 -> continue
        print(i.Str())    // prints 5
        break
    }
}
```

`return` inside a loop exits the enclosing function:

```oxynium
def f() {
    let mut i = 0
    while {
        i += 1
        if i > 2 -> return
        print(i.Str(), "")
    }
}
f()    // prints 12
```

## `for` loops

Iterates over any value that has `len()` and `at_raw()` methods. The built-in iterable types
are `List<T>`, `Str`, and `Range`.

```text
for item in collection {
    // body
}

// With index (0-based):
for item, index in collection {
    // body
}

// Arrow form:
for item in collection -> print(item.Str())
```

`for` declares its loop variables in the enclosing function, so it can only be used inside a
function, and each loop in a function needs its own variable names (see
[Loop variable scope](#loop-variable-scope)).

**Iterating a `List<T>`** yields values of type `T`:

```oxynium
def main() {
    let arr = List.empty!<Int>()
    arr.push(1)
    arr.push(2)
    arr.push(3)
    for n in arr {
        print(n.Str(), ",")
    }
    // prints 1,2,3,
}
```

**Iterating a `Str`** yields `Char` values:

```oxynium
def main() {
    for c, i in "abc" {
        print(i.Str() + c.Str(), " ")
    }
    // prints 0a 1b 2c
}
```

**Iterating a `Range`** yields `Int` values:

```oxynium
def main() {
    for i in range(5) {
        print(i.Str(), "")    // 01234
    }
}
```

**Custom iterables** — any class with `len` and `at_raw` methods works:

```oxynium
class Tens {
    def len(self) Int -> 3,
    def at_raw(self, i: Int) Int -> i * 10
}
def main() {
    let t = new Tens
    for x in t { print(x.Str(), " ") }    // prints 0 10 20
}
```

Iterating over a non-iterable type (e.g. `Int`, `Bool`) is a `TypeError`.

## Loop variable scope

Loop variables persist after the loop ends. The item variable holds the last item. The index
variable currently holds the number of iterations, not the last index:

```oxynium
def main() {
    for c, i in "abc" {}
    print(i.Str() + c.Str())    // 3c
}
```

Because there is no block scope, a second loop can't reuse the same variable name:

```oxynium
def main() {
    for i in range(2) {}
    for i in range(2) {}    // TypeError: symbol `i` is already declared
}
```

## `break` and `continue` in `for` loops

```oxynium
def main() {
    for i in range(5) {
        if i >= 3 -> break
        print(i.Str(), ",")
    }
    // prints 0,1,2,
}
```

```oxynium
def main() {
    for i in range(5) {
        print(i.Str(), "")
        if i >= 3 -> continue
        print(",", "")
    }
    // prints 0,1,2,34
}
```

## Nested loops

```oxynium
def main() {
    let arr = List.empty!<Int>()
    arr.push(1)
    arr.push(2)
    for n in arr {
        for m in "ab" {
            print(m.Str() + n.Str(), "")
        }
    }
    // prints a1b1a2b2
}
```
