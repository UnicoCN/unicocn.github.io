---
title: Stanford CS110L Notes 2
description: Stanford CS110L Notes lec03
slug: rust-cs110l-notes-2
date: 2023-06-12 10:20:00+0000
image: cover.png
categories:
    - Study
tags:
    - Rust
    - Course Notes
    - CS110L
    - Technology
---

> Rust —— A language empowering everyone to build reliable and efficient software.

## Lec03: Error Handling
### Ownership

Example One:

```rust
fn f(s: String) {
	println!("{}", s);
}

fn main() {
	let s = String::from("Hello");
	f(s); // will work
	f(s); // won't work
}
```

The problem is in the first `f(s)`, `main` has give `s` ’s ownership to `f`, and `f` doesn’t give it back. So `main` lose `s` ’s ownership in the second `f(s)`. By the way, after the first `f(s)` ends, `s` will be free.

🌟Exception：

```rust
fn om_nom_nom(param: u32) {
    println!("{}", param);
}

fn main() {
    let x = 1;
    om_nom_nom(x); // will work
    om_nom_nom(x); // will work
}
```

It works fine because `u32` implements a “copy trait” that changes what happens when it is assigned to variables or passed as a parameter. 

Good news, only primitive types + a handful of others use copy semantics, and you just need to remember those.

Example Two:

```rust
fn main() {
    let s = String::from("hello");
    let s1 = &s;
    let s2 = &s;
    println!("{} {}", s, s1);
}
```

It works fine, because `s` `s1` `s2` are all immutable.


> Remember, you can have as many read-only pointers to something as you want, as long as no one can change what is being pointed to.

🌟 Counter Example One:

```rust
fn main() {
    let s = String::from("hello");
    let s1 = &mut s; // invalid
    let s2 = &s;
    println!("{} {}", s, s1);
}
```

This fails to compile because `s` is immutable, and on the next line, we try to borrow a *mutable* reference to `s`. If this were allowed, we could modify the string using `s1`, even though it was supposed to be immutable.

🌟 Counter Example Two:

```rust
fn main() {
    let mut s = String::from("hello");
    let s1 = &mut s;
    let s2 = &s;
    println!("{} {} {}", s, s1, s2);
}
```

This fails again, but for a different reason.

- We first declare `s` as mutable. 👍
- We borrow a mutable reference to `s`. 👍
- We try to borrow an immutable reference to `s`. However, there already exists a mutable reference to `s`. Rust doesn’t allow multiple references to exist when a mutable reference has been borrowed. Otherwise, the mutable reference could be used to change (potentially reallocate) memory when code using the other references least expect it.

🌟 Counter Example Three:

```rust
fn main() {
    let mut s = String::from("hello");
    let s1 = &mut s;
    println!("{} {}", s, s1);
}
```

- We first declare `s` as mutable. 👍
- We borrow a mutable reference to `s`. 👍
- We try to use `s`. However, the value has been “borrowed out” to `s1` and hasn’t been “returned” yet. As such, we can’t use `s1`.

Example Three:

```rust
fn main() {
	let mut s = String::from("hello");
	let s1 = &mut s; // s1 borrows s here
  println!("{}", s1); // return s's ownership after this line
	println!("{}", s)
}
```

It works fine.

💭 Thinking

*“One thing that’s confusing is why sometimes I need to &var and other times I can just use var: for example, set.contains(&var), but set.insert(var) – why?"*

Answer:

When inserting an item into a set, we want to transfer ownership of that item into the set; that way, the item will exist as long as the set exists. (It would be bad if you added a string to the set, and then someone freed the string while it was still a member of the set.) However, when trying to see if the set contains an item, we want to retain ownership, so we only pass a reference.

### Error Handling about null pointer

> Introduce Option\<T\> in Rust

Null pointer is dangerous in C/C++. To solve this problem, we might want some way to indicate to the compiler when a value *might* be `NULL`, so that the compiler can then ensure code using those values is equipped to handle `NULL`.

Rust does this with the `Option` type. A value of type `Option<T>` can either be `None` or `Some(value of type T)`.

Definition of Option\<T\>:

```rust
pub enum Option<T> {
    None,
    Some(T),
}
```

Example:

```rust
fn feeling_lucky() -> Option<String> {
    if get_random_num() > 10 {
        Some(String::from("I'm feeling lucky!"))
    } else {
        None
    }
}

// how to use Option<T>?
// mothod 1, use is_none() or is_some() to check
if feeling_lucky().is_none() {
    println!("Not feeling lucky :(");
}

// method 2, use unwrap_or(default), if it's none, use default
let message = feeling_lucky().unwrap_or(String::from("Not lucky :("));

// method 3, a more idiomatical way
match feeling_lucky() {
    Some(message) => {
        println!("Got message: {}", message);
    },
    None => {
        println!("No message returned :-/");
    },
}
```

### Handling errors

> Introduce Result<T, E> in Rust

- C has an absolutely garbage system for handling errors.
- C++ and many other languages use *exceptions* to manage error conditions. It works well, but also has many disadvantages
    - failure modes are hard to spot

Rust takes a different, two-pronged approach to error handling

- unrecoverable error, use `panic!`

```rust
if sad_times() {
  panic!("Sad times!");
}
```

Panics terminate the program immediately and cannot be caught. 

(Side note: it’s technically possible to catch and recover from panics, but doing so really defeats the philosophy of error handling in Rust, so it’s not advised.)

- recoverable error

You should return a `Result`. If you return `Result<T, E>`, you can either return `Ok(value of type T)` or `Err(value of type E)`

```rust
fn poke_toddler() -> Result<&'static str, &'static str> {
    if get_random_num() > 10 {
        Ok("Hahahaha!")
    } else {
        Err("Waaaaahhh!")
    }
}

fn main() {
    match poke_toddler() {
        Ok(message) => println!("Toddler said: {}", message),
        Err(cry) => println!("Toddler cried: {}", cry),
    }
}
```

Or you could use `unwrap` instead

```rust
// Panic if the baby cries:
let ok_message = poke_toddler().unwrap();
// Same thing, but print a more descriptive panic message:
let ok_message = poke_toddler().expect("Toddler cried :(");
```

If the `Result` was `Ok`, `unwrap()` returns the success value; otherwise, it causes a panic. 

`expect()` does the same thing, but prints the supplied error message when it panics

> Written by Jiacheng Hu, at Zhejiang University, Hangzhou, China.
