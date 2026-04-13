# HL

A lightweight, QBasic-inspired scripting language. Designed for terminal use and small tools.

## Usage

Download the latest release from [Releases](../../releases) and run:

```bash
./hl yourprogram.hl
```

## Language Reference

### Assignment

`set` is the universal assignment command. Variables exist when you assign to them — no declarations needed.

```
set !x 10
set !name "hello world"
set !pi 3.14
set !nums []
set !cfg {}
set @nums[0] 99
set @nums[-1] 1 <- append to a list
set @cfg(name) "name"
```

Strings require quotes. Numbers are auto-typed as int or float. Bare unquoted words error out.

### Token Resolution

| Syntax | Meaning |
|---|---|
| `!name` | Variable value |
| `@name[i]` | List element |
| `@name(key)` | Dict value |
| `@name{arg}` | Function argument (inside the function body) |
| `~token` | Char to ASCII code |
| `$name` | Length - 1 |
| `"text"` | String literal |

### Inline Operations

Math and helpers work as expressions and nest freely:

```
set !x add(!a !b)
set !x mul(add(!a !b) 2)
set !y sqrt(pow(!a 2))
set !r random(1 100)
```

Available: `add`, `sub`, `mul`, `div`, `mod`, `pow`, `sqrt`, `sin`, `cos`, `abs`, `flr`, `cel`, `random`

Other inline functions: `int(...)`, `dbl(...)`, `chr(...)`, `type(...)`, `strip(...)`, `comp(a b)`, `inp()`

### Output

```
show "Hello" !name
```

### Control Flow

```
choice !x = 10 -> 200
choice !name = "hello" -> 300
choice !x > 5 <- 3
```

`->` jumps to a label. `<-` jumps by relative offset. Comparators jump when the comparison is **false**.

### Switch

```
switch !x
case 1
show "one"
endc
case 2
show "two"
endc
ends
```

### Labels & Goto

```
1
show "label one"
goto 1
```

### Functions

```
define func greet
show "hello!"
endf

call greet
```

With arguments:

```
define func add <- a b
set !result add(@add{a} @add{b})
endf

call add <- 5 10
```

### Removal

```
del !x
del @nums[0]
del @cfg(name)
```

### File I/O

```
file txt create myfile
file txt write myfile
content here
endfw
file txt read myfile -> contents
```

### Directories

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

### Wait

```
wait 1.5
```

## Example: Counter

```
set !x 0

1
set !x add(!x 1)
show !x
choice !x > 5 -> 1
show "done"
```

## Example: Greeter

```
show "what's your name?"
set !name inp()
show "hello" strip(!name)
```

## Example: Nested Math

```
set !a 5
set !b 3
set !c 2

set !result add(mul(!a !b) div(pow(!b !c) 3))
show !result
```

`(5 * 3) + (3^2 / 3)` = `15 + 3.0` = `18.0`. Operations nest freely and resolve from the inside out.

## License

MIT
