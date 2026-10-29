# Program Structure

## Two modes of execution

**Script mode** — no `main` function. Top-level statements execute in the order they appear:

```oxynium
const MSG = "Hello"
print(MSG)          // runs immediately
print("World")      // runs immediately
```

Local variables cannot be declared at the top level, even in script mode, so `let` and `for`
(which declares its loop variables) must be inside a function. Use `main` for anything that
needs them.

**Main mode** — a `main` function is defined. Only `main`, `const`, `class`, `def`, and
`extern def` may appear at the top level. Any executable statement at the top level, before
or after `main`, is a `SyntaxError`:

```oxynium
def f() {}
def main() {
    print("hi")
}
f()           // SyntaxError: cannot have top level statements and 'main' function
```

## What can appear at the top level

In script mode: `const`, `class`, `def`, `extern def`, and executable statements other than
`let` and `for`.
In main mode: `const`, `class`, `def`, `extern def` only.

## Semicolons

Semicolons are optional. The compiler performs automatic end-of-statement insertion. Both
styles compile identically:

```oxynium
print("a"); print("b")
print("a")
print("b")
```

> **Note:** end-of-statement insertion currently does not happen after a line that ends with
> a character literal (`let c = 'a'`), or before a line that starts with a character literal
> or `!`. End the previous line with `;` in those cases.

## Declaration order

The compiler performs a setup pass before type checking, so functions and classes can be
used before their definition in the source file:

```oxynium
def f(a: A) A { return a }
class A;    // A is used above its definition — valid
```

Local variables, however, cannot be used before their `let` declaration in the same
function body — this is a `TypeError`.

## Locally-defined classes

Classes can be defined inside function bodies and are scoped to that function:

```oxynium
def main() {
    class C
    let x = new C    // C is usable here
}
def f(a: C) {}    // UnknownSymbol: C is not in scope here
```

## Comments

Single-line comments start with `//` and run to the end of the line:

```oxynium
// This entire line is a comment.
def main() {
    let x = 1  // This is an inline comment.
}
```

There is no block comment syntax. Use multiple `//` lines.

`///` is used in the standard library for documentation annotations. In user code, `///`
behaves identically to `//`.

## Reserved names

The standard library reserves a number of class names for future use, including `Box`,
`Map`, `Set`, `Vec`, `Array`, `String`, `Error`, `Test`, `Path`, `Iter` and `Iterator`
(see `std/target-any/reserved.oxy` in the compiler). Defining a class with one of these
names is a `TypeError`.
