# typeof

`typeof expr` evaluates at compile time to a `Str` naming the type of the expression.

## Syntax

```text
typeof expr
typeof(expr)    // parentheses are optional
```

`typeof` always returns a `Str`. Since its result is a `Str`, `typeof typeof x` is valid and
returns `"Str"`.

`typeof` applies to the whole expression after it, so `typeof 1 + 2` is `typeof (1 + 2)`.
Wrap it in parentheses to use the result in a larger expression:

```oxynium
print((typeof 1) + "!")    // Int!
```

## Examples

```oxynium
print(typeof "abc")         // Str
print(typeof 2)             // Int
print(typeof true)          // Bool
print(typeof new Void)      // Void
print(typeof typeof Bool)   // Str
```

On classes and instances:

```oxynium
class C
print(typeof C)       // C
print(typeof new C)   // C
```

On functions and calls:

```oxynium
def a() {}
print(typeof a)      // Fn a() Void
print(typeof a())    // Void
```

Generic classes show their type parameters, and anonymous functions show a
compiler-generated name:

```oxynium
def main() {
    print(typeof List)                 // List<T>
    print(typeof fn (x: Int) -> x)     // e.g. Fn fn42@mainoxy(x: Int) Int
}
```

## Inside generic functions

`typeof` reflects the concrete type at the call site. Within the generic function body,
`typeof T` prints the type parameter name as a string, not the concrete type:

```oxynium
def show<T>(a: T) T {
    print(typeof a, ",")     // prints "T"
    print(typeof T, ",")     // prints "T"
    return a
}
def main() {
    let int = show!<Int>(1)
    print(typeof int, ",")           // Int
    print(typeof show!<Str>("Hi"))   // Str
}
// prints T,T,Int,Str
```

## Restrictions

The following are `SyntaxError`:
- `typeof` alone with nothing after it
- `typeof 1 2` (two expressions)
- `typeof ()` (empty parentheses)
- `typeof while {}` or `typeof if true {}` (control-flow as argument)

Using an undeclared name — `typeof Foo` where `Foo` is not defined — is an `UnknownSymbol`.
