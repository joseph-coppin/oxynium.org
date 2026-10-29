# Standard Library

All standard library types are automatically available — no import needed.

| Type / Function  | Description |
|------------------|-------------|
| `Int`            | 64-bit signed integer |
| `Bool`           | Boolean value |
| `Char`           | Single Unicode character |
| `Str`            | Immutable string with rich methods |
| `Utf8Str`        | Raw UTF-8 string for OS interop |
| `List<T>`        | Dynamic resizable array |
| `Option<T>`      | Optional / nullable value |
| `Result<T, E>`   | Success-or-error value |
| `Range`          | Integer sequence (start, end, step) |
| `Ptr<T>`         | Unsafe heap pointer |
| `range()`        | Constructor for `Range` values |
| `print()`        | Print to stdout |
| `input()`        | Read a line from stdin |
| `panic()`        | Abort with an error message |
| `exit()`         | Terminate with an exit code |

---

## `Str`

Immutable string. Uses an internal UTF-64 encoding — each Unicode character occupies exactly
8 bytes, giving O(1) indexing by character position.

```oxynium
def main() {
    let a = "hello"     // Str literal
    let b = ""          // empty string
    let c = "line1\
line2"                  // continuation: "line1line2"
}
```

Escape sequences: `"\n"`, `"\t"`, `"\r"`, `"\\"`, `"\""`.

| Method         | Signature | Description |
|----------------|-----------|-------------|
| `len`          | `(self) Int` | Number of characters (not bytes) |
| `at`           | `(self, i: Int) Option<Char>` | Character at index; supports negative indices |
| `at_raw`       | `(self, i: Int) Char` | Character at index; unchecked |
| `+`            | `(self, other: Str) Str` | Concatenate |
| `concat`       | `(self, other: Str) Str` | Same as `+` |
| `contains`     | `(self, other: Str) Bool` | True if `other` appears as a substring |
| `find`         | `(self, other: Str) Int` | Index of first occurrence, or `-1` |
| `starts_with`  | `(self, other: Str) Bool` | True if string begins with `other` |
| `ends_with`    | `(self, other: Str) Bool` | True if string ends with `other` |
| `substr`       | `(self, start=0, end=<end of string>) Str` | Substring (supports negative indices) |
| `repeat`       | `(self, n: Int) Str` | Repeat `n` times |
| `reversed`     | `(self) Str` | Reversed copy |
| `replace`      | `(self, search: Str, replace_with="", max=-1) Str` | Replace occurrences (`max=-1` replaces all) |
| `insert`       | `(self, index: Int, other: Str) Str` | Insert at position |
| `remove`       | `(self, index: Int, count=1) Str` | Remove characters |
| `split`        | `(self, separator: Str) List<Str>` | Split into substrings |
| `join`         | `(self, l: List<Str>) Str` | Join list with `self` as separator |
| `List`         | `(self) List<Char>` | Convert to list of characters |
| `List_strings` | `(self) List<Str>` | Convert to list of one-character strings |
| `Int`          | `(self) Result<Int, Str>` | Parse as decimal integer |
| `Utf8Str`      | `(self) Utf8Str` | Convert to raw UTF-8 string |
| `utf8_size`    | `(self) Int` | Size in UTF-8 bytes |

There are no list literals, so build a `List` with `push` before passing it to `join`:

```oxynium
def main() {
    print("hello".len().Str())            // 5
    print("hello".reversed())             // olleh
    print("abc".at(-1).unwrap().Str())    // c
    print("a,b,c".split(",").len().Str()) // 3

    let parts = List.empty!<Str>()
    parts.push("a")
    parts.push("b")
    parts.push("c")
    print(",".join(parts))                // a,b,c
}
```

`Str.find` returns `-1` when not found. Searching for an empty string always returns `0`.

`Str.Int()` returns `Result<Int, Str>`:

```oxynium
def main() {
    let r = "42".Int()
    print(r.unwrap().Str())    // 42

    let bad = "abc".Int()
    print(bad.ok.Str())        // false
}
```

---

## `List<T>`

Heap-allocated, resizable array.

```oxynium
def main() {
    let l = List.empty!<Int>()
    let l2 = List.with_capacity!<Int>(10)
}
```

| Method             | Description |
|--------------------|-------------|
| `len`              | Number of elements |
| `push(value)`      | Append an element |
| `pop`              | Remove and return the last element (`Option<T>`) |
| `at(i)`            | Element at index as `Option<T>` (safe; supports negative indices) |
| `at_raw(i)`        | Element at index (unchecked) |
| `set_at(i, val)`   | Set element at index (safe; returns `Result<Void, Str>`) |
| `set_at_unchecked(i, val)` | Set element at index (unchecked) |
| `remove_at(i)`     | Remove and return element at index (`Option<T>`) |
| `map<To>(f)`       | New list with every element transformed; callback receives `(element, index)` |
| `filter(f)`        | New list of the elements matching the predicate |
| `sort(f)`          | New sorted list; the comparator returns negative/0/positive |
| `clone`            | Shallow copy |
| `concat(other)`    | New list with all elements of both |
| `index_of(item, cmp?)` | `Option<Int>`: first index where item matches |
| `contains(item, cmp?)` | True if any element matches |
| `Str`              | `"List[<length>]"` (does not print the elements) |

