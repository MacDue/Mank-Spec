## Structural bindings

Structural bindings are a way of binding names to the elements/fields (nested or otherwise) of an initializer.
They are used within the [structural binding statement](#structural-binding-statement)
and [switch expressions](#switch-expressions).

```ebnf
StructuralBinding = TupleBindings | PodBindings ;
TupleBindings = "(", [TupleBindingList], ")" ;
TupleBindingList = TupleBind, {",", TupleBind}, [","] ;
TupleBind = Bind | StructuralBinding ;
PodBindings = "{", [PodBindingList], "}" ;
PodBindingList = PodBind, {",", PodBind}, [","] ;
PodBind = ".", identifier, ([ "/", (Bind |  StructuralBinding) ] | TypeAnnotation);
Bind = identifier, [TypeAnnotation] ;
```

Given these definitions and an instance of `Example`
```mank
pod Other {
  apple: f64
}

pod Example {
  foo: i32,
  bar: f64,
  nested: Other,
  tuple: (f64, bool, str)
}
```

the following are all valid bindings:
```mank
# binds foo and bar to the fields with the same name
{.foo, .bar}
# the same with explicit types
{.foo:i32, .bar:f64}
# binds the fields foo and bar, but renames them to my_foo and my_bar
{.foo/my_foo, .bar/my_bar:f64}
# binds foo (like before), but also binds the nested field apple
{.foo, .nested/{.apple}}
# binds bar (like before), but also the nested tuple elements as a, b, and c
{.bar, .tuple/(a, b, c)}
```
(in this context, to "bind" a field/element means to create a local variable for it)

Given this tuple (where `example` is an instance of `Example`)
```mank
(example, (1, true), 5.0)
```

the following are all valid bindings:
```mank
# binds the elements of the tuple to my_example, tuple, and num
(my_example, tuple, num)
# binds the first element to a the nested tuple elements to b and c, and
# then the final element to d
(a, (b, c), d)
# uses a pod binding on the first element, then binds the remaining elements
({.foo/a}, b, c)
# binds the elements to a, b, and c with type annotations on a and c
(a: Example, b, c: f64)
```
Note that unlike pod bindings (where fields can be skipped and reordered), tuple bindings must have the same shape as the initializing tuple.


Bindings can also take references to fields/elements of initializers if they are [lvalues](#binding-points),
by annotating the binding with a reference type:
```mank
# bind my_exampe as a reference (reference type inferred)
# also binds c to a reference (full reference type given)
(my_exampe: ref, b, c: ref f64)
```

<!-- <div class="page"/> -->

## Expressions

Expressions are loosely defined as anything that can yield a value. In Mank,
this can be simple constants, variables, or mathematical formulas, as in most languages,
but can also incorporate control low such as in if and switch expressions.

Unless otherwise stated, all expressions are assumed to be [rvalues](#binding-points).


Note that in expressions that require certain types, references to those types are also valid (references are transparent).

### Expression groups

Expressions are split into several categories based on where they can occur and their precedence.

```ebnf
Expression = BinaryOperation ;
PostfixExpression = PrimaryExpression, {Call, IndexAccess, FieldAccess} ;

PrimaryExpression =
    Path
  | Identifier
  | MarcoIdentifier
  | SpecializedIdentifier
  | PrimitiveLiteral
  | ParenthesisedExpression
  | Block
  | PodLiteral
  | TupleLiteral
  | ArrayLiteral
  | LambdaExpression
  | IfExpression
  | SwitchExpression ;
ExpressionWithoutStructs = (*
  same as Expression but cannot include struct literals at the top level *) ;
ExpressionWithoutCallsOrStructs = (*
  same as Expression but cannot include calls or struct literals
  at the top level *) ;
```

`PrimitiveLiteral` expressions are any literal for a primitive type (including strings)
given in the [lexical elements](#lexical-elements) section.

### Paths and identifiers

Paths and identifier expressions are used to refer to entities within a scope.

```ebnf
Identifier = identifier ;
MarcoIdentifier = Identifier, "!" ;
SpecializedIdentifier = Identifier, Specializations ;
Specializations = "@", "(", TypeList, ")" ;

(* >= 1 sections e.g. a::b -- a alone normally is an identifier*)
Path = Identifier, {"::", Identifier}, [Specializations] ;
```

- Identifiers can refer to variables, global constants, and functions.
- Macro identifiers are used to call [builtin macros](#builtin-functions-and-macros).
- Specialized identifiers are used to specialize generic functions or types (in cases where the specializations cannot be inferred).
- Paths are used to access entities within a named scope.
  - Currently, paths are only used to access enum members.

```mank
foo_bar        # identifier
println!       # marco identifier
new_vec@(i32)  # specialized identifier
Foo::Bar       # path
```

### Calls

Given an expression `f` with a callable type `F` ([lambda](#lambda-types) or [function](#functionprocedure-declarations)),

```mank
f(arg1, arg2, … argn)
```

calls `f` with arguments `arg1, arg2, … argn`. The arguments must be [assignable](#assignability) to the parameter types of `F`. Arguments are evaluated in order (left to right) before the function is called. The type of the expression is the return type of `F`, and its value is passed back to the caller when the function returns.


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
StatementList = { Statement [";"] } ;
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
IfExpression = "if", IfCondition, Block, ["else", (IfExpression | Block)] ;
IfCondition = ExpressionWithoutStructs ;
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

```ebnf
UnaryOperation = {unary_op}, PostfixExpression ;
```

| Operator (`unary_op`) | Meaning         | Definition (given `x`)        | Applicable types  |
|-----------------------|-----------------|-------------------------------|-------------------|
| `+`                   | plus            | `0 + x`                       | integers, floats  |
| `-`                   | minus           | `0 - x`                       | integers, floats  |
| `~`                   | bitwise not     | each bit of `x` flipped       | integers          |
| `¬` or `!`            | logical not     | `if x { false } else { true }`| booleans          |
| `ref`                 | take reference  | if `x` is an [lvalue](#binding-points) take a reference to `x`  | any type |
`x` is any applicable type for the operator.


Note that unary operators can be repeated and mixed:
```mank
x := -1;    # single unary minus
y := -+-x;  # mutiple unary operations -(+(-(x)))
```

### Binary operations

```ebnf
BinaryOperation = (Expression, binary_op, Expression)
                | (Expression, "as", Type)
                | UnaryOperation ;
```
The second form of binary expression is an [as cast](#as-casts), and is described separately.

<!-- <div class="page"/> -->

| Operator (`binary_op`) | Meaning                 | Definition (given `x`, `y`)          | Applicable types  |
|-----------------------|-------------------------|--------------------------------------|-------------------|
| `+`                   | add                     | `value_of(x)` $+$ `value_of(y)`      | integers, floats, strings  |
| `-`                   | minus                   | `value_of(x)` $-$ ` value_of(y)`     | integers, floats           |
| `*`                   | times                   | `value_of(x)` $\times$ `value_of(y)` | integers, floats           |
| `/`                   | divide                  | `value_of(x)` $\div$ `value_of(y)`   | integers, floats           |
| `%`                   | modulo                  | integer remainder of `x / y`         | integers
| `<<`                  | left shift              | logical left shift of `value_of(x)` by `value_of(y)`          | integers         |
| `>>`                  | right shift             | arithmetic right shift of `value_of(x)` by `value_of(y)`      | integers         |
| `<`                   | less than               | `true` if `value_of(x)` $\lt$ `value_of(y)` otherwise `false` | integers, floats |
| `>`                   | greater than            | `true` if `value_of(x)` $\gt$ `value_of(y)` otherwise `false` |  integers, floats          |
| `<=`                  | less than or equal      | `¬(x > y)`                           |  integers, floats          |
| `>=`                  | greater than or equal   | `¬(x < y)`                           |  integers, floats          |
| `==`                  | equal to                | `(x >= y) && (x <= y)`               |  integers, floats          |
| `!=`                  | not equal to            | `¬(x == y)`                          |  integers, floats          |
| `&`                   | bitwise and             | `0` at all bit positions where `x` and `y` differ (keep bit otherwise)   | integers |
| `|!`                  | bitwise xor             | `1` at all bit positions where `x` and `y` differ (zero otherwise)       | integers |
| `|`                   | bitwise or              | `1` at all bit positions where at least of one `x` and `y` contains a `1` | integers |
| `&&`                  | logical and             | `if x { y } else { false }`          |  booleans                  |
| `||`                  | logical or              | `if x { true } else { y }`           |  booleans                  |
`value_of` gives the result of evaluating an expression.


Note:
  - for strings: `x + y` is the string concatenation of `x` and `y`.
  - for integers: `x / y` is the integer division (truncated towards zero).
<!-- <div class="page"/> -->

#### Operator precedence
<table style="margin-left: auto; margin-right: auto;">
   <thead>
      <tr>
         <th>Precedence</th>
         <th>Operator</th>
      </tr>
   </thead>
   <tbody>
      <tr>
         <td>100</td>
         <td><code>*</code>, <code>/</code>, <code>%</code></td>
      </tr>
      <tr>
         <td>90</td>
         <td><code>+</code>, <code>-</code></td>
      </tr>
      <tr>
         <td>80</td>
         <td><code>&lt;&lt;</code>, <code>&gt;&gt;</code></td>
      </tr>
      <tr>
         <td>70</td>
         <td><code>&lt;</code>, <code>&lt;=</code>, <code>&gt;</code>, <code>&gt;=</code></td>
      </tr>
      <tr>
         <td>60</td>
         <td><code>==</code>, <code>!=</code></td>
      </tr>
      <tr>
         <td>50</td>
         <td><code>&amp;</code></td>
      </tr>
      <tr>
         <td>40</td>
         <td><code>|!</code></td>
      </tr>
      <tr>
         <td>30</td>
         <td><code>|</code></td>
      </tr>
      <tr>
         <td>20</td>
         <td><code>&amp;&amp;</code></td>
      </tr>
      <tr>
         <td>10</td>
         <td><code>||</code></td>
      </tr>
   </tbody>
</table>

Note that postfix expressions always have higher precedence than unary operations,
which have always have higher precedence than binary operations. However, unlike
binary operations among unary and postfix expressions, all operations/expressions
have equal precedence.

### Index accesses

Given `a` with an indexable type (fixed-size array, list, or string)
```mank
a[idx]
```
accesses the element of `a` at the index `idx`.


`idx` must be an integer type and if `idx` is out of range at runtime, the program is aborted with an index error.


If `a` is a [fixed-size array](#fixed-size-array-types) or [list type](#list-types):
  - the type of `a[idx]` is the element type of the array
  - if `a` is an [lvalue](#binding-points), `a[idx]` is an lvalue

Additionally, if `a` is a [fixed-size array](#fixed-size-array-types):
  - constant indexes must be in range

If `a` is a [string type](#string-types):
  - the type of `a[idx]` is always `char`

### Field accesses

Given an expression `x`
```mank
x.f
```
accesses the field of the value `x` given by `f`, where `f` is an [identifier](#identifiers).

If `x` is a [fixed-size array](#fixed-size-array-types), [list type](#list-types), or [string](#string-types):
  - `x.length` gives the length as an integer

If `x` is an [enum type](#enum-types):
  - `x.tag` gives the unique tag of the enum member x contains as an integer

If `x` is a [pod type](#pod-types):
  - `x.<field>` accesses a field as defined the pod's [declaration](#pod-declarations), the access type is the type of the declared field
  - if `x` is an [lvalue](#binding-points), `x.f` is an lvalue

Accessing non-existent fields is an error.

### Tuple literals

A tuple literal constructs a value of a [tuple type](#tuple-types).

```ebnf
TupleLiteral = ("(", [TupleElements], ")") ;
TupleElements = Expression, ",", [Expression, {"," Expression}, [","]] ;
```

The types of the expressions in the tuple literal are used to derive the tuple type:

```mank
(1, true, 3.0)   # three element tuple -- (i32, bool, f64)
(1, true, 3.0,)  # tuple with trailing comma -- (i32, bool, f64)
((1,3), false)   # nested tuple literal -- ((i32, i32), bool)
(42,)            # single element tuple -- (i32)
(42)             # parenthesised expression (not a tuple)
```

If all members of a tuple are [lvalues](#binding-points) then the tuple becomes a special lvalue. If used on the left-hand side of an assignment of a tuple with matching types, assigns each lvalue to the corresponding value of the tuple.

<!-- <div class="page"/> -->

```mank
a := 1;
b := 2;
c := 3;
# a = 1, b = 2, c = 3

(a, b, c) = (100, 200, 300);
# a = 100, b = 200, c = 300
```

### Pod literals

A pod literal constructs a value for a [pod type](#pod-types) or pod [enum member](#enum-types).

```ebnf
PodLiteral = Path, "{", [FieldInitializerList], "}" ;
FieldInitializerList = FieldInitializer, { ",", FieldInitializer } ;
FieldInitializer = ".", Identifier, "=", Expression ;
```
In a pod literal the order of fields does not matter, however, a pod literal is invalid
if it does not provide all the fields from the pod declaration or repeats fields.
Additionally, the type of each field initializer must match the type of the field in
the pod declaration.


Given the following declarations:

```mank
pod Dog {
  age: i32,
  name: str,
  favorite_toy: DogToy
}

enum DogToy {
  Stick,
  Teddy{ name: str }
}
```

these are valid instantiations:
```mank
harrold := DogToy::Teddy { .name = "Harrold" };
cassie := Dog { .age = 10, .favorite_toy = harrold, .name = "Cassie" };
woody := Dog { .name "Woody", .age = 6, .favorite_toy = DogToy::Stick };
```

### Array literals

An array literal construct a value of a [fixed-size array type](#fixed-size-array-types).

```ebnf
ArrayLiteral = "[", [ExpressionList], "]" ;
ExpressionList = Expression { ",", Expression } ;

ArrayRepeatLiteral = "[", "=", Expression, ";", Expression, "]" ;

```

There are two types of array literals:

- Normal list literals
  - For these, there must be at least one element in the array literal, and all elements must have matching types
- Repeat literals
  - These say how many times to repeat (given by the second expression -- which must be constant) an initializer (the first expression)

The type of the array literal is a fixed-size array of the element type, with the size being the number of elements in the literal:

```mank
a := [];                    # invalid empty literal (unknown type)
a := [1,2,3,4,5];           # i32[5]
a := [[1.1,2.1],[1.2,2.2]]; # f64[2,2]
a := [1,2,true,"hello"];    # invalid literal (mixed element types)
a := [=0; 10];              # i32[10] (repeat literal)
```

### Lambda expressions

A lambda expression creates a lambda function with and has a [lambda type](#lambda-types).

```ebnf
LambdaExpression = "\", [LambdaParameterList], "->", [Return], Block ;
LambdaParameterList = {identifier, [TypeAnnotation]} ;
Return = TypeAnnotation ;
```

Type annotations for lambda parameters and return types are optional and can be (in most cases),
inferred based on the context and usage of the lambda. If the types cannot be inferred the lambda is invalid.


The type of the lambda is a lambda type, with the corresponding parameter and return types to the lambda expression.

```mank
add := \x, y -> { x + y };  # untyped lambda that adds to values
result := add(1,2);         # inferred to be \i32, i32 -> i32 based on usage

sub := \x: f64, y: f64 -> f64 { x - y }; # explicitly type annotated lambda
```

#### Closures/captures

A lambda can form a closure if it captures its variables/values from its surrounding (lexical)
environment. Values are captured [by value](#type-storage) (contrary to by reference), though existing references in the outer scope can be captured (though this can lead to dangling references if the lambda is returned from a function).


The environment of a lambda with captures is heap-allocated and lives as long as the lambda.

Some simple closures:

```mank
language_name := "Mank";
display_name := \ -> {
  println!("The {name} Programming Language", language_name);
}
display_name();
```
The above demonstrates a simple capture, the variable `language_name` is
declared in the scope outside of the lambda (note that there are no declarations within the lambda).
However, the lambda captures the variable from its outer scope, making it able to print it in its body.


Running this snippet will result in: ``The Mank Programming Language`` being printed.


```mank
make_adder := \x -> { \y -> { x + y } };
add_10 := make_adder(10);
fifteen := add_10(5); # = 15
```

This is another example that demonstrates closures and returning lambdas from functions.
When `make_adder` is called it returns its nested lambda (`\y -> { x + y }`),
which captures the value of `x`. Then when `add_10` is called, the captured x is added to the
parameter `y` to result in 15.

```mank
fun make_counter: \ -> i32 (start: i32) {
  count := start - 1;
  \ -> i32 {
    count += 1;
    count
  }
}
```

```mank
counter := make_counter(4);
counter() # = 4
counter() # = 5
counter() # = 6
```

Lambdas can also mutate their captured variables, as shown with the simple counter above.

### As casts

An "as cast" converts values between types. They are the only way to convert between types as there are no implicit conversions.


They are a special form of [binary operation](#binary-operations).


Allowed (base) casts:
| Source type | Target type   | Notes         |
|-------------|---------------|---------------|
|   `T`       |   `T`         | if the source and target types are the same the cast is a nop |
|  `i32`      |  `f32`        |  |
|  `i32`      |  `f64`        |  |
| `f32`       | `i32`         | fractional part discarded (round down) |
| `f64`       | `i32`         | fractional part discarded (round down) |
| `i32`       | `char`        | if the integer does not fit within the char the top bits are truncated |
| `char`      | `i32`         | the integer is the characters' ASCII value       |
| `bool`      | `i32`         | 1 if the bool is true 0 otherwise                |
| `char`      | `str`         | creates a new string consisting of a single char |

Additional casts are allowable due to the transitive (directed) closure of these base casts.
For example, the cast `true as f64` is allowed as you can cast a `bool` to an `i32` then to an `f64`.
A cast like `1.2 as bool` would be invalid (since there are no casts from any type to `bool`).

### Switch expressions

"Switch" expressions select a block to execute based on which case a value matches.

```ebnf
SwitchExpression = "switch", SwitchedExpression, "{", [SwitchCaseList] "}" ;
SwitchCaseList = SwitchCase, { "," SwitchCase }, [","] ;
SwitchCase = (MatchExpression, [StructuralBindings]) | "else", "=>", Block ;
MatchExpression = ExpressionWithoutCallsOrStructs ;
SwitchedExpression = ExpressionWithoutStructs ;
```

"Switch" expressions can be used on integer types along with enums. When used with
an integer type the match expression for each case must be a constant expression.

```mank
# simple switch against an integer:
switch number => {
  1 => { println("one!"); },
  2 => { println("two!"); },
  1 + 2 => { println("three!"); },
  else => { println("something else!"); }
}
```

```mank
# AST for simple s-expressions (+ 1 2 3 4 (* 5 6 4))
enum Expr {
  Number(f64),
  Add { operands: Expr[] },
  Mult { operands: Expr[] }
}
```

When used with enums each case can (optionally) include a [structural binding](#structural-bindings)
to extract pod or tuple data from the enum member.

<!-- <div class="page"/> -->

```mank
fun eval_expr: f64 (expr: Expr) {
  switch expr {
    Expr::Number(num) => { num },
    Expr::Add { .operands } => {
      sum := 0.;
      for i in 0 .. operands.length {
        sum += eval_expr(operands[i]);
      }
      sum
    },
    Expr::Mult { .operands } => {
      product := 1.;
      for i in 0 .. operands.length {
        product *= eval_expr(operands[i]);
      }
      product
    }
  }
}
```
The above shows a simple example of a (tagged union) enum being used to represent some [S-expressions](https://en.wikipedia.org/wiki/S-expression), which can
then be evaluated recursively with a `switch`.


In a similar fashion to `if` expressions, if a switch is exhaustive (covers all possible cases), then it can evaluate to a value. The type of the switch comes from the type of all the cases (blocks) in the switch, all of which must have matching types.
