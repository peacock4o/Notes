ocaml
=====
- Category: Learning
- Tags: 
- Created: 2024-10-19T15:59:54-07:00

# OCaml Basics

## Types and values

utop is a good CLI tool for OCaml. We'll use it for a number of examples.

When we write a line in OCaml toplevel, we end it with ``;;``

Example:

```ocaml
42;; (;; to end the expression)
	- : int = 42
```
In the above statement,
- ``-`` indicates value not assigned
- ``int`` = type 
- ``42`` = value

We can assign variables with a ```let``` definition

```ocaml
let x = 42;;
	val x : int = 42
```

Think of this as "x has type int and equals 42"

## Functions

We can define functions in this way as well

```ocaml
let increment x = x + 1;;
	val increment : int -> int = <fun>
```

It's like a variable! Follow the same "pronunciation" structure as a variable. 
"increment has type int input to int output and equals this (unprintable) function represented with <fun>"

Think of the arrow as a transformation of the input into the output

Now, what are the different ways we can call this function?

```ocaml
increment 0;;
	- : int = 1
increment(21);;
	- : int = 22
increment(increment 5);;
	- : int = 7
```

Typically, OCaml refers to this as "applying" a function rather than "calling" it.
- This kind of echoes the general programming style of OCaml and functional languages. Start with something, then apply various functions until you get what you want. 

Also, good OCaml style tries to omit parenthesis when possible. One of the challenges is figuring out where parenthesis are actually required. Adding parenthesis may also solve syntax errors sometimes

## Loading code into toplevel.

In order to load other OCaml code, use the following directive:

```ocaml
#use "<filename>.ml";;
```

## Typical toplevel workflow

From the website:

The best workflow when using the toplevel with code stored in files is:

- Edit the code in the file.
- Load the code in the toplevel with #use.
- Interactively test the code.
- Exit the toplevel. Warning: do not skip this step.

## Storing code in files

General flow goes as such:

- Create a directory for your project. Generally, try to avoid using /home/<name> bcause dune doesn't like it
- Create your <filename>.ml file inside the folder
- Save the file. compile it like so.

```bash
ocamlc -o <outputname>.byte <inputfile>.ml
```

It'll also produce a couple .cmo and .cmi files. Remember to clean these up after.

## What about main?

Ocaml is a functional programming language. You write a bunch of functions that chain together.
Therefore, Ocaml doesn't actually have a main function. You just have the very last definition in a file kick off whatever computation you need.

## Dune

What about larger projects? Doesn't really make sense to manually compile each time via command line.
So instead, we'll get our build system to automatically find and link libraries.

In our project's directory, create a file called dune. This file uses s-expressions. For more information, read here -> ``https://dune.readthedocs.io/en/stable/reference/dune/index.html``

For our example of a hello_world program, put the following in the dune file

```
(executable
 (name hello))
```

This says "Hey, I'd like to declare an executable whose main file is "hello.ml"

Also, create a file named dune-project and put the following in it:

```
(lang dune 3.4)
```

This tells Dune that the project uses Dune 3.4, but obviously replace with whatever version you're using.
The dune-project file is needed in the root directory of each project using Dune. 

Then run the following command:
```bash
dune build hello.exe
```

Note that it uses .exe. This tells dune to create a native executable rather than a bytecode executable.
Running this command creates a build directory and uses it as a sandbox for all the crap files generated during compilation. Our executable is in _build/default/<filename>.exe
We can build and execute in one go by running:

```bash
dune exec ./hello.exe
```

Then, to get rid of all the leftover files from compilation and just leave the executable, we can do:
```bash
dune clean
```

Don't edit files inside _build! If you're getting errors, this could be the reason.

## Creating a dune project automatically

Workflow for doing this:
- Go to the subdirectory where you want to start your project's directory
- dune init project <project_name>
- cd ./<project_name>
- code . (for VSCode)
- (Do your changes and write your files)
- dune exec bin/main.exe

To automatically format your source code, add a .ocamlformat file to the prject's root directory (thje directory wit hthe dune-project file in it.)

To automatically rebuild and recompile whenever there's a change to the project's files, run:
```bash
dune build --watch
```

This starts a blocking process that waits for filesystem changes and rebuilds every time a file change happens.

## Expressions

Programs in functional languages are built out of expressions.
- Expression : functional languages :: command : imperative languages
A functional language's primary task is to evaluate an expression to a value.
- A value is an expression with no computation remaining.
	- All values are expressions, but not all expressions are values.
An expression might not evaluate to a value if:
- Evaluation raises an exception
- Evaluation never terminates (infinite loop)

## Primitives

