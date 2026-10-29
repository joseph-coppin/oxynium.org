# Types

## Built-in types

| Type     | Kind      | Description                | Default (`new T`) | Size    |
|----------|-----------|----------------------------|-------------------|---------|
| `Int`    | primitive | 64-bit signed integer      | `0`               | 8 bytes |
| `Bool`   | primitive | Boolean                    | `false`           | 8 bytes |
| `Char`   | primitive | Single Unicode character   | null char (0)     | 8 bytes |
| `Ptr<T>` | primitive | Unmanaged pointer          | —                 | 8 bytes |
| `Str`    | class     | Immutable character string | `""` (empty)      | pointer |
| `Void`   | class     | Unit / no value            | —                 | —       |

See [Primitives](./primitives.md) for what makes a type primitive.

`Void` is the return type of functions that produce no value. `new Void` is a valid expression
used in low-level code but has no useful value.

## Literals

```oxynium
def main() {
    let i = 42          // Int
    let n = -7          // unary minus applied to Int 7
    let t = true        // Bool
    let f = false       // Bool
    let c = 'a';        // Char (see note below about the `;`)
    let nl = '\n';      // Char — newline
    let s = "hello"     // Str
    let e = ""          // Str — empty string
}
```

A line ending in a character literal currently needs an explicit `;`
(see [Semicolons](./program_structure.md#semicolons)).

## Type annotations

Annotations appear after the identifier, separated by `:`:

```oxynium
def f(a: Int, b: Bool) Str { return "" }

def main() {
    let x: Int = 5
    let s: Str = "hello"
}
```

## Type inference

The compiler infers types when they can be determined from context. Parameter types are
**never** inferred — they must always be explicit. Return types can be inferred for
single-expression (`->`) functions.

```oxynium
def square(n: Int) -> n * n   // return type inferred: Int

def main() {
    let x = 42               // inferred: Int
    let s = "hi"             // inferred: Str
}
```

Using a variable before it is declared is a `TypeError`:

```oxynium
def f() Int {
    let b = a    // TypeError: a not yet declared
    let a = 5
    return b
}
```

## Generic types

Parameterised with angle brackets. The standard library provides `List<T>`, `Option<T>`,
`Result<T, E>` and `Ptr<T>`. `Range` is not generic.

```oxynium
def f(a: List<Int>, b: Option<Str>, c: Result<Int, Str>, d: Ptr<Bool>) {}
```

Nesting is allowed:

```oxynium
def g(a: List<Option<Int>>, b: Option<List<Str>>, c: Result<Option<Int>, Str>) {}
```

## Optional shorthand

`T?` is syntactic sugar for `Option<T>`. Multiple `?` can be chained:

```oxynium
def main() {
    let a: Int? = Option.some!<Int>(1)                 // Option<Int>
    let b: Int?? = Option.some!<Option<Int>>(a)        // Option<Option<Int>>
    let c: Str? = Option.none!<Str>()                  // Option<Str>
}
```

## Function types

Written as `Fn(Param1, Param2, ...) ReturnType`. Used as parameter or variable types.

```oxynium
def apply(f: Fn(Int) Int) Int { return f(5) }
def run(f: Fn() Void) { f() }
def combine(f: Fn(Int, Str) Bool) Bool { return f(1, "a") }
```

Generic function types (`Fn<T>(T) T`) are not permitted — this is a `SyntaxError`.

## The `typeof` operator

`typeof expr` evaluates to a `Str` naming the compile-time type of the expression.
See [typeof](./typeof.md).
