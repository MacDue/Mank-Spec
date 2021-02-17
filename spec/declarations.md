## Declarations and constructs

```ebnf
TypeAnnotation = ":", Type ;
OptionalTypeAnnotation = ":", [Type] ;
```
All the declarations in this section appear at the top level of a file (not within functions).


When resolving names the order of declarations **does not** matter.

### Constant declarations

Constant declarations bind a [name](#identifiers) a compile time constant. These are global and can be used between functions.


Constants can refer to other constants (that are possibly declared later) as long as it's not recursive.


```ebnf
ConstDecl = "const", OptionalTypeAnnotation, "=", ConstantExpression ;
ConstantExpression = Expression ; (* constness checked in semantics *)
```

```mank
# constant with explict type
const WITH: i32 = 10000;
# constant with inferred type
const HEIGHT := 4000;

# constant using constants defined later
const PROGRAM_VERSION := "version-" + PROGRAM_MAJOR + "." + PROGRAM_MINOR;
const PROGRAM_MAJOR := "123";
const PROGRAM_MINOR := "21";

# invalid (recursive)
const INVALID := INVALID + 1;
```

### Function/procedure declarations

Function declarations bind a [name](#identifiers) to a function.
Mank supports serveral types of functions: functions (`fun`) which return a value, procedures (`proc`) that always return void, and `test` functions.


"`test`" functions are a special case, and are used by the built-in test runner.
If not building with the `--tests` flag they are not compiled.

```ebnf
FunctionDecl = FunctionHeader, Block ;
FunctionHeader = ("fun", identifier, Return, [ParameterList])
               | ("proc", identifier, [ParameterList])
               | ("test", identifier) ;
Return = TypeAnnotation ;
ParameterList = "(", {identifier, TypeAnnotation}, ")" ;
```

```mank
fun max: i32 (a: i32, b: i32) {
  if a > b { a } else { b}
}

# the parentheses for the arguments can be omitted there are none
fun get_version: str {
  "Mank v0.1"
}

proc main (args: str) {
  println!("Hello {name}", args[1]);
}

# invalid! procedures don't return a value
proc add: i32 (a: i32, b: i32) {
  a + b
}

# tests never can have returns or paramters
test that_addtion_works {
  assert!(1 + 1 == 2);
}
```

### Pod declarations

Pod declarations declare a new pod type, shown in the [pod types](#pod-types) section.
