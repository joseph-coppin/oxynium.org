# Primitives

Primitives are types backed by a single 64-bit value rather than a heap-allocated struct.
They are not stored as a reference.

The built-in primitives are `Int`, `Bool`, `Char`, and `Ptr<T>`. These are defined using the
`primitive` keyword in the standard library. User code should not define its own primitive
types.

`new Int` produces `0`. `new Bool` produces `false`. `new Char` produces the null character.

## `Int`

64-bit signed integer. Range: −2⁶³ to 2⁶³−1.

Integer literals exceeding 2⁶³−1 produce a `NumericOverflow` error at compile time. Because
`-` is applied after the literal is parsed, the minimum value can't be written directly;
use `-9223372036854775807 - 1`.

Runtime overflow wraps around silently (two's complement):

```oxynium
print((9223372036854775807 + 1).Str())    // -9223372036854775808
```

| Method    | Description |
|-----------|-------------|
| `Str`     | Convert to decimal string |
| `Bool`    | `0` → `false`, any other value → `true` |
| `max(other)` | Return the larger of the two values |
| `min(other)` | Return the smaller of the two values |
| `abs`     | Absolute value |
| `compare(other)` | Returns `-1`, `0`, or `1` |

Called with no argument, `max()` returns the largest `Int` and `min()` the smallest
(`0.min()` is a common way to get the minimum `Int`).

Method calls bind tighter than unary `-`, so negative receivers need parentheses:
`(-5).abs()` is `5`, but `-5.abs()` is `-(5.abs())`.

```oxynium
print((-5).abs().Str())      // 5
print(3.max(7).Str())        // 7
print(10.min(4).Str())       // 4
print(0.Bool().Str())        // false
print((-1).Bool().Str())     // true
print(Int.compare(1, 2).Str()) // -1
```

## `Bool`

Boolean type. Literals are `true` and `false`.

Operators: `&&` (AND), `||` (OR), `!` (NOT). Neither `&&` nor `||` short-circuits.
`Bool` does not support `==` or `!=`.

| Method | Description          |
|--------|----------------------|
| `Str`  | `"true"` or `"false"` |

## `Char`

Single Unicode character. Single-quoted literals: `'a'`, `'Z'`, `'💖'`.

Escape sequences: `'\n'` (newline), `'\t'` (tab), `'\r'` (CR), `'\\'` (backslash),
`'\''` (single quote).

`Char` supports `==` and `!=`, but not `<`, `>`, `<=` or `>=`.

| Method           | Description |
|------------------|-------------|
| `Str`            | Convert to a one-character string |
| `Int`            | Character as an integer (the code point for ASCII; see below) |
| `from_int(i)`    | Create `Char` from an integer (static; inverse of `Int`) |
| `is_digit`       | True for `'0'`–`'9'` |
| `is_alphabetic`  | True for `'A'`–`'Z'` or `'a'`–`'z'` |
| `is_uppercase`   | True for `'A'`–`'Z'` |
| `is_lowercase`   | True for `'a'`–`'z'` |
| `is_alphanumeric`| True for alphabetic or digit |
| `is_ascii`       | True if code point < 128 |
| `is_whitespace`  | True for `' '`, `'\n'`, `'\t'`, `'\r'` |

A `Char` is stored as its UTF-8 bytes packed into 64 bits. `Int` and `from_int` work on that
packed value, so they match Unicode code points only for ASCII characters (`'a'.Int()` is
`97`, but `'💖'.Int()` is `2526191600`, not `128150`).

```oxynium
print('A'.is_uppercase().Str())    // true
print('3'.is_digit().Str())        // true
print('a'.Int().Str())             // 97
print(Char.from_int(65).Str())     // A
```

## `Ptr<T>`

An unmanaged pointer to a heap-allocated value. No garbage collection or reference counting.
Use `Ptr<T>` only for low-level code — prefer `Option<T>` or `Result<T, E>` for safe values.

```oxynium
def main() {
    let p = Ptr.make!<Int>(42)
    print(p.unwrap().Str())     // 42
    print(p.is_null().Str())    // false
}
```

See [Memory Management](./memory.md) for full details.
