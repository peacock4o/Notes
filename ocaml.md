ocaml
=====
- Category: Learning
- Tags: 
- Created: 2024-10-19T15:59:54-07:00

# OCaml Basics

## Types and values

Try using utop for a CLI.

Example:

```ocaml
42;; (;; to end the expression)
	- : int = 42
```

```ocaml
42 = value
int = value type
- = value not given a name
```

We can bind values to names with a ```let``` definition

```ocaml
let x = 42
	val x : int = 42
```

Think of this as "x has type int and equals 42"

## Functions

We can define functions in this way as well. 

```ocaml
let increment x = x + 1
	val increment : int -> int = <fun>
```

Follow the same "pronunciation" structure as a variable. 
"increment has type int input to int output and equals this (unprintable) function represented with <fun>"

Think of the arrow as a "transformation" (or function, more intuitively)

Now, what are the different ways we can call this function?

```ocaml
increment 0
	- : int = 1
increment(21)
	- : int = 22
increment(increment 5)
	- : int = 7
```

Typically, OCaml refers to this as "applying" a function rather than "calling" it.

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

In our project's directory, create a file called dune. For our example of a hello_world program, put the following in the dune file

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
