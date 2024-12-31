rust
====
- Category: Learning
- Tags: 
- Created: 2024-11-03T20:33:07-08:00

GOAL: Use this in tabanus https://docs.rs/hpfeeds/latest/hpfeeds/


## CHAPTER 1 - Getting Started

To install rustup (used to then install rust):
```bash
$ curl --proto '=https' --tlsv1.2 https://sh.rustup.rs -sSf | sh
```

Make sure to reboot shell/computer to get rust into your $PATH variable.

Make a file with .rs extension, like hello_world.rs

Here's the hello world:

```rust
fn main() {
    println!("Hello, world!");
}
```

We define functions like ```fn ____()```
- Note that main runs first, like with C
- Allegedly, good style dictates to keep the opening curly bracket on the same line as the func declaration. Too bad I'm a bad programmer.
- Rust indentation is four spaces, **NOT** a tab.

Look at the println. What's up with that exclamation mark?
- This tells us that we're using a macro called println.
	- A function would have no exclamation mark.
- Macros can obey different rules from functions. 

Cargo is the build system and package manbager, like dune for OCaml. Try running it with:

```bash
cargo new <package_name>
```

This creates:
- TOML file with Cargo's configuration format
	- Which contains compilation information and dependencies
- A git repository within the new project folder
- An src folder with a sample main.rs file

```bash
cargo build 	#Builds an executable
cargo run 	#Runs the executable
cargo check 	#Checks if the code can compile, but doesn't build an executable
```

# CHAPTER 2 - Guessing Game

For this chapter, I'll write down some interesting lines and what they mean.

```let mut guess = String::new();```

Here, you usually declare variables with

```let guess = String::new();```
 
But the mut keyword enables the variable to be changed after being first declared - declares it to be *mutable* (not mutex)
Note that String::new() uses ::. This means that new() is an associated function of the string type. This particular one creates a new, empty string.

We use similar colon structure in the following:

```rust
io::stdin()
        .read_line(&mut guess)
        .expect("Failed to read line");

```

Again, we use the colon notation to denote usage of the stdin function from the io library.
If we didn't import the io library at the start, we would have to do std::io::stdin instead of just io:stdin

In this call, we string together .read_line and .expect. They're on seperate lines for readability

First part reads in a line from the stdin object (returned by stdin())
Note that we pass in &mut guess isntead of &guess. This is because references are *also* immutable by default.

- Secure reference usage is apparently a big selling point of Rust

read_line places the input string into the refernce. This is fine and dandy. It also passes back a Result enum - think of this as equal to returning 0/-1 for error code in C

- Each possible state of the enum is called a "variant"
- The Result enum only returns Ok or Err. The Err variant contains error information.
- If you don't call expect, Rust compiler warns you. You're *expect*ed (hah!) to put in error handling.

So we get our stdin object, read_line into &guess, and parse the return message.

```rust
println!("You guessed: {}", guess);
```

{} is a placeholder, like printf(). You can insert variables inline as well, like {x}

Now, let's add the rand crate to our project. We can do this by adding our package to Cargo.toml like so:
```rust
rand = "0.8.5"
```

This is nice because it takes the mystery out of managing packages. Cargo just installs from the file whenever you build. 

When you build for the first time, Cargo also makes a Cargo.lock file to determine which package versions to download/use
- This means you have to EXPLICITLY update. This is also nice!
- You can also automatically update with ```cargo update```
	- This ignores the lock file and finds the next update within this block of the update - but it won't autoupdate to the next step (like if I update from 0.8.5, it won't go past 0.9)
 
Let's use this rand package.

```use rand::Rng;```

In this case, we import the Rng "trait", which defines methods that the RNGs implement.

```let secret_number = rand::thread_rng().gen_range(1..=100);```

Let's thread together what we know already.
- Declaring secret_number immutably
- Using the thread_rng function from rand. This one creates a RNG in the current thread
- (new) the gen_range method takes a range and generates an int.
	- ..= is inclusive on lower and upper bounds. 


Try running this command to get a list of documentation for relevant packages in your browser:

```bash
cargo doc --open
```

^^ SUPER NICE!

```rust
use std::cmp::Ordering;

match guess.cmp($secret_number)
{
    Ordering::Less => println!("Too small!"),
    Ordering::Greater => println!("Too large!"),
    Ordering::Equal => println!("YOu win!"),
}
```
- Ordering is another enum, with the variants Less, Equal, and Greater.
- <___>.cmp(&<___>) returns one of the three Ordering enums
- We pass the variant into match, kind of like a base case. If it's abc, then do => xyz
	- "A match expression is made up of *arms*, which consists of a pattern to match against and the code that should be run."

If we try to compile now, it gives us an error for comparing string guess against int secret_number
- Rust is strong, static typing. But it also uses type inference. So it infers that guess is a string.
- Solve this by casting guess as an int and handling any error like so:
```
let guess: u32 = guess.trim().parse().expect("Please type a number!");
```
- Take guess, eliminate bounding whitespace
- The parse part is interesting. It turns your input into another type. Us annotating guess forces parse to turn the input into u32 (unsigned 32bit)
- Then expect does what we know.

```rust
loop {
    <code here>
}
```

Loop creates an infinite loop. But we need a way to break out of it.
- So instead of just the println for the "equals" match case, do this:

But then what about bad inputs? We don't want the program to fully crash when it errors.
- Instead, we'll run our parse line, but compare the output (Ok/Err vars.)
- If it's an error, force a continue (next iteration of the loop)
- Otherwise, return the num that's carried by the Ok() enum

```rust
let guess: u32 = match guess.trim().parse()
{
    Ok(num) => num,
    Err(_) => continue,
};
```
And that's that!



