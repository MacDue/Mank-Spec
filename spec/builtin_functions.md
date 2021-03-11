## Builtin functions and macros

### I/O functions

```mank
fun putchar: i32 (c: char)
```

Writes a single character `c` to `stdout` and returns the character written if successful and `-1` if not.

---

```mank
fun stderr_putchar: i32 (c: char)
```
Same as `putchar` but writes to `stderr`.

---

```mank
fun getchar: i32
```
Returns a single character read from `stdin` or `-1` if the read fails.

---

```mank
proc print(s: str)
```
Prints the string `s` to `stdout`.

---

```mank
proc println(s: str)
```
Prints the string `s` to `stdout` with a newline.

---

```mank
proc eprint(s: str)
```
Prints the string `s` to `stderr`.

---

```mank
proc eprintln(s: str)
```
Prints the string `s` to `stderr` with a newline.

---

```mank
fun input: str
```
Reads a line from `stdin` and returns it as a string (without the newline).

---

```mank
fun prompt: str (msg: str)
```
Same as `input` but, if the message (`msg`) is non-empty print it first with a trailing space.

---

```mank
fun input_int: i32
```
Attempts to return an integer read from `stdin` (using `input`), if what is read is not an integer the user is asked to try again.

---

```mank
fun prompt_int: i32 (msg: str)
```
Same as `input_int` but adds a prompt in the same way as the `prompt` function.

---

### Maths functions

All trigonometric assume their inputs are in radians.

```mank
fun sqrt: f64 (f: f64)
```
Calculates the square root of `f`.

---

```mank
fun pow: f64 (x: f64, y: f64)
```
Calculates `x` raised to the power `y`.

---

```mank
fun sin: f64 (f: f64)
```
Calculates the sine of `f`.

---

```mank
fun cos: f64 (f: f64)
```
Calculates the cosine of `f`.

---

```mank
fun tan: f64 (f: f64)
```
Calculates the tangent of `f`.

---

```mank
fun asin: f64 (f: f64)
```
Calculates the inverse sine of `f`.

---

```mank
fun atan2: f64 (x: f64, y: f64)
```
Calculates the arc tangent of the argument.

### Utility functions

```mank
fun parse_int: (i32,bool) (s: str)
```

Attempt to parse the given string as an integer. The string is assumed to
to be in base 10.


If the parse succeeds:
  - the tuple `(n, true)` is returned (where `n` is the parsed integer)
  - otherwise, `(-1, false)` is returned, the second element `false` indicates the parse failed.

---

```mank
fun int_to_string: str (i: i32)
```
Returns the (base 10) string representation of the integer `i`.

---

```mank
fun str_compare: i32 (a: str, b: str)
```
[Lexicographically compares](https://en.wikipedia.org/wiki/Lexicographic_order) two strings.

The result is negative if $a \lt b$, zero if $a = b$, and positive if $a \gt b$.

---

```mank
fun str_equal: bool (a: str, b: str)
```

Compares two strings and returns `true` if they are the same and `false` if not.

### Vector functions

Vector functions (are currently) special pseudo generic functions.
They use the syntax of generics (that are as yet unimplemented) but are internally
translated to type erased builtin functions.


For all functions other than `new_vec` the generic types can be inferred.

```
fun (T) new_vec: T[]
```
`new_vec(T)()` constructs a new empty vector ([list type](#list-types)) of type `T`.

---

```
proc (T) push_back(v: ref T[], e: T)
```
Appends an element `e` to the back (end) of a vector `v`.

---

```
proc (T) pop_back(v: ref T[])
```
Removes the last element from the vector `v`.

---

```
proc (T) fill_vec(v: ref T[], e: T, count: i32)
```
Populates the vector `v` with `count` copies of the element `e`. If the vector
is non-empty it is emptied first.


### Exceptional functions

```mank
proc abort
```
Abnormally terminates the program by raising a `SIGABRT` signal.

---

```mank
proc fail(msg: str)
```

Abort (using `abort`) the program and provide a message to print to `stderr`.

### Macros

Given a function or lambda `f`

```mank
pbind!(f,p1,p2,…,pn)
```
constructs a version of `f` with the last n parameters of bound (to fixed to a value).

---

Given a function or lambda `f`

```mank
curry!(f)
```
constructs a [curried](https://en.wikipedia.org/wiki/Currying) version of `f`.

---

The `print!` macro (and it's friends `println!`, `eprint!`, and `eprintln!`) are
used to format and print a string.


The first argument must be a [string literal](#string-literals) which is the
template, then additional parameters replace holes in the template given by `{}`.


All parameters of a print macro must be strings and the number of holes in
the template must match the number of additional parameters.

```mank
eprintln!("No file named {} in {}", filename, folder);
print!("Hello {name}!", name);  # text inside a hole is ignored
```

---

The `assert!` is used to make sure a constraint or invariant has not been broken.


It takes a boolean expression and optionally a string. If the expression
evaluates to `true` nothing happens, however, if it evaluates to `false`, the
assertion fails. If an assertion fails, the filename and line number along with the
failing expression and, if provided, the string (message) are printed to `stderr`.

```mank
assert!(1 + 1 == 2);
assert!(v.length > 0, "can't pop back on empty vector");
```

---

The `vec!` macro is used to construct a populated vector from an [array literal](#array-literals).

```mank
vec!([1,2,3,4])
vec!(["the", "cat", "sat", "on", "the", "mat"])
```

<br/>
<div style="text-align:center ">
...and they all lived happily ever after!<br/>
<b>The End.</b>
</div>

