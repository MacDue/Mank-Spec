## Lexical elements

### Letters and digits

```ebnf
letter = ('A' .. 'Z') | ('a' .. 'z') ;

decimal_digits = digit, { digit } ;
digit =  '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' ;
```

### Comments

```mank
# This is a comment.
```

Single line comments start with `#` and stop at the end of the line.


Comments cannot start within a string literal.
All comments are treated as whitespace, and they don't currently have any semantic meaning (e.g. documentation generation).

<!-- <div class="page"/> -->

### Integer literals

Integer literals are sequences of digits representing a integer constant.

```ebnf
integer_literal = decimal_digits ;
```

There's currently no special prefixes to allow for different bases.

```mank
42
1337
099     # = 99
000000  # = 0
100
```

### Floating point literals

Floating point literals are decimal representations of floating point constants.


Floating point literals consist of a integer part, decimal point, and a fractional part. The fractional part may be omitted (though keeping the decimal point).

```ebnf
floating_point_literal = decimal_digits, '.', [ decimal_digits ] ;
```

```mank
42.       # = 42.0
10.0
3.14
1.00      # = 1.0
00032.    # = 32.0
0010.000  # = 10.0
093.0     # = 93.0
```

### Indentifiers

Indentifiers name things within a program, such as varaibles and functions. Indentifiers must start with a letter or underscore, and then can be followed by any number of letters, underscores, or numbers.

```ebnf
indentifier = (letter | '_'), { letter | digit | '_' } ;
```

```mank
foo_bar
FooBar
_foo
_0123
bar1
fooBar
```

### Keywords

```
fun    proc   domain  until   pod    bind   continue
of     spawn  if      return  ref    loop   as
else   for    while   true    false  break  test
```

### Operators and punctuation

```
|!  ||  <<  >>  >=  <=  ==  !=  &&  +=  -=  |=  &=
/=  *=  %=  +   -   /   *   %   <   >   ~   &   |
!   ¬   ->  ..  ,   ;   :   .   {   }   (   )   [
]   =   @   \
```

### Character literals

### String literals

```mank
"Hello World", "I'm on a type writer!\r\n", "Name set to \"Ben\""
```
