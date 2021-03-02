## Declarations and constructs

```ebnf
TypeAnnotation = ":", Type ;
OptionalTypeAnnotation = ":", [Type] ;
```
All the declarations in this section appear at the top level of a file (not within functions).


When resolving names the order of declarations **does not** matter (e.g. a function can refer to a constant declared after it).


For each category of declaration (function, pod, constant, etc) all declarations within a module must have
a unique name, otherwise, you get a `redeclaration error`. Shadowing is allowed between declarations of different types (as a warning), but does not serve much purpose due to the out-of-order name resolution meaning only the last declaration will be used (at any point).

### Constant declarations

Constant declarations bind a [name](#identifiers) a compile-time constant. These are global and can be used between functions.


Constants can refer to other constants (that are possibly declared later) as long as it's not recursive definition.


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
Mank supports several types of functions: functions (`fun`) which return a value, procedures (`proc`) that always return void, and `test` functions.

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

Top-level functions are not strictly first-class functions, though they will be coerced to lambdas where needed (e.g. if you try to pass top-level function as an argument to a function accepting a lambda).
This is done by automatically wrapping the function within a [lambda expression](#lambda-expressions) with a matching signature (same return and parameter types).


It is not possible to nest top-level functions to create closures/non-local access, but this is possible
with [lambdas](#lambda-expressions).


A function is free to mutate any local variables, or parameters passed to it (though unless these are references or heap-allocated data like [lists](#list-types) this is not be
observable outside the function).


If a function body ends with an expression (without a semicolon after it), then that expression is returned from the function. Values can also be returned early, or explictly with [return statements](#return-statements).

```mank
fun max: i32 (a: i32, b: i32) {
  if a > b { a } else { b}
}

# the parentheses for the arguments can be omitted there are none
fun get_version: str {
  "Mank v0.1"   # implict return: final expression (no semicolon)
}

fun compareInt: i32 (a: i32, b: i32) {
  return a - b; # explict return statement
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

Pod declarations bind a [name](#identifiers) to a [pod type](#pod-types).

Each field name within the pod must be unique.
Recursive pod declarations are invalid (as they lead to a pod of infinite size).

```ebnf
PodDecl = "pod", PodName, "{", [Fields], "}" ;
PodName = identifier ;
Fields = FieldDecl, { ",", FieldDecl }, [","] ;
FieldDecl = identifier, ":", Type ;
```

```mank
# An empty pod
pod Empty {}

# Pod called "MyPod" with 3 fields (foo, bar, baz)
pod MyPod {
  foo: i32,
  bar: bool[5],
  baz: ref f64
}

# Invalid recurisve definition
pod Recurisve {
  foo: Recurisve;
}

# This is allowed due to the indirection from the reference
pod Recurisve {
  foo: ref Recurisve;
}
```

### Enum declarations

Enum declarations bind a [name](#identifiers) to a [enum type](#enum-types).


Each member within the enum must have a unique name. As with pods, recursive enums
are invalid (e.g. using the enum being declared in the pod, or tuple data of a member).
If a member has pod data then that member's declaration must follow the same rules as a normal [pod declaration](#pod-declarations).

```ebnf
EnumDecl = "enum", EnumName, "{", [Members], "}" ;
EnumName = identifier ;
Members = MemberDecl, { ",", MemberDecl }, [","] ;
MemberDecl = identifier, [MemberData] ;
MemberData = TupleData | PodData;
TupleData = TupleType ; (* same as the tuple type *)
PodData = Fields ; (* same as the pod declaration fields *)
```

```mank
# a simple C-like enum of colours
enum Colour {
  RED,
  GREEN,
  BLUE
}

# a tagged union of input events
enum Event {
  Keypress(char),                 # member with tuple like data,
  MouseClick { x: i32, y: i32 },  # with pod like data,
  Quit                            # and without any associated data
}
```
