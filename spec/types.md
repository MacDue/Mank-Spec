## Types

```ebnf
Type = ReferenceType | BaseType ;
BaseType = TypeName | LambdaType | TupleType | FixedSizeArrayType | ListType ;
```

`TypeName` covers any type referred to by name, such as primative types and pod types.
Other types are constructed from simpler types, e.g. a tuple type could be `(i32, bool)`.

### Boolean types

Booleans represented by the `bool` type, and can take the value `true` or `false`
(which are builtin keyword constants).

### Numeric types

Numeric types are sets of floating point or integer types (which also includes `char`).

```mank
char  # set of all signed 8-bit integers (that use character literals)
i32   # set of all signed 32-bit integers
i64   # set of all signed 64-bit integers

f32   # set of all 32-bit floating point numbers
f64   # set of all 64-bit floating point numbers
```

More numeric types are planned but are currently unimplemented.

### Reference types

A reference is a type holds a pointer to a value of another type (the referenced type).
References cannot be null.


In some cases (such as in lambda expressions, or bind statements), the referenced type can be omitted, and inferred based on context.

```ebnf
ReferenceType = "ref", [ReferencedType] ;
ReferencedType = BaseType ;
```

```mank
ref i32
ref bool[10]
```

### Fixed size array types

Fixed size arrays hold a fixed number (the length) of elements of another type (the element type). The length cannot be a negative number, the length of a fixed sized array can be obtained from the `.length` attribute.


Fixed size arrays can have multiple dimensions, given by a comma seperated list of dimensions.
In outermost dimension is given first, preceding to nested dimensions.

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



### Pod types

### Tuple types

### Lambda types

### List types
