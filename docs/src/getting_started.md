# Getting Started

Oxynium is a statically-typed, compiled language. It compiles directly to x86-64 NASM assembly,
which is then assembled and linked by GCC to produce a native binary. It targets Linux x86-64
and macOS x86-64.

## Installation

You can install Oxynium on any Linux/macOS system by running the following command in your terminal:

```bash
curl -sSL https://oxynium.org/scripts/install | bash
```

#### Unstable version

You can also install the latest version by passing the `latest` argument to the installation
script.

```bash
curl -sSL https://oxynium.org/scripts/install | bash -s -- latest
```

#### Requirements

Oxynium requires `cargo`, `rust`, `nasm`, and `gcc` to be installed on your system.

## Upgrade

To upgrade to the latest version, run the installation script again:

```bash
curl -sSL https://oxynium.org/scripts/install | bash
```

## Uninstallation

```bash
curl -sSL https://oxynium.org/scripts/uninstall | bash
```

## Compiling and running

The compiler binary is `oxy`. Pass a source file to compile it:

```bash
oxy hello.oxy
```

This produces an executable called `oxy-out` in the current directory. Run it directly:

```bash
./oxy-out
```

Use `-o` to choose a different output name, e.g. `oxy hello.oxy -o hello`.

You can also pass the program source on the command line with `-e`:

```bash
oxy -e 'print("Hello, world!")' && ./oxy-out
```

> **Note:** Oxynium produces x86-64 binaries only. On Apple Silicon Macs, linking does not
> currently work out of the box.

## Hello, World

The simplest possible Oxynium program is a single top-level statement:

```oxynium
print("Hello, world!")
```

Output:
```
Hello, world!
```

For larger programs, define a `main` function:

```oxynium
def main() {
    print("Hello, world!")
}
```

When `main` is defined, all executable code must live inside it — top-level statements
outside `main` are a compile error. See [Program Structure](./program_structure.md) for details.

## Accepting command-line arguments

```oxynium
def main(args: List<Utf8Str>) {
    print(args.len().Str())
    for arg in args {
        print(arg.Str())
    }
}
```

`args` includes the binary name as the first element, so a program run with no user arguments
has `args.len()` equal to `1`. `Utf8Str` is the raw OS string type; call `.Str()` to convert
it to the standard `Str` type before using string operations.

## Compilation pipeline

```
source.oxy
  → Lexer (tokens)
  → Parser (AST)
  → Type Checker
  → Code Generator (NASM assembly)
  → Post-processor
  → NASM assembler
  → GCC linker
  → native binary
```

## Error types

The compiler emits one of the following error kinds:

| Error kind        | When it occurs |
|-------------------|----------------|
| `SyntaxError`     | The source text cannot be parsed |
| `TypeError`       | A type rule is violated |
| `UnknownSymbol`   | A name is used that has not been declared |
| `IoError`         | A file can't be read, or assembling/linking fails (e.g. a called `extern def` has no implementation) |
| `NumericOverflow` | An integer literal exceeds the 64-bit signed range |
| `InvalidArguments`| The command-line arguments passed to `oxy` are invalid |