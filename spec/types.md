## Types

```ebnf
Type = ReferenceType | BaseType ;
BaseType = TypeName | LambdaType | TupleType | FixedSizeArrayType | ListType ;

TypeList = Type, { ",", Type }, [","] ;
```

`TypeName` covers any type referred to by name, such as primitive types and pod types.
Other types are constructed from simpler types, e.g. a tuple type could be `(i32, bool)`.

### Boolean types

Booleans represented by the `bool` type and can take the value `true` or `false`
(which are built-in keyword constants).

### Numeric types

Numeric types are floating-point or integer types (which also includes `char`).

```mank
char  # set of all signed 8-bit integers (that use character literals)
i32   # set of all signed 32-bit integers
i64   # set of all signed 64-bit integers

f32   # set of all 32-bit floating-point numbers
f64   # set of all 64-bit floating-point numbers
```

More numeric types are planned but are currently unimplemented.

### Reference types

A reference is a type that holds a pointer to a value of another type (the referenced type).
References cannot be null.


In some cases (such as in lambda expressions, or bind statements), the referenced type can be omitted and inferred based on context.

```ebnf
ReferenceType = "ref", [ReferencedType] ;
ReferencedType = BaseType ;
```

```mank
ref i32
ref bool[10]
```

### Fixed-size array types

Fixed-size arrays hold a fixed number (the length) of elements of another type (the element type). The length cannot be a negative number, the length of a fixed-sized array can be obtained from the `.length` attribute.


Fixed-size arrays can have multiple dimensions, given by a comma-separated list of dimensions.
In outermost dimension is given first, preceding nested dimensions.

```ebnf
FixedSizeArrayType = ElementType, "[" Dimensions "]" ;
ElementType = BaseType ;
Dimensions = Dimension, { "," Dimension } ;
Dimension = integer_literal ;
```

```mank
f32[10]
i32[2,3]         # multidimensional array (two arrays of three)
(i32,bool)[999]  # 999 tuples of i32 and bool
```

### String types

Strings represented by the `str` type are sequences of bytes (chars), with a positive length.
Strings are immutable (they cannot be changed once created).


The length of any string can be obtained from the `.length` attribute.
Strings are indexable from `0` to `length - 1`, indexing a string gives the character at
that position in the string, as a `char`.

### Pod types

A pod (from plain old data) is a collection of named elements, called fields, each with its own associated type.


Pod types are [declared](#pod-declarations) at the top level of a module and then referred to by name in the rest of the program.

### Enum types

An enum (from enumeration) is a collection of named members which may be used as constants within a program (e.g. each member could be a state).


Optionally each member can be associated with tuple or pod-like data to form a tagged union. The data can then be extracted when matched against in [swtich expression](#switch-expressions).


Enums are [declared](#enum-declarations) at the top level of a module and then referred to by name in the rest of the program.

### Tuple types

A tuple is an unnamed collection of (also unnamed) elements of various types.
The empty tuple is used as the void type (`void` in C or C++).

```ebnf
TupleType = "(", [TypeList], ")" ;
```

```mank
()                         # empty tuple/void
(i32, bool)                # tuple of i32 and bool
(MyPod, (i32, f64), bool)  # tuple of a pod type, a nested tuple, and a bool
(char)                     # one element tuple
```

### Lambda types

Lambda types are the types of [lambda expressions](#lambda-expressions), and functions/procedures.
A specific lambda type represents all function-like objects with the same parameters, and return type.

```ebnf
LambdaType = "\", [ParameterTypes], "->", ReturnType ;
ParameterTypes = TypeList ;
ReturnType = Type ;
```

```mank
\ -> char         # lambda with no parameters, returning a char
\char -> ()       # lambda that takes a char and returns void
\i32, i32 -> i32  # lambda that takes two integers returning a integer
# lambda taking a list of integers and a lambda and retuning a list of integers
\i32[], \i32 -> i32 -> i32[]
```

### List types

Lists (also called vectors) hold a dynamic amount of elements of another type (the element type).
The number of elements in a list can be queried with the `.length` attribute.

```ebnf
ListType = ElementType, "[", "]", ;
ElementType = BaseType ;
```

```mank
str[]           # list of strings
i32[][]         # list of lists of integers
(i32, bool)[]   # list of tuples of i32 and bool
```

### Type matches

Types either match or are different.

All named types (primitives like strings, bools, ints, along with pods) are always
different to any other type (other than their own type).

- At the top level, any type can be matched with a reference of its type.
  - This excludes references nested in composite types.
    -  e.g. ``(i32, ref bool) != (i32, bool)``.
- Fixed-size arrays match if they're the same length and their element types match.
- Lists match if their element types match.
- Tuples match if all elements of the tuples match, in the same order.
- Lambdas match if their parameters and return types match respectively.

Note for references there are additional restrictions on the types of values they can be assigned to alongside matching types.

### Type storage

All types with exception of [strings](#string-types) and [lists](#list-types) are stored on the stack,
and are copied by value when assigned to a variable or passed to a function (for [reference types](#Reference-types) the value is just the pointer/memory address).

Strings are either stored within the program's static data or on the heap (depending on if they're compile-time constants or not), and lists are always heap-allocated. When these types are copied only their internal pointers are copied. This poses no issues for strings (as they're immutable), but it does mean lists need to be explicitly copied in some cases (e.g. before passing a list to a function that does some in-place processing while also maintaining the original list).