Primitive types are the most basic kinds, like with other languages. These include...
- ``int``
	- Range from ``-2^62`` to ``2^62 - 1``
		- You may notice the -1. A bit is reserved to indicate whether the value is a pointer or not. Therefore, this is a 63-bit value, or +/-62 bits.
		- If true 64 bit is needed, there's an ''Int64'' module in standard library. Beyond 64-bit, there's the ``Zarith`` library.
- ``float``
	- IEEE 754 format double-precision float.
	- You must use float values **and float operators** when operating with floats (like ``3.14 *. 2.``)
	- OCaml doesn't automatically convert between int and float. Do this with ``int_of_float`` and ``float_of_int``
		- OCaml *intentionally* doesn't do a lot of the automatic type conversion.
	- Floating point operations can result in rounding errors
- ``bool``
	- ``true`` and ``false``
	- ``&&`` (and) and ``||`` (or) will short circuit like other languages
- ``char``
	- Written with single quotes - ``'a', 'b', 'c',`` etc
	- Represented as a single byte
	- Convert between characters and ints with ``char_of_int`` and ``int_of_char``
- ``string``
	- Sequence of characters
	- Written with double quotes
	- Concatenation operator is ``^``
		- Ex. ``"abc" ^ "def"``
	- Convert other types into strings with ``string_of_int``, ``string_of_float``, and ``string_of_bool``.
		- Strangely, no ``string_of_char``. Use ``String.make`` instead.
	- Convert strings into other types (**if possible**) with ``int_of_string``, ``float_of_string``, and ``bool_of_string``.
		- Strings are sequences of chars, so just access by index like so: ``"abc".[0] -> 'a'``

## More Operators

See the manual for more operators. -> ``https://ocaml.org/manual/5.2/expr.html#ss%3Aexpr-operators``

Worth bringing up the equality operators.
- ``=`` with negative version ``<>`` examines structural equality
- ``==`` with negative version ``!=`` examines "physical" equality

We'll dive further into this in the future. For now, get used to using ``=`` for comparison instead of ``==`` like you're used to.

## Assertions

The expression ``assert e`` evaluates e. 
- If it's true, the whole expression evaluates to a special value called *unit*. 
- If it's false, an exception is raised.

Example for testing a function 
```ocaml
let () = assert (f input1 = output1)
let () = assert (f input2 = output2)
let () = assert (f input3 = output3)
```
^ Note that we use let() here. This is done to handle the unit value returned by the assertions.

## If Expressions (!)

OCaml if expressions  are written in the format ``if e1 then e2 else e3``. We can also split this up into multiple lines - see below.

```ocaml
if e1 then e2
else if e3 then e4
else if e5 then e6
...
else en
```

Here, ``e`` corresponds to any OCaml expression. We call ``e1`` the ""guard" of the ``if`` expression.

Generally, try to regard the final ``else`` statement as mandatory.

If-then-else **expressions** (not statements) are expressions like any other. This means that they can be evaluated anywhere an expression could.
- ``4 + (if 'a' = 'b' then 1 else 2)``

Let's look at our expression above with ``e1`` and ``e2`` and analyze it a bit.
- Dynamic semantics (of an if expression)
	- If ``e1`` evaluates to ``true``, and if ``e2`` evaluates to a value ``v``, then ``if e1 then e2 else e3`` evaluates to ``v``
	- If ``e1`` evaluates to ``false``, and if ``e3`` evaluates to a value ``v``, then ``if e1 then e2 else e3`` evaluates to ``v``
		- This defines how to evaluate expressions. It takes two rules to describe evaluation of an ``if`` expression, one for if guard is true, and one for if guard is false.
		- Using a less mathematical form of explanation for now. We'll come back to it later.
- Static semantics
	- If ``e1`` has type ``bool`` and ``e2`` has type ``t`` and ``e3`` has type ``t`` then ``if e1 then e2 else e3`` has type t
		- Call this a "typing rule"" - describes how to type check an expression.
		- Both outputs are of type t, so the output is of type t.
			- Think - why would an expression return two different types of variable? How stupid would that be? (ahem... python...)
		- We use "type ``t``" to represent any expression type.
			- Instead of saying "``e`` has type ``t``" and repeating "has type" over and over, instead use a colon, like so: ``e : t``
				- This is also consistent with how the toplevel responds - ``val x : int = 42`` means ``value x has type int of value 42``
		- Makes no difference whether guard is true or false - in fact, the compiler has no way to tell this at compile time.

## Let Expressions

In previous usage, we've been making definitions for values, like with ``let x = 42;;`` -> ``val x : int = 42``
- We'll call this usage a *let definition*

We can also use ``let`` as an expression.

Ex.  ``let x = 42 in x + 1`` ->  ``- : int = 43``

