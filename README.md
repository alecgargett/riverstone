# riverstone

Riverstone is a planned programming language that looks most similar to Lua and Julia, but with a feature-set primarily based on Gleam and Roc, and with a few unique features.

## 1. Unique Features

### Targeting Gleam and Roc

The first compiler will most likely target Gleam and the second most likely will target Roc, but compilers will hopefully eventually exist for other languages such as Kotlin, Go, F# and Python.
Likely not all programs that compile to one language will compile to all of the others.

### The "Roughstone/Shinystone" Subset Architecture

Riverstone is a superset of two subset languages: roughstone and shinystone.

Roughstone includes shinystone as a proper subset, so you can write shinystone in a .rough file but not roughstone in a .shiny file, because roughstone also includes some things that aren't allowed in shinystone, Gleam or Roc, such as top level execution in the main.rough file.

Roughstone and shinystone ideally will compile directly to the non-riverstone target language, and roughstone will also compile to shinystone. Shinystone will be designed such that Gleam and Shinystone can each transpile to the other. Ideally shinystone and at least one other target language such as Roc will each transpile to the other.

### Ambiguity-Free Block Termination

Other than returns, whitespace is not meaningful, since colons and keywords determine the beginnings and ends of blocks, but the compiler will enforce formatting.
Formatting rules are stricter for shinystone, which uses indentation for delimiting most blocks but empty lines for public and private blocks. In roughstone, these are interchangable.

#### roughstone

```roughstone
"Hello, world!"
```

```roughstone
print! "Hello, world!"
```

```roughstone
define main() ::
  print! "Hello, world!"
end main()
```

```roughstone
define main() ::

print! "Hello, world!"

end main()
```

```roughstone
public:
  define main() -> Result(Ok, Err) ::
    print! "Hello, world!"
  end main()
end public
```

#### shinystone

```shinystone
public:

define main() -> Result(Ok, Err) ::
  print! "Hello, world!"
end main()

end public
```

```shinystone
public:

define main() -> Result(Ok: string, Err: string) ::
  print! "Hello, world!"
  if Result:
    is Ok: "Result: Ok"
    is Err: "Result: Err"
end main()

end public
```

```Output
Hello, world!
```

### Types in value names

Value names must contain the type, prefixed by an underscore.
This is used for explicit type checking, and increases readability when subsequently referenced.
After type checking, these type annotations can be optionally stripped on compilation.

```riverstone
define otter_string :: "otter"
define deer_string :: "deer"
```

### Explicit Shadowing with `new`
Variable names must be suffixed with "_var" after the type name.
Declaring a new variable requires the `declare` keyword and an `<-` assignment.
Shadowing an existing variable requires the `new` keyword and an `<-` assignment.

```riverstone
declare chu_string_var <- "Pikachu"
new chu_string_var <- "Raichu"

declare count_int_var <- 0
new count_int_var <- prior count_int_var + 1 // new count_int_var == 1
```

 ### Symmetrical Indexing
Riverstone allows both symmetrical 1-based positional indexing and symmetrical 0-based vector indexing, with a clear syntax that aims to reduce the risk of off-by-one errors.

1-based positional indexing:

[1, 2, 3, ... , -3, -2, -1]

0-based vector indexing:

[start, s+1, s+2, ... , e-2, e-1, end]

The abbreviations `s` and `e` are equivalent to `start` and `end` in roughstone. In shinystone, there will be enforced standards on when to use each.

```riverstone
let list = ["a", "b", "c", "d"]

// 1-based positional access
print list[1]           // "a"
print list[4]          // "d"
print list[-1]         // "d"

// 0-based vector access
print list[start]     // "a"
print list[s + 1]     // "b"
print list[s + 2]    // "c"
print list[e - 1]     // "c"
print list[end]     // "d"

```
Inclusive slicing uses `:` and exclusive slicing uses `..`. Using these for 1-based indexing and 0-based indexing respectively is enforced, which along with the different index syntax will reduce the risk of off-by-one errors.

```riverstone
define abcd_string_list :: ["a", "b", "c", "d"]

print abcd_string_list[1:2]          // ["a", "b"]
print abcd_string_list[2:4]          // ["b", "c", "d"]
print abcd_string_list[1:-1]         // ["a", "b", "c", "d"]

print abcd_string_list[s..s+2]         // ["a", "b"]
print abcd_string_list[s+1..s+4]     // ["b", "c", "d"]
print abcd_string_list[s..e+1]         // ["a", "b", "c", "d", "e"]
```

Are there any inconsistencies? Suggested changes such as additional features?