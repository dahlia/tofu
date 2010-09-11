Tofu Programming Language
=========================

Tofu is a programming language to be embedded in C++ programs.  You can use it
to embed while you are writing a big C++ program and it should have various
small plug-in programs.  If you want a nicer embeddable programming language,
there are [Lua][] and [Io][].

  [lua]: http://www.lua.org/
  [io]: http://iolanguage.com/


Functions
---------

There are 5 built-in types in Tofu.

 - function
 - boolean (true, nil)
 - string
 - list
 - map

Every type is derived from function type, so all values are like function.
Actually there are no types; there are only functions, just behave as other
types.

Following code defines some functions.

    abs <- (number): (number < 0)("-1", 1) * number

    max <- ([numbers]): {
      top <- numbers(0)
      foreach numbers, (number): {
        .top <- (top > number)(top, number)
      }
      top
    }

You may be able to infer some interesting rules.

 - The assignment operator is not `=`, but `<-`.

 - No named function definition syntax; However, you can assign function values
   to variable name, like JavaScript, Lua.  Tofu supports lambda functions with
   following syntax:

       (arg1, arg2): form
       (arg1, arg2): { form; form2 }

 - Two literal types.  General function literal (block), and string literal.
   For example, `123` same as `"123"`.

 - When boolean values are called, they behave like trinary operator in the
   other well-known languages such as C and Java.

 - There's no specific number type.  Instead, it can use arithmetic operators
   like `+`, `*` with string values.

 - Instead of the `return` keyword in other languages, it just returns last
   expression of function block, in similar way to `begin` of Scheme.


Lambda And Lexical Closure
--------------------------

All values are functions and all functions are values in the Tofu 
programming language.  To call function generally, you should name it.
'Naming' in computer programming means assignment or binding.

    add <- (x, y): x + y

We can call evaluating `(x + y)` `add(x, y)`.  After this naming, `add(3, 9)`
means `(3 + 9)` viz. `12`.  Functions can be more complex and longer.

    multiply <- (x, y): {
      result <- 0; i <- 0
      loop (): {
        result <- add(result, x)
        i <- i + 1; i <= y
      }
      result
    }

When the function contains two or more expressions, curly brackets should wrap
its body, and EOL or semicolon separates each expression.

However, above code is a just example. You can multiply easily by `*`.

    multiply <- (x, y): x * y

Functions keep their own context environment.  Read a following example code:

    accumulator <- (initval <- 0): {
      (num <- 0): {
        .initval <- initval + num
      }
    }

    acc <- accumulator(1)
    acc(2)
    acc(3)
    acc()

The last call returns ``6``. The form `.var <- val` is similar to
`(set! var val)` in Scheme.

Language    | Definition            | Assignment             
----------- | --------------------- | -----------------------
Tofu        | `name <- value`       | `.name <- value`      
Scheme      | `(define name value)` | `(set! name value)`    
JavaScript  | `var name = value`    | `name = value`         
Lua         | `local name = value`  | `name = value`         
Python (>3) | `name = value`        | `nonlocal name = value`
Perl        | `my $name = value`    | `$name = value`        


Omitting Parentheses In Function Call
-------------------------------------

You can omit parentheses in function call when possible and readable.
Following two codes share the same behavior.

    func 1, 2, 3
    func(1, 2, 3)

However you cannot omit it when there are no arguments to pass into `func`.
Following expression returns `func` itself, not the value in the calling `func`.

    func

If you want to call `func` without any arguments, a pair of parentheses should
be explicit.

    func()

Omitting parentheses of function's arguments makes calling higher-order function
and applying function with list or dictionary prettier.

    # function with list and dictionary
    func []
    func [1, 2, 3]
    func {}
    func { a <- 1; b <- 2 }

    # higher-order function
    map (filter [1, 2, 3, 4], (x): x % 2 == 1), (x): {
      log(x & "\n")
    }


Pseudo-attributes And Pseudo-methods
------------------------------------

Tofu programming language has no attributes. It is just syntactic sugar
for calling function.  Following two expressions are the same.

    function.attribute
    function("attribute")

In other words, all function call can be also attribute accessor of the
object.  Likewise setting attribute is translated into calling function.  The
name of attribute passes into first parameter and value to set passes into
second it.

    function.attribute <- value
    function("attribute", value)

You can apply this feature to imitate object's method call.  It will behave as
higher-order function.

    object.method()
    object("method")()

You can use non-negative numeral as the name of attributes also.

    list.0           # list(0)
    list.0 <- value  # list(0, value)


Pseudo-operators
----------------

There are no operators in Tofu programming language.  However,
pseudo-methods call can seem to use operators.  Following expressions are the
same.

    value operator operand      # pseudo-operator style
    value.operator(operand)     # pseudo-method style
    value("operator")(operand)  # higher-order function style

The pseudo-operator syntax can be used only when just one argument passes into
method.


String Literal
--------------

There are two types of the string literal in Tofu programming language.

 - double quotation
 - numeric string

The double quotation string literal is a widely used way to represent string
values.  It can contains any characters except double quotation character.

    "To be, or not to be."
    "1234"
    "!@#@!#@!#"

    "string with
    a line break"

