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

Numeric types are sets of floating-point or integer types (which also includes `char`).

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

A pod (from plain old data) is named a collection of named elements, called fields, each with its own associated type. The name of a pod and its fields are [identifiers](#identifiers). The pod's name must be unique among all other pods in a module, and each field name must be distinct from the others within a pod.

```ebnf
PodType = "pod", "{", [Fields], "}" ;
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
```

### Tuple types

A tuple is a unnamed collection of (also unnamed) elements of various types.
The empty tuple is used as the void type (`void` in C or C++).

```ebnf
TupleType = "(", [TypeList], ")" ;
```

```mank
()                         # empty tuple/void
(i32, bool)                # tuple of i32 and bool
(MyPod, (i32, f64), bool)  # tuple of a pod type, a nested tuple and a bool
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
