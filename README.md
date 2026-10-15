# HL

**HL v0.3** is an experimental interpreted programming language implemented in Python. It provides variables, expressions, lists and dictionaries, branching, user-defined functions, and basic filesystem commands using a compact, line-oriented syntax.

> HL is distributed as a compiled interpreter. Its source code is not publicly available.

## Getting started

Download the HL v0.3 executable from the GitHub release assets and extract it. Create an `.hl` file and pass its path to the interpreter executable. The exact launch command depends on the name and platform of the packaged build.

Create `myprogram.hl`:

```hl
set !name "world"
show "Hello," !name
show add(2 mul(3 4))
```

Output:

```text
Hello, world
14
```

**Path note:** Use an explicit file path if the program is not in your current working directory.

## Syntax at a glance

- Each line begins with a command such as `show`, `set`, `choice`, or `call`.
- Arguments are separated by whitespace, except whitespace inside quoted strings or parenthesized expressions.
- Strings can be enclosed in single or double quotes; integers and decimal numbers are recognized automatically.
- Expressions use **function-call notation**, with **space-separated** arguments: `add(2 3)`, not `2 + 3` or `add(2, 3)`.
- Expressions can be nested: `mul(add(2 3) sub(10 4))`.
- `comp(...)` and `not(...)` return integer `1` or `0`.
- Unrecognized commands may currently be ignored silently. There is no implemented comment syntax to rely on.

## Variables and types

Use `set !name value` to create or replace a variable, and `!name` to read it.

```hl
set !count 10
set !title "HL"
set !ratio 1.5
show !title !count !ratio
set !count add(!count 1)
show !count
```

`type(value)` returns the underlying Python type name, such as `int`, `float`, `str`, `list`, or `dict`.

```hl
show type(123)
show type("hello")
```

`del !name` deletes a variable. `warp name type` performs a type conversion using the currently supported type names `int`, `float`, and `char` (which converts to a string). Note that the first argument of `warp` is resolved as an expression, so its calling convention is unusual and should be treated as experimental.

### Lists

```hl
set !numbers []
set @numbers[-1] 10
set @numbers[-1] 20
set @numbers[0] add(@numbers[1] 5)
show @numbers[0]
```

Output: `25`.

- `set !numbers []` initializes an empty list.
- `set @numbers[-1] value` **appends** an element (it does not replace the final element).
- `@numbers[index]` reads an element, using zero-based indexing.
- `set @numbers[index] value` replaces an existing element.
- `del @numbers[index]` removes an element.

**Limitation:** The Python v0.3 implementation does not support chained indexing such as `@outer[0][1]`. You can store a list inside another list and retrieve the inner list as a whole.

### Dictionaries

```hl
set !person {}
set @person("name") "Alex"
show @person("name")
```

Dictionary keys are accessed with parentheses after `@`. Deletion uses `del @person("name")`. Dictionary/list values may also come from reading JSON files.

## Expressions and built-ins

Expressions are evaluated recursively. Available operations in the supplied source:

| Expression | Meaning |
| --- | --- |
| `add(a b)` | Addition; also concatenation for compatible types |
| `sub(a b)` | Subtraction |
| `mul(a b)` | Multiplication |
| `div(a b)` | Division; current implementation returns `0` if either argument is zero |
| `mod(a b)` | Modulo |
| `pow(a b)` | Exponentiation |
| `random(a b)` | Random integer between the endpoints |
| `sin(x)`, `cos(x)` | Trigonometry (radians) |
| `sqrt(x)` | Square root |
| `abs(x)`, `flr(x)`, `cel(x)` | Absolute value, floor, ceiling |
| `comp(a b)` | Equality comparison, returning `1` or `0` |
| `not(value)` | Logical negation, returning `1` or `0` |
| `int(value)` | Convert to integer (with fallback behavior) |
| `dbl(value)` | Convert to float (with fallback behavior) |
| `chr(value)` | Convert to string (despite its name, **not** a Unicode code-point conversion) |
| `type(value)` | Python type name as a string |
| `strip(value)` | Strip leading and trailing whitespace from a string |
| `inp()` | Read a line from standard input |
| `~value` | Attempt to obtain the character code of a value |