```oxynium
def main() {
    let nums = List.empty!<Int>()
    nums.push(10)
    nums.push(20)
    nums.push(30)

    print(nums.len().Str())               // 3
    print(nums.at(0).unwrap().Str())      // 10
    print(nums.at(-1).unwrap().Str())     // 30

    let doubled = nums.map!<Int>(fn (n: Int, i: Int) -> n * 2)
    let big = nums.filter(fn (n: Int) -> n > 15)
    let sorted = nums.sort(fn (a: Int, b: Int) -> a.compare(b))
    print(nums.index_of(20).unwrap().Str())   // 1
}
```

Without a comparator, `index_of` and `contains` compare the raw 64-bit values, which for
`Str` and class instances means comparing pointers. Two equal strings are usually not
the same pointer, so provide a comparator:

```oxynium
def main() {
    let strs = List.empty!<Str>()
    strs.push("hello")
    let idx = strs.index_of("hello", fn (a: Str, b: Str) Bool -> a == b)
    print(idx.unwrap().Str())    // 0
}
```

---

## `Option<T>`

Wraps either a value (`some`) or nothing (`none`). The `T?` shorthand is exactly equivalent
to `Option<T>`.

```oxynium
def main() {
    let present = Option.some!<Int>(42)
    let absent  = Option.none!<Int>()

    let a: Int? = Option.some!<Int>(5)
}
```

| Field / method | Description |
|---------------|-------------|
| `is_some`     | Field (`Bool`): `true` if this holds a value |
| `unwrap(err_message="Unwrapping None Option")` | Get the value, or `panic` with `err_message` |
| `or(default)` | Get the value, or return `default` |
| `map<U>(f)`   | Transform the value if present; propagates `none` |
| `is_some_and(f)` | `true` if present and predicate holds |
| `??`          | None-coalescing: same as `or` |

```oxynium
def main() {
    let x: Int? = Option.some!<Int>(5)
    print(x.unwrap().Str())        // 5
    print(x.or(0).Str())           // 5
    print((x ?? 99).Str())         // 5

    let y: Int? = Option.none!<Int>()
    print(y.or(0).Str())           // 0
    print((y ?? 99).Str())         // 99
}
```

---

## `Result<T, E>`

Holds either a success value of type `T` or an error value of type `E`. The `ok: Bool` field
distinguishes the two cases.

```oxynium
def main() {
    let success = Result.ok!<Int, Str>(42)
    let failure = Result.err!<Int, Str>("something went wrong")
}
```

| Field / method | Description |
|-----------|-------------|
| `ok`      | Field (`Bool`): `true` if success |
| `unwrap`  | Return the success value, or `panic` |
| `Option`  | Convert to `Option<T>` (discards the error) |
| `error`   | Return the error as `Option<E>`, or `none` if success |

```oxynium
def main() {
    let r = Result.ok!<Int, Str>(42)
    print(r.ok.Str())         // true
    print(r.unwrap().Str())   // 42

    let bad = Result.err!<Int, Str>("oops")
    print(bad.ok.Str())       // false
    print(bad.error().unwrap()) // oops
}
```

---

## `Range` and `range()`

```text
range(n)             // 0 to n (exclusive), step 1
range(start, end)    // start to end (exclusive), step 1
range(start, end, step)
```

```oxynium
def main() {
    for i in range(5) { print(i.Str(), "") }        // 01234
    for j in range(2, 8, 2) { print(j.Str(), "") }  // 246
}
```

> **Warning:** the length of a range is currently computed as `(end - start) / step`,
> rounded down, so the last value is skipped when `end - start` isn't a multiple of `step`
> (`range(0, 10, 3)` yields `0 3 6`). A range with `start >= end` and a positive `end`
> yields `start` once instead of nothing.

| Method   | Description |
|----------|-------------|
| `len`    | Number of elements in the range |
| `at_raw` | Value at position `i` (0-based) |
| `List`   | Materialise as a `List<Int>` |
| `Str`    | `"Range(<start>, <end>, <step>)"` |

---

## `Utf8Str`

Raw null-terminated UTF-8 string. Used at OS boundaries (command-line arguments, file I/O).
Not interchangeable with `Str`.

```oxynium
def main(args: List<Utf8Str>) {
    for arg in args {
        print(arg.Str())    // convert to Str before operations
    }
}
```

Convert: `utf8_value.Str()` → `Str`; `str_value.Utf8Str()` → `Utf8Str`.

---

## I/O functions

### `print`

```text
def print(msg="", line_end="\n")
```

Writes `msg` to stdout followed by `line_end`. Default `line_end` is `"\n"`.

```oxynium
print("Hello")          // Hello\n
print("Hello", "")      // Hello  (no newline)
print("item", ", ")     // item, 
```

> **Note:** `print` currently writes the internal 8-bytes-per-character representation, so
> each ASCII character is followed by NUL bytes. A terminal hides them, but they show up
> when output is piped or redirected to a file.

### `input`

```text
def input(prompt="", buffer_size=1000) Str
```

Optionally prints `prompt`, then reads a line from stdin. Strips the trailing newline.

```oxynium
def main() {
    let name = input("Enter your name: ")
    print("Hello, " + name + "!")
}
```

### `panic`

```text
def panic(msg="explicit panic")
```

Prints `"PANIC: " + msg` to stdout and exits with code `1`.

### `exit`

```text
def exit(code=0)
```

Terminates the program with the given exit code.