To contain double quotation character, it should be escaped with backslash.

    "Lin Chi said, \"If you meet the Buddha on the road, kill him.\""

The numerical string literals are used usually to represent numbers.  Because
there's no specific number types in Tofu.  However, `123` equals `"123"`.

    42
    "3.14159265"


List And Map
------------

To contain one or more values, you can use containers.  There are two built-in
container types.

 - ordered list, e.g. `[]`, `[1, 2, 3]`, `["hello", [1, 2], (x): x + 1]`
 - map of string keys and values, e.g. `{ name <- "Hong Minhee"; age <- 20 }`

Lists have index numbers each element which starts from 0, and, it does't limit
the number of elements. It accepts any type.

    list <- ["a", "b", "c", [1, 2, 3]]

You can count the number of elements in the list.  Use `count` function.

    count(list)  # 4

To iterate the list, use `foreach` function.  This is a higher-order function
which takes a function takes one or two arguments: value and optional index
number of element.

    foreach list, (value): stdout.write(value ++ "\n"
    # it prints elements of list

    foreach list, (value, index): {       # it prints elements and its index
      stdout.write(index ++ ". " ++ value)  # of list
      stdout.write("\n")
    }

If you need `for`-like loop of C/C++, use function `range`.  It generates a
list that contains sequential numbers.  It is equal to the function of the
same name in Python.

    range(5)         # [0, 1, 2, 3, 4]
    range(3, 7)      # [3, 4, 5, 6]
    range(2, 12, 3)  # [2, 5, 8, 11]

    foreach range(1, 10), stdout.write  # it prints: 123456789

Maps have keys of string and its values.  The key and its value are called
"pair", and you can translate pairs of the map to the two-dimensional list by
function `pairs`.

    name <- { family <- "Hong"; given <- "Minhee" }
    pairs(name)  # [["family", "Hong"], ["given", "Minhee"]]

There are `keys` and `values` also.

    keys(name)    # ["family", "given"]
    values(name)  # ["Hong", "Minhee"]

There is no order in the map, so, order of the list which returned by `pairs`,
`keys` and `values` depend on implementation.

Functions `count` and `foreach` are generic: it works well for maps also.

    count(name)  # 2

    foreach name, (key): write(stdout, key ++ ": " ++ name(key) ++ "\n")
    # above prints: (but order of lines are not deterministic.)
    #   family: Hong
    #   given: Minhee


Immutable values
----------------

All types of values in Tofu are immutable.

Lists and maps are also immutable.  So you cannot append a list or set a new
key of a map.  Instead, they returns a new list or a new map.

    lst <- [1, 2, 3]
    newlst <- lst ++ [4, 5]     # [1, 2, 3, 4, 5]
    newlst2 <- newlst << 6      # [1, 2, 3, 4, 5, 6]

In the same manner, strings are immutable.

    str <- "abcd"
    newstr <- str ++ "ef"  # "abcdef"
    replace(newstr, "ab", "AB")  # "ABcdef"


BNF
---

    <program>      ::= { <expr> <terminate> } [ <expr> ]
    <termiate>     ::= ( ";" | <newline> ) [ <terminate> ]
    <expr>         ::= "(" <expr> ")"
                     | <literal>
                     | <id>
                     | <attr>
                     | <apply>
                     | <operator>
                     | <define>
    <literal>      ::= <str-literal>
                     | <dict-literal>
                     | <func-def>
                     | <list-literal>
    <str-literal>  ::= /"([^"]|\\.)*"/
                     | <number>
    <number>       ::= /\d+/
    <dict-literal> ::= "{" <program> "}"
    <func-def>     ::= "(" <params> ")" ":" <dict-literal>
    <list-literal> ::= "[" { <expr> "," } [ expr [ "," ] ] "]"
    <lvalue>       ::= [ "." ] <id>
                     | <attr>
    <id>           ::= /[^[:digit:][:space:]][^[:space:]]*/ except "<-"
    <attr>         ::= <expr> "." ( <id> | <number> )
    <apply>        ::= <expr> "(" <args> ")"
                     | <expr> <args>
    <args>         ::= { <expr> "," } [ <expr> [ "," ] ]
    <operator>     ::= <expr> <id> <expr>
    <define>       ::= <lvalue> "<-" <expr>


Author And License
------------------

Tofu is designed and implemented by Hong Minhee. <http://dahlia.kr/>

The implementation is distributed under MIT license.

    Copyright (c) 2010 Hong Minhee <http://dahlia.kr/>

    Permission is hereby granted, free of charge, to any person
    obtaining a copy of this software and associated documentation
    files (the "Software"), to deal in the Software without
    restriction, including without limitation the rights to use,
    copy, modify, merge, publish, distribute, sublicense, and/or sell
    copies of the Software, and to permit persons to whom the
    Software is furnished to do so, subject to the following
    conditions:

    The above copyright notice and this permission notice shall be
    included in all copies or substantial portions of the Software.

    THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND,
    EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES
    OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND
    NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT
    HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY,
    WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING
    FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR
    OTHER DEALINGS IN THE SOFTWARE.

