## Statements

```ebnf
Statement =
    ExpressionStatement
  | VariableDeclaration
  | ReturnStatement
  | AssignmentStatement
  | ForLoop
  | WhileLoop
  | LoopStatement
  | LoopControl
  | StructuralBindingStatement ;
```

### Expression statements

These allow expressions to appear in a statement context.

```ebnf
(* if the expression ends with a } the semicolon can be omitted *)
ExpressionStatement = Expression, ? ";" ?;
```

For expressions with no observable side-effects this has little purpose, but
it's necessary for certain expressions such as [ifs](#if-expressions) and [calls](#calls).

```mank
println("Hello World");
1 + 3 + 1; # expression statement without any side-effects (a warning)
```

### Variable declarations

Variable declarations introduce a new variable to a scope, binding them to a name,
and giving them an initial value.

```ebnf
VariableDeclaration = identifier, OptionalTypeAnnotation, "=", Expression, ";";
```

Declarations can be type annotated, though in most cases the type is easily inferred from
the initializer.

```mank
meaning_of_life := 42;
pi: f64 = 4.0;          # explicitly type annotated
```

### Return statements

A "return" statement exits a function, optionally providing a return value.
The type of the return value must match the function declaration,
the return value can only bit omitted for functions that return `void`.


Once a "return" statement is reached, any statements after it won't be executed.

```ebnf
ReturnStatement = "return", [Expression], ";" ;
```

```mank
fun get_cool_number: i32 {
  return 1337;
  println("Yay!");  # unreachable
}
```

### Assignments

Assignments change the value of an existing [lvalue](#binding-points) or lvalues.
The type of the assigned expression must be [assignable](#assignability) to the lvalue expression's type.

```ebnf
AssignmentStatement = LValueExpression, "=", Expression, ";" ;
LValueExpression = Expression ;
```

There is a special case where mutiple lvalues can be assigned if the left-hand side
is an [lvalue tuple](#tuple-literals).

```mank
pi = 3.14;
foo[3] = 100;
dog.name = "James";
(a, b, c) = (1, 2, 3);  # lvalue tuple assignment
```

### For loops

"For" loops repeatedly execute a block for every value in a given range
(incrementing by 1 in each iteration). The range must have a numeric type.

```ebnf
ForLoop = "for", identifier, [TypeAnnotation], "in", ForRange, Block ;
ForRange = ExpressionWithoutStructs ".." ExpressionWithoutStructs ;
```

The ranges are exclusive, so `0 .. 100` is `0`, `1`, `2` .., to `99`,
sometimes written as `[0,100)`.

<!-- Loop counter + counter scope  & start/end mutations -->

```mank
# Prints "Hello!" 10 times
for i in 0 .. 10 {
  println("Hello!");
}
```

The loop body must evaluate to `void`.

### While loops

"While" loops repeatedly execute a block while a boolean condition holds.
The loop body (block) must evaluate to `void`.

```ebnf
WhileLoop = "while", WhileCondition, Block ;
WhileCondition = ExpressionWithoutStructs ;
```

```mank
# 2 to the power 5 with while loops
x := 5;
y := 1;
while x > 0 {
  y *= 2;
  x -= 1;
}
```

### Loop statements

Loop statements execute a block forever (unless they encounter a `break` or `return`).

```ebnf
LoopStatement = "loop", Block ;
```

```mank
loop {
  println("It never ends!");
}
```

These are just a convenience and could equivalently be written as:
```mank
while true {
  println("It never ends!");
}
```

### Loop control

Loop control statements allow for the early exit of a loop, or interaction of a
loop. It is not valid to use these statements outside of a loop.

```ebnf
LoopControl = ("break" | "continue"), ";" ;
```

Inside a loop:
- If a `break` is encountered the control flow will jump to the first statement after the loop
- If a `continue` is encountered the loop will skip to the next iteration
  -  If the loop is a "for" loop the loop counter will also be incremented

Both of these only apply to one loop at a time so inside a nested loop, a `break` or `continue`
will only effect the inner most loop.


Using these outside of a loop is invalid.

### Structural binding statement

A structural binding statement allows elements or fields to be extracted from pod or tuple
values, using a [structural binding](#structural-bindings).

```ebnf
StructuralBindingStatement = "bind", StructuralBinding, "=", Expression, ";" ;
```

One handy use case for this is to declare multiple variables at a time, using a
tuple literal:
```mank
bind (a, b, c) = (1, true, 3.0);
```
