Riverstone is a planned programming language that looks similar to Lua or Julia but with a feature-set based on Gleam and Roc, and a few unique features.

It is still early in the syntax design stage and does not have a compiler.

## 1. Unique Features

### Targeting Gleam and Roc

The first compiler will most likely target Gleam and the second most likely will target Roc, but compilers will hopefully eventually exist for binary, Kotlin, F#, at least one low-level language and at least one scripting language (Lua or Python). Likely not all programs that compile to one language will compile to all of the others.

### The "Roughstone/Shinystone" Subset Architecture

Riverstone is a superset of two subset languages: roughstone and shinystone.

Roughstone includes shinystone as a proper subset, so you can write shinystone in a `.rough` file but not roughstone in a `.shiny` file, because roughstone also includes some things that aren't allowed in shinystone, Gleam or Roc, such as top level execution in the `main.rough` file.

#### roughstone

```roughstone
"Hello, world!"
```

```roughstone
print! "Hello, world!"
```

```roughstone
"Hello, world!" |> print!
```

### Output
```output
Hello, World!
```

Roughstone and shinystone ideally will compile directly to the non-riverstone target language, and roughstone will also compile to shinystone. Shinystone will be designed such that Gleam and Shinystone can each transpile to the other. Ideally shinystone and at least one other target language such as Roc will each transpile to the other.

### Readibility via both keywords and symbols

Defining any global constant, including a function, uses both the "define" keyword and the "::" operator for easier readbility than other language protocols.

#### roughstone

```roughstone
define otter :: "otter"
define deer :: "deer"
```

### Types in value names

Value names (excluding function names) in shinystone must contain the type, prefixed by an underscore. This is used for explicit type checking, and increases readability when subsequently referenced. After type checking, these type annotations can be optionally stripped on compilation.

#### shinystone

```shinystone
define otter_string :: "otter"
define deer_string :: "deer"
```

It is strongly encouraged to do this in roughstone too.

### Explicit variable declaration, naming and shadowing with `declare`, `_var`, `new` and `<-`

*   Declaring a new variable requires the `declare` keyword and an `<-` assignment.
*   Shadowing an existing variable requires the `new` keyword and an `<-` assignment.

#### roughstone

```roughstone
declare chu_string <- "Pikachu"
new chu_string <- "Raichu"

declare count_int <- 0
new count_int <- prior count_int + 1 // new count_int == 1
```

In shinystone, variable names must be suffixed with `_var` after the type name.

#### shinystone

```shinystone
declare chu_string_var <- "Pikachu"
new chu_string_var <- "Raichu"

declare count_int_var <- 0
new count_int_var <- prior count_int_var + 1 // new count_int_var == 1
```

Using `<-` reduces ambiguity and reduces operator overloading.

### Local Constants with `let` and `<:`

While `declare` and `new` handle variables that can be shadowed, Riverstone offers `let` for defining strictly immutable local constants. These are bound using the `<:` operator.

Unlike variables declared with `<-`, a local constant defined with `<:` cannot be shadowed or redeclared within the same scope.

#### roughstone

```roughstone
let pi_float <: 3.14159
```

### Ambiguity-Free Block Termination

Other than returns, whitespace is not meaningful, since colons and keywords determine the beginnings and ends of blocks, but the compiler will enforce formatting.

Formatting rules are stricter for shinystone, which uses indentation for delimiting most blocks but **empty lines** for public and private blocks. In roughstone, these are interchangable.

In shinystone, `public:` blocks are not indented; instead, they are separated by empty lines, and must include an explicit `end public` (also separated by an empty line).

**Roughstone Examples:**

#### roughstone

```roughstone
define main() ::
  print! "Hello, world!"
end main()
```

```roughstone
public:
  define main() :: Status(Ok, Err)
    "Hello, world!" |> print!
  end main()
end public
```

```output
Hello, World!
```

**Shinystone Examples:**

```shinystone
public:

define main() :: Status(Ok, Err)
  "Hello, world!" |> print!
end main()

end public
```

