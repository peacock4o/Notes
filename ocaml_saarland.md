saarland
========
- Category: Learning
- Tags: 
- Created: 2025-01-28T17:22:57-08:00

``https://www.ps.uni-saarland.de/~smolka/drafts/prog2021.pdf``

# CHAPTER 1 - GETTING STARTED

## 1.1 - Programs and Declarations

- An OCaml program is a series of declarations executed in the order of being written.
	- Ex. ``let a = 2 * 3 + 2``
		- ``2 * 3 + 2`` - **expression** to be evaluated. Expressions exist to be evaluated.
		- ``a`` - **identifier** to which the evaluated expression's value (8) is bound
	- The OCaml **interpreter** takes in values and determines if they're valid. 
		- If so, determine a **type** for every identifier and expression
	- Declared variables in OCaml are immutable, but you can **shadow** them with another declaration. 
		- This essentially replaces the given identifier with another one of the same name in the given scope.
	- Declarations in this chapter consist of:
		- **Keyword** - ``let``
		- **Head** - ``rec fact n``
		- ``=``
		- ``Body`` - ``...`` (insert code here)
			- The bodies of declarations are expressions
			- Expressions can be nested with parentheses. These parentheses can be:
				- Redundant - ``((2 * 3) + 2) - x``
				- Not redundant - ``2 + (2 * 3) - x``
					- Try to avoid redundant parentheses - bad style.
	- Every expression has a type. 
		- Previous examples have been of type ``int``.
		- OCaml's ``int`` is limited, though.
			- Limited to max size of variable, as opposed to $\Z$, which is infinite.
			- Generally, this limitation is fine with sufficiently small calculations.

## 1.2 - Functions and Let Expressions

- The declaration ``let square x = x * x`` declares a function.
	- The identifier ``square`` receives a **functional type** describing functions that, given an integer, return an integer.
		- AKA ``int -> int``
	- Now that we've declared a function, execution of the declaration ``let a = square 5`` binds the identifier ``a`` to the ``int`` ``25``
		- We can string together multiple functions.
			- ``let pow8 x = square (square (square x))``
			- We evaluate the innermost ``square x`` first, then use the value to evaluate the next, and so on.
				- Remember - expressions exist to be evaluated.
		- Can think of a function as an incomplete evaluation bound to an identifier
	- Functions can be declared locally with *let expressions* (within a given scope)
		- ``let a = x * x in let b = a * a in let b * b``
		- Let expressions != top level declarations.
			- Tell them apart with the ``in`` keyword - let expressions have it, TL declarations don't

## 1.3 - Conditionals, Comparisons, and Booleans

- Consider the following functions.
	- ``let abs x = if x < 0 then -x else x``
		- Declares a function ``abs : int -> int``
		- ``x < 0`` is a **comparison**. Evaluates to type ``bool``
		- The keywords ``if ... then .. else ...`` form a **conditional.**
	- ``let max x y : int = if x >= y then x else y``
		- Declares a function ``max : int -> int -> int``
			- More than two ``int`` - what does it mean?
				- The last one is always the return type.
				- All the rest before are the argument types.
		- Uses an explicit return type ``: int`` in the declaration's head.
			- Tells us that we're returning an int
			- Since the expression returns the parameters, it's automatically required for ``x`` and ``y`` to be ints.
	- ``let max3 x y z = max (max x y) z``
		- Declares a function ``max3 : int -> int -> int -> int``
		- ``max`` forces ``int -> int -> int``, so we don't need to annotate any types
	- ``let test (x : int) y z = if x <= y then y <= z else false``
		- The **type specification** for x is required because we could be comparing strings, bools, etc. otherwise. Not clear.
			- We can only compare values of the same type, so x forces ``y`` which forces ``z``
			- Alternatively, we could specify ``y`` or ``z`` or all of the arguments. 

## 1.4 - Recursive Power Function

- Let's try to create a function computing powers of $x^n$.
	- Powers satisfy the equations:
		- $x^0 = 1$
		- $x^n = x * x^{n-1}$
	- Using these equations, we can solve $2^3$
		- $2^3 = 2 * 2^2$
		- $= 2 * 2 * 2^1$
		- $= 2 * 2 * 2 * 2^0$
		- $= 2 * 2 * 2 * 1$
			- 1st equation applied instead of 2nd like previous
		- $= 2 * 2 * 2$
		- $= 8$
	- This is a typical recursive computation, which we can capture with a **recursive function**
		- Let's first combine the two equations we defined.
			- $x^n = IF n < 1 THEN 1 ELSE x * x^{n-1}$
		- Then convert this into code
			- ``let rec pow x n = if n < 1 then 1 else x * pow x (n - 1)``
				- Declares ``pow : int -> int -> int``
				- Designed mathematically, implemented in any language