**Verified in the updated v0.3 source:** Unary math calls work with one argument, including when nested. For example, `sqrt(25)` returns `5.0`, `abs(-10)` returns `10`, and `add(sqrt(25) abs(-3))` returns `8.0`. The dispatcher currently does not enforce exact argument counts: surplus operands may be ignored (`add(2 3 4)` returns `5`), while missing operands can produce an error.

```hl
show comp(add(2 mul(3 4)) 14)
show not(comp("yes" "no"))
show type(dbl(5))
```

### Verified math examples

```hl
show sqrt(25)
show abs(-10)
show sin(0)
show cos(0)
show flr(2.9)
show cel(2.1)
show add(sqrt(25) abs(-3))
```

Expected output:

```text
5.0
10
0.0
1.0
2
3
8.0
```

These outputs were confirmed by running the updated interpreter.

## Output and input

```hl
show "The answer is" add(6 7)
set !name inp()
show "Hello" !name
```

`show` evaluates its arguments and prints them separated by spaces. `inp()` waits for user input.

## Branching and line control

### `choice`

`choice left comparator right <- offset` conditionally skips forward when the comparison **fails**. The available comparators are `=`, `<`, and `>` for numbers, and `=` for other values. With `<-`, the offset is relative to the current source line. `->` instead targets a numbered line label; label handling is experimental.

```hl
set !age 18
choice !age = 18 <- 2
show "Age is 18"
show "Continuing"
```

This is a low-level conditional jump, not a conventional `if` block. Use care with offsets.

### `switch`

```hl
set !option 2
switch !option
    case 1
        show "one"
    endc
    case 2
        show "two"
    endc
ends
```

The interpreter scans cases and executes the body of a matching case. Nested switches and some malformed inputs are not robustly handled in the current implementation.

### `goto`

`goto` jumps to a numbered line label previously collected by the launcher. This is an experimental, source-line-oriented feature; test your labels and jumps carefully.

## User-defined functions

```hl
define func greet <- who
    show "Hello" @greet{who}
endf

call greet <- "world"
```

- `define func name <- arg1 arg2` starts a definition; `endf` closes it.
- `call name <- value1 value2` invokes it.
- Within a function, `@functionName{argumentName}` accesses a supplied parameter.
- Functions without arguments omit the `<-` portion.
- The current implementation uses shared global variable storage and a function-argument dictionary. There is no documented return-value mechanism; functions can communicate by setting variables.

## Files and directories

### Text files

```hl
file txt create note
file txt write note
Hello from HL!
endfw
file txt read note -> contents
show !contents
file txt remove note
```

Files are addressed by a basename, with `.txt` appended automatically. A write block copies the intervening lines into the file. The `read` command stores the result in a variable named by the bare identifier after `->`.

### JSON

`file json create name`, `file json read name -> variable`, and `file json remove name` are available. JSON reads use Python's `json.load()`. **Writing JSON is not serialized automatically:** the current `write` block writes literal text, so it is up to the program to provide valid JSON content.

### Directories

```hl
dir create scratch
dir list .
dir list . -> entries
show !entries
dir delete scratch
```

`dir delete` removes empty directories only. Filesystem commands operate with the Python process's filesystem permissions and working directory.

## Other commands

- `wait seconds`: pauses execution, e.g. `wait 1`.
- `sys [command]`: invokes a system command using Python `subprocess.run`; its handling of command strings and arguments is currently limited. Only run trusted HL scripts.

## Known limitations

1. Chained list indexing is not implemented in HL v0.3.
2. Escaped quotes are not decoded into ordinary quote characters; parser handling of escape sequences is incomplete.
3. Unary mathematical operations work, but exact arithmetic argument counts are not enforced; surplus operands may be ignored.
4. `choice`, `goto`, `switch`, and block scanning have edge cases; avoid deeply nested control-flow constructs.
5. Error reporting is inconsistent, and some invalid commands are ignored.
6. There is no stable library/import interface, package installer, or formal language specification yet.
7. `sys` executes external commands. Do not run untrusted HL programs.

## Project status

HL is a personal programming-language project distributed as an executable. The interpreter source code is private. Bug reports and reproducible `.hl` examples are welcome.