```shinystone
public:

define main() :: Status(Ok: string, Err: string)
  print! "Hello, world!"
  if Status is:
    Ok: "Status: Ok"
    Err: "Status: Err"
  end if
end main()

end public
```

```output
Hello, World!
```

### Symmetrical Indexing

Riverstone allows both symmetrical 1-based positional indexing and symmetrical 0-based vector indexing, with a clear syntax that aims to reduce the risk of off-by-one errors.

*   **1-based positional indexing:** `[1, 2, 3, ... , -3, -2, -1]`
*   **0-based vector indexing:** `[s, s+1, s+2, ... , e-2, e-1, e]`

The abbreviations `s` and `e` are short for "start" and "end" and are equivalent to `1` and `length(list_name)` under the hood, so effectively one-based indexing is being used under the hood.

```riverstone
declare list_string_var <- ["a", "b", "c", "d"]

// 1-based positional access
print! list_string_var[1]          // "a"
print! list_string_var[4]          // "d"
print! list_string_var[-1]         // "d"

// 0-based vector access
print! list_string_var[s]      // "a"
print! list_string_var[s + 1]      // "b"
print! list_string_var[s + 2]      // "c"
print! list_string_var[e - 1]      // "c"
print! list_string_var[e]        // "d"
```

### Slicing
Inclusive slicing uses `:` and exclusive slicing uses `..`. Using these for 1-based indexing and 0-based indexing respectively is enforced, which along with the different index syntax will reduce the risk of off-by-one errors.

```riverstone
define abcd_stringlist :: ["a", "b", "c", "d"]

print! abcd_stringlist[1:2]          // ["a", "b"]
print! abcd_stringlist[2:4]          // ["b", "c", "d"]
print! abcd_stringlist[1:-1]         // ["a", "b", "c", "d"]

print! abcd_stringlist[s..s+2]       // ["a", "b"]
print! abcd_stringlist[s+1..s+4]     // ["b", "c", "d"]
print! abcd_stringlist[s..e+1]       // ["a", "b", "c", "d"]
```

## 2. Other Differences with Gleam

### Unified Arithmetic Operators

Unlike in Gleam, operator overloading exists in riverstone, but it is restricted to doing math with `+`, `-`, and `*` operators, which are used for both integers and floats. Mixing `int` and `float` types in a single operation remains prohibited.

```riverstone
declare i_int_var <- 1 + 2
declare f_float_var <- 1.0 + 2.0
```

### Elixir-style string concatenation and pattern matching in function heads

#### roughstone

```roughstone
define execute("echo " <> message_string) ::
  print! "Echoing: " <> message_string
end execute()

define execute("warn " <> warning_text_string) ::
  print! "!!! WARNING: " <> warning_text_string <> " !!!"
end execute()
```

### Automatic Import Management

Riverstone hides target-specific import statements. The compiler identifies and injects necessary native modules based on usage.

```riverstone
public:

define main() ::
  string.reverse("gnirts") |> print!
end main()

end public
```

## 3. Shared Features with Gleam

### Immutability

All data structures are immutable. When you use `new` to shadow a variable, you are creating a new binding, not mutating the underlying data.

```riverstone
define original_intlist :: [1, 2, 3]
define extended_intlist :: [0, ..original_intlist]
```

### Static Typing

Riverstone is statically typed and requires homogeneous lists. The type-in-name requirement helps the developer track these types visually.

```riverstone
declare names_stringlist_var <- ["Alice", "Bob"]
// new names_stringlist_var <- [1, ..prior names_stringlist_var] (This will not compile)
```

### Safety

The language uses `Option(Some, None)` and `Status(Ok, Err)` (Result) types to handle errors and the absence of values. Pattern matching is used to safely extract values.

```riverstone
public:

define divide(numerator_float, denominator_float) :: Status(Ok: int, Err: string)
  if denominator_int is:
    0: Status(Err: "Division by zero")
    _: Status(Ok: numerator_float / denominator_float)
  end if
end divide()

define main() :: Status(Ok: int, Err: string)
  declare calculation_float_var <- divide(10, 2)
  
  if calculation_float_var is:
    Ok: print! "Success!"
    Err: print! "Error!"
  end if
end main()

end public
```