## 1.5 - Integer Division

- Division formula:
	- Given two integers $x \ge 0$ and $y > 0$, there exist unique integers $k,r \ge 0$ such that $x = k * y + r$ and $r < y$
		- In English? When dividing ``x`` by ``y``, you get a quotient ``k`` and remainder ``r``
		- This is called Euclidean division, or integer division.
		- Also think of it like placing identical boxes on a shelf until no more can fit.
			- i.e. If you place another beyond this point, it'll fall off.
	- We can do this and modulo (output ``r`` instead of ``k``)
	- Remember that div/mod can be found with consecutive subtractions of y.
- Digit sum:
	- With div and mod, we can extract digits by dividing/mod by multiples of 10. 
	- ``let rec digit_sum x = if x < 10 then x else digit_sum (x / 10) + (x mod 10)``
		- ``digit_sum : int -> int``
- Digit reversal:
	- What about a function that reverses the digits of an int?
	- The trick is to use an additional **accumulator argument**.
		- Start it at 0, use it to gather the digits we cut off.
		- Looking at the trace might help
			- ``rev' 456 0 = rev' 45 6 = rev' 4 65 = rev' 0 654 = 654``
	- We also use a **worker function** to do the operations, which is the one that includes the accumulator argument.
- GCD
	- Follow Euclid's algorithm
	- ``let rec gcd x y = if y < 1 then y else gcd y (x % y)``
- Note that these employ tail recursion. We'll discuss this concept later.

## 1.6 - Mathematical Level vs. Coding Level

- When we design a function, we do so at the mathematical level.
	- We refer to the realization of our algorithm of the function in a programming language as coding
- Important to distinguish between the mathematical and coding levels
	- Some key differences:
		- ``int`` is a limited mechanical construct. Instead, we'l use infinite mathematical types.
			- $\N$ - natural numbers (0, 1, 2, 3, 4, ...)
			- $\N^+$ - positive integers (1, 2, 3, 4, ...)
			- $\Z$ - integers (.., -2, -1, 0, 1, 2, ...)
			- $\B$ - booleans (false, true)
	- Useful to declare mathematical type for your function first, then come up with a collection of equation sufficient to compute the function.
		- Ex. power function
			- Start with *specifying* type and equation:
				- Mathematical type $pow : \Z \rightarrow \N \rightarrow \Z$
				- Equation $pow x n = x^n$
					- Think of this as the end goal. What is the function trying to accomplish?
			- Now, introduce *defining equations*
				- Powers $x^n$ satisfy the equations: 
					- $x^0 = 1$
					- $x^n = x * x^{n-1}$
				- We adapt these to the function ``pow``
					- ``pow x 0 = 1``
					- ``pow x n = x * pow x (n - 1) if n > 0``
						- Note that the second equation now comes with an **application condition**
					- These equations are **exhaustive** and **disjoint**
						- Disjoint - only one equation is applicable at a time
						- Exhaustive - valid for all values of ``x`` and ``n`` respective of the types specified by ``pow``
					- Summarize defining mathematical equations with:
						- $pow : \Z \rightarrow \N \rightarrow \Z$
						- ``pow x 0 := 1``
						- ``pow x n := x * pow x (n - 1) is n > 0``
							- ``:=`` reads like "is defined as"
					- We observe the defining equations are **terminating**.
						- This is the **termination argument**:
							- Each recursion step issued by the second equation decreases ``n`` by 1
							- Since ``n`` occupies natural numbers, it must eventually reach 0.
				- There are some issues with perfectly translating from mathematical form to code form.
					- Type changes to ``int -> int -> int``.
						- There isn't a special type for ``\N``, so it will admit arguments not admissible for the mathematical function.
							- Think negative numbers
							- These are called **spurious arguments**
					- What about terminating for negative numbers? We handle this by checking if below 1, but not perfect.
					- Type ``int`` isn't the full mathematical type $\Z$ of integers - just a finite interval of machine ints
				- Arguing the **correctness** of a function happens at the mathematical level using $\Z$ and $\N$.
					- When realizing in OCaml, we kind of just pray (lol) that our computations are small enough to fit in the machine int space. 
				
