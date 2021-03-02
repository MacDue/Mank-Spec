## Expressions

Expressions are loosely defined as anything that can yield a value. In Mank
this can be simple constants, variables, or mathematical formulas, as in most languages,
but can also incorporate control low such as in if and switch expressions.


Unless otherwise stated, all expressions are assumed to be [rvalues](#binding-points).

### Operands/identifiers

???

### Calls

Given a expression `f` with a callable type `F` ([lambda](#lambda-types) or [function](#functionprocedure-declarations)),

```mank
f(arg1, arg2, … argn)
```

calls `f` with arguments `arg1, arg2, … argn`. The arguments must be [assignable](#assignability) to the parameter types of `F`. Arguments are evaluated in order (left to right) before the function is called. The type of the expression is the return type of `F`, and it's value is passed back to the caller when the function returns.


If the return type of `F` is a [reference type](#reference-types), then the expression can be used as an lvalue.

```mank
fun index_cats: ref Cat (cats: Cat[], index: i32) {
  cats[index]
}

proc add_angus (my_cats: Cat[], index: i32) {
  # call as lvalue (index_cats returns a reference to Cat)
  index_cats(my_cats, index) = Cat { .name = "Angus", .age = 5 };

  # normal rvalue call (here returning void -- empty tuple)
  print("Added Angus");
}
```

### Blocks

A block is a possibly empty sequence of statements, optionally ending with an expression.

```ebnf
Block = "{", [StatementList], [Expression], "}" ;
StatementList = { Statement ";" } ;
```

If a block ends with an expression, the block has the type and value of that expression,
otherwise, the block is void (with the value the empty tuple).


```mank
{
  x := 10;
  x -= 1;
  x + 1     # ending expression (block has the value 10)
}
```
<!-- // TODO: scopes -->

### If expressions

"If" expressions conditionally select a block to execute based on a boolean expression.
If the expression is `true` the "then" block (first block) is executed, otherwise, if given, the "else" block is executed.


```ebnf
IfExpr = "if", Expression, Block, ["else", (IfExpr | Block)] ;
```

If both branches are supplied, they both must evaluate to the same type (which can be void). Then the "if" expression will take the value of the selected block.

```mank
fun bool_to_str: str (b: bool) {
  # evaluates to the string "true" or "false" depending on b
  if b { "true" } else { "false" }
}
```

If no "else" branch is given then the "then" block must evaluate to void.

### Unary operations

### Binary operations

### Field accesses

### Index accesses

### Tuple literals

### Pod literals

### Array litetals

### Lambda expressions

### As casts

### Switch expressions