Here, we bind a value to the name x, then use that name/binding inside another expression. We'll call this a *let expression*
- This is not the same as a *let definition*. Let definitions don't evaluate to anything, but let expressions do.
- **Definitions do not evaluate to values.** This is why you can't do something like ``(let x = 42) + 1`` - the compiler expects a value on the left-hand side of the plus sign.
	- However, ``(let x = 42 in x) + 1`` works fine. ``- : int = 43``

Here's another way to think about definitions.

```ocaml
# let a = "big";;
# let b = "red";;
# let c = a ^ b;;
# ...
```

... evaluates the same as...

```ocaml
let a = "big" in
let b = "red" in
let c = a ^ b in
...
```

- Syntax
	- ``let x = e1 in e2``
	- ``x`` is called an "identifier." 
	- ``e1`` is called the *binding expression*. This is what x is being bound to.
	- ``e2`` is called the *body expression*. This is the body of code in which the binding will be in scope.

- Dynamic semantics
	- Evaluate e1 to a value v1.
	- Substitute v1 for x in e2, yielding a new expression e2'.
	- Evaluate e2' to a value v2.
	- The result of evaluating the let expression is v2.

Here's an example.

```ocaml
let x = 1 + 4 in x * 3
	--> (evaluate binding expression e1 to value v1)
let x = 5 in x * 3
	--> (substitute v1 for x in body expression e3, yielding e2')
5 * 3
	--> (this is e2'. evaluate this to v2)
15
	--> (this is the final result of our let expression, v2)
```

- Static semantics
	- If ``e1 : t1`` and if under the assumption that ``x : t1`` it holds that ``e2 : t2``, then ``(let x = e1 in e2) : t2``
		- Basically, if ``e2`` has ``t2`` then the let expression has ``t2``, which seems obvious.

## Scope

``Let`` bindings are in effect on in the block of code in which they occur. This is just like other languages.

The "scope" of a variable is where its name is meaningful. From our previous example, I could only use ``x`` inside the following body expression.

It's possible to have overlapping bindings of the same name in the same scope, but it's terrible form. 
- Hold on - but aren't variables in OCaml immutable? What's all this "same name" nonsense?
	- Think about previously, how we could write let declarations instead as chains of let expressions. Each new ``x`` "shadows" the previous for that scope.

Here's an example.

```ocaml
let x = 5 in
  ((let x = 6 in x) + x)
```

But what even happens here?

```ocaml
(* possibility 1 *)
let x = 5 in
  ((let x = 6 in 6) + 5)

(* possibility 2 *)
let x = 5 in
  ((let x = 6 in 5) + 5)

(* possibility 3 *)
let x = 5 in
  ((let x = 6 in 6) + 6)
```

The first option is probably the best answer for most. But why? Because of the "Principle of Name Irrelevance." Identical functions need to work regardless of name.
- See ``https://cs3110.github.io/textbook/chapters/basics/expressions.html`` for the written out example.

Study this utop transcript. A bit of a brain bender.
```ocaml
# let x = 42;;
val x : int = 42
# let f y = x + y;;
val f : int -> int = <fun>
# f 0;;
: int = 42
# let x = 22;;
val x : int = 22
# f 0;;
- : int = 42  (* x did not mutate! *)
```

Why does this happen? IDK, honestly. I think it's because ``f`` was defined when ``x = 42`` was in scope. We don't pass the new value of ``x`` to ``f`` in the second evaluation, so it uses the one in the same scope.
- Actually, it's because the second ``x`` is beyond the scope of ``f``

## Type Annotations

Sometimes, it can be useful to manually specify the desired type of an expression. A *type annotation* does that.
- ``(5 : int)`` -> "5 has type int"
- ``(5 : float)`` causes a compile-time error because it's not the right type. Remember - OCaml is strict!
	- This makes it useful for debugging. 
		- Imagine you have ``5 +. 1.1`` (wrong because 5 should be a float)
		- You could check by annotating 5 as a float. ``(5 : float) +. 1.1`` would cause a compile-time error as previously mentioned.

Annotations are **NOT** type casts. They're just a check that an expression actually *is* a given type.

- Syntax
	- ``(e : t)``
		- Unlike other examples, parenthesis are actually required
- Dynamic semantics
	- No run-time meaning for a type annotation - it only exists to be useful in compiler-land. ``(e : t)`` compiles down to ``e``
- Static semantics
	- If ``e`` has type ``t`` then ``(e : t)`` has type ``t``

## Functions

Functions are independent. Methods are part of objects. Remember this difference.

``let x = 42`` is an example of a definition.

``let f x = ...`` is an example of a function definition.

``let rec f x = ...`` is an example of a recursive function definition.


