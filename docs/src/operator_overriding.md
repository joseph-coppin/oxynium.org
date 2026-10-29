# Operators

## Arithmetic operators

All arithmetic operators work on `Int` operands and return `Int`. Mixing types is a `TypeError`.

| Operator | Description      | Example | Result |
|----------|------------------|---------|--------|
| `+`      | Addition         | `3 + 4` | `7`    |
| `-`      | Subtraction      | `10 - 3`| `7`    |
| `*`      | Multiplication   | `3 * 4` | `12`   |
| `/`      | Integer division | `7 / 2` | `3`    |
| `%`      | Modulo           | `7 % 2` | `1`    |

Integer division truncates toward zero. Division by the literal `0` is a compile-time
`TypeError`. Modulo by the literal `0`, and division or modulo by a zero value at runtime,
crash the program (SIGFPE).

Operator precedence follows standard mathematical convention:
- `*`, `/`, `%` bind tighter than `+`, `-`
- Left-to-right associativity within the same precedence level

```oxynium
print((1 + 2 * 3 + 4 * 5).Str())      // 27  (= 1 + 6 + 20)
print((1 + 2 * 3 - 4 * 5 / 2).Str())  // -3
```

## Unary operators

| Operator | Description        | Example | Result  |
|----------|--------------------|---------|---------|
| `-`      | Arithmetic negation| `-5`    | `-5`    |
| `!`      | Logical NOT        | `!true` | `false` |

Unary `-` applies only to `Int`. `!` applies only to `Bool`. Applying `!` to an `Int`
is a `TypeError`.

Method calls and field access bind tighter than both: `-5.abs()` is `-(5.abs())` and
`!a.b` is `!(a.b)`.

## Comparison operators

Return `Bool`. For `Int`, all six are built in. `Str` and `Char` support `==` and `!=` only.
`Bool` supports none of them.

| Operator | Description           |
|----------|-----------------------|
| `==`     | Equal                 |
| `!=`     | Not equal             |
| `<`      | Less than             |
| `>`      | Greater than          |
| `<=`     | Less than or equal    |
| `>=`     | Greater than or equal |

Comparison operators are **not chainable** — `1 < 2 < 3` is a `TypeError` because
the result of `1 < 2` is `Bool`, and `Bool < Int` is not defined.

## Logical operators

| Operator | Description  | Operand types | Return type |
|----------|--------------|---------------|-------------|
| `&&`     | Logical AND  | `Bool, Bool`  | `Bool`      |
| `\|\|`   | Logical OR   | `Bool, Bool`  | `Bool`      |
| `!`      | Logical NOT  | `Bool`        | `Bool`      |

`&&` and `||` do **not** short-circuit — both sides are always evaluated.

`&&` and `||` have the **same** precedence and are evaluated left to right, so
`true || false && false` is `(true || false) && false`, which is `false`. Use parentheses
when mixing them.

```oxynium
print((true && false).Str())    // false
print((true || false).Str())    // true
print((!true).Str())            // false
```

```oxynium
print((0 && 1).Str())    // TypeError: cannot use operator `&&` on type `Int`
```

## String concatenation

`+` on two `Str` values returns a new `Str`:

```oxynium
print("Hello, " + "world!")    // Hello, world!
print("" + "abc")              // abc
```

## None-coalescing operator (`??`)

Unwraps an `Option<T>`. Returns the contained value if `some`, or the right-hand default
if `none`:

```oxynium
def main() {
    let x: Int? = Option.none!<Int>()
    let v = x ?? 42     // v = 42

    let y: Int? = Option.some!<Int>(7)
    let w = y ?? 42     // w = 7
}
```

`??` is left-associative, so `a ?? b ?? c` is `(a ?? b) ?? c`.

## Compound assignment

`+=`, `-=`, `*=`, `/=`, `%=` — see [Variables](./variables.md). These can't be overloaded
directly, but `a += b` is the same as `a = a + b`, so it works on any class that overloads `+`.

## Full precedence table (high to low)

| Level | Operators                                      | Associativity |
|-------|------------------------------------------------|---------------|
| 1     | call `f()`, field/method access `.`            | left          |
| 2     | `!`                                            | right         |
| 3     | unary `-`                                      | right         |
| 4     | `*`, `/`, `%`                                  | left          |
| 5     | `+`, `-`                                       | left          |
| 6     | `==`, `!=`, `<`, `>`, `<=`, `>=`               | left          |
| 7     | `??`                                           | left          |
| 8     | `&&`, `\|\|`                                   | left          |
| 9     | `=`, `+=`, `-=`, `*=`, `/=`, `%=`              | right         |

`typeof` takes a whole expression, so it binds more loosely than all of these:
`typeof 1 + 2` is `typeof (1 + 2)`. Use `(typeof x) + "..."` to concatenate its result.

## Operator overloading

Classes may define methods named after binary operator symbols. Only binary operators may
be overloaded; unary operators (`!`) may not.

Valid overloadable operators: `+`, `-`, `*`, `/`, `%`, `==`, `!=`, `<`, `>`, `<=`, `>=`,
`&&`, `||`, `??`.

Rules:
- The method must take exactly one parameter in addition to `self`.
- That parameter must not have a default value.
- Operator methods can only be defined inside a `class`, not at the top level.
- Compound assignment operators (`+=` etc.) cannot be overloaded.

```oxynium
class Vec2 {
    x: Int,
    y: Int,
    def + (self, other: Vec2) Vec2 ->
        new Vec2 { x: self.x + other.x, y: self.y + other.y },
    def == (self, other: Vec2) Bool ->
        self.x == other.x && self.y == other.y
}

def main() {
    let a = new Vec2 { x: 1, y: 2 }
    let b = new Vec2 { x: 3, y: 4 }
    let c = a + b
    print(c.x.Str())          // 4
    print(c.y.Str())          // 6
    print((a == b).Str())     // false
}
```

Invalid overload forms (each line is a separate error):

```text
class C {
    def ! (self) C { return self }              // SyntaxError: cannot overload `!` operator
    def += (self, other: C) C { return self }   // SyntaxError: compound assignment not overloadable
    def + (self) C { return self }              // TypeError: must have exactly 2 parameters
}
def + (a: C, b: C) C { return a }               // SyntaxError: `+` is not a valid function name
```
