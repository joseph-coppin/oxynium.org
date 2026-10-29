# Classes

## Defining a class

```oxynium
class Point {
    x: Int,
    y: Int
}
```

Fields are separated by commas. A trailing comma is allowed. Fields and method definitions
can be interleaved in any order.

Empty class forms:

```oxynium
class Token
class Empty;
class Nothing {}
```

Classes cannot be defined inside other classes (`SyntaxError`). Redefining an existing class
name is a `TypeError`. This includes the names the standard library reserves, such as `Box`
and `Error` (see [Reserved names](./program_structure.md#reserved-names)).

## Instantiation (`new`)

```oxynium
class Point { x: Int, y: Int }

def main() {
    let p = new Point { x: 3, y: 4 }
}
```

All fields must be provided. Missing fields, unknown fields, or wrongly-typed fields are a
`TypeError`. Don't repeat a field: duplicates are not currently detected and cause a
crash at runtime. For classes with no fields, `new C`, `new C{}`, and `new C { }` are all valid.

**Field shorthand:** if a local variable has the same name as a field, the name alone can be used:

```oxynium
class C { a: Int }

def main() {
    let a = 1
    print(new C { a }.a.Str())    // prints 1
}
```

## Field access and assignment

Fields can be read and assigned through any variable — field mutation is independent of
whether the variable itself is `mut`:

```oxynium
class A { a: Int }

def main() {
    let a = new A { a: 1 }
    a.a = 2
    print(a.a.Str())    // 2
}
```

## Instance methods

Defined with `self` as the first parameter. `self` must not have a type annotation:

```oxynium
class Point {
    x: Int,
    y: Int,
    def magnitude_sq(self) Int -> self.x * self.x + self.y * self.y
}

print(new Point { x: 3, y: 4 }.magnitude_sq().Str())    // 25
```

## Static methods

Methods without a `self` parameter. Called on the class name, not on an instance:

```oxynium
class Counter {
    count: Int,
    def zero() Counter -> new Counter { count: 0 }
}
print(Counter.zero().count.Str())    // 0
```

**Calling instance methods statically** by passing the instance explicitly is also valid:

```oxynium
class S {
    def f(self, msg: Str) Str { return msg }
}
def main() {
    print(S.f(new S, "abc"))    // abc
}
```

## Default parameters on methods

Same rules as for standalone functions:

```oxynium
class Greeter {
    def say(self, msg: Str = "Hello") {
        print(msg)
    }
}
new Greeter{}.say()        // Hello
new Greeter{}.say("Hi")   // Hi
```

## Methods as first-class values

Methods can be stored in variables and used as first-class functions:

```oxynium
class S {
    def g() -> 1
}
def main() {
    let g = S.g
    print(g().Str())    // 1
}
```

An instance method taken this way becomes a function whose first argument is the instance:

```oxynium
class S {
    def h(self) -> 2
}
def main() {
    let h = S.h
    print(h(new S).Str())    // 2
}
```

## Operator overloading

See [Operators](./operator_overriding.md) for full details:

```oxynium
class Foo {
    x: Int,
    def + (self, other: Foo) Foo {
        return new Foo { x: self.x + other.x }
    }
}
```

## Class composition

```oxynium
class Inner { value: Int }
class Outer { inner: Inner, tag: Str }

def main() {
    let o = new Outer {
        inner: new Inner { value: 42 },
        tag: "hello"
    }
    print(o.inner.value.Str())    // 42
    print(o.tag)                  // hello
}
```
