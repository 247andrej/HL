# HL

A lightweight, QBasic-inspired scripting language. The headless sibling of [HGL](https://github.com/247andrej/HGL) — same syntax, no graphics, smaller footprint. Designed for terminal automation, small tools, and constrained hardware like the Raspberry Pi Zero.

## Usage

Download the latest release from [Releases](../../releases) and run:

```bash
./hl yourprogram.hl
```

HL files must use the `.hl` extension.

## Language Reference

### Variables

```
var int x 10
var float y 3.14
var char name Hello there
var list items 1 2 3
var dict cfg
```

**Types:** `int`, `float`, `char`, `list`, `dict`

`char` joins all remaining tokens with spaces, so multi-word strings work without quoting.

`dict` creates an empty dictionary you can populate with `edit dict`.

### Token Resolution

| Syntax | Meaning | Example |
|---|---|---|
| `!name` | Get variable value | `!x` |
| `@name[i]` | List index | `@items[0]` |
| `@name{key}` | Function argument | `@myFunc{arg1}` |
| `~token` | Char to ASCII code | `~!letter` |
| `$name` | Length - 1 | `$items` |
| `token<-i` | Cast to int | `5<-i` |
| `token<-f` | Cast to float | `3<-f` |
| `int(token)` | Inline int cast | `int(!x)` |
| `dbl(token)` | Inline float cast | `dbl(!x)` |
| `chr(token)` | Inline string cast | `chr(!x)` |
| `type(token)` | Get type name | `type(!x)` |
| `strip(token)` | Strip whitespace | `strip(!name)` |
| `comp(a b)` | String compare, returns 0 or 1 | `comp(!a hello)` |
| `inp()` | Read line from stdin | `inp()` |

Multi-word arguments inside parentheses work because tokens are merged automatically.

### Output

```
show Hello !name
```

`show` joins all its tokens with spaces. There are no string literals — quotes are printed as-is.

### Math & Operations

```
change x add 5<-i
change x sub 1<-i
change x mul 2<-i
change x div 3<-i
change x mod 2<-i
change x set 100<-i
change x random 1 10
change x sin !angle
change x cos !angle
change x sqrt
change x abs
change x pow 2<-i
change x flr
change x cel
```

Note: numeric literals are strings by default. Use `<-i` or `<-f` to cast them.

### Control Flow

```
choice !x = 10<-i -> 200
choice !name = hello -> 300
choice !x > 5<-i <- 3
```

Comparators: `=`, `<`, `>` (numeric only for `<` and `>`)

`->` jumps to a label. `<-` jumps by a relative offset from the current line.

**Note:** label jumps (`->`) only work at the top level, not inside function bodies. Use relative jumps (`<-`) inside functions.

### Switch Statements

```
switch !x
case 1<-i
show one
endc
case 2<-i
show two
endc
case 3<-i
show three
endc
ends
```

### Labels & Goto

Lines starting with a number are labels:

```
1
show this is label 1
goto 1
```

### Functions

```
define func greet
show Hello!
endf

call greet
```

With arguments:

```
define func add <- a b
show @add{a} @add{b}
endf

call add <- 5 10
```

The number of arguments must match the function's declaration.

### Type Conversion

```
warp x int
warp y float
warp z char
```

### Lists

```
var list myList 1<-i 2<-i 3<-i
edit list myList change 0 99<-i
edit list myList remove 1
```

### Dictionaries

```
var dict myDict
edit dict myDict change name andrej
edit dict myDict change age 25
edit dict myDict remove age
```

### File I/O

```
file txt create myfile
file txt write myfile
This content gets written to the file.
endfw
file txt read myfile -> contents
file json read data -> myData
```

Supported types: `txt`, `json`

### Directory Operations

```
dir create newfolder
dir delete oldfolder
dir list ./mydir -> files
```

### System Commands

```
sys [ls -la]
sys [echo !myVar] *
```

The `*` suffix resolves HL variables inside the brackets before execution.

### Wait

```
wait 1.5
```

### Delete a Variable

```
del x
```

---

## Example: Greeter

```
show whats your name?
var char name inp() *
show hello strip(!name)
```

## Example: Counter

```
var int x 0

1
change x add 1<-i
show !x
choice !x > 5<-i -> 1
show done
```

## Example: Guessing Game

```
var int target 0
change target random 1 10
var int guess 0

1
show guess a number between 1 and 10:
var char raw inp() *
warp raw int
change guess set !raw

choice !guess = !target -> 1
show got it! the number was !target
```

## Example: FizzBuzz

```
var int i 0
var int check 0

1
change i add 1<-i

change check set !i
change check mod 15<-i
choice !check = 0<-i -> 100
show FizzBuzz
goto 400

100
change check set !i
change check mod 5<-i
choice !check = 0<-i -> 200
show Buzz
goto 400

200
change check set !i
change check mod 3<-i
choice !check = 0<-i -> 300
show Fizz
goto 400

300
show !i

400
choice !i = 20<-i -> 1
```

## License

MIT
