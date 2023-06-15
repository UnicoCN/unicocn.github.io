---
title: Stanford CS110L Notes 3
description: Stanford CS110L Notes lec03
slug: rust-cs110l-notes-3
date: 2023-06-15 00:00:00+0000
image: cover.png
categories:
    - Study
tags:
    - Rust
    - Course Notes
    - CS110L
    - Technology
---

> Code up a linked list using Rust!

Specifically, we will learn:

- How to write object-oriented rust code and how this code is situated within the paradigm of ownership and borrowing we’ve been discussing.
- How to allocate memory on the heap using the `Box` smart pointer type.
- How to do error handling in Rust.
- How to represent the idea of a null-pointer using `Option`.
- How to convert a mutable reference to an owned value using `take()`
- (if time permits) How to use traits in Rust.

## Code

```rust
struct Node {
    value : u32,
    next : Option<Box<Node>>, // next could be nullptr, use Option<>
}

pub struct LinkedList {
    head : Option<Box<Node>>,
    size : usize,
}

impl Node {
    fn new(value : u32, next : Option<Box<Node>>) -> Node{
        Node { value: value, next: next }
    }
}

impl LinkedList {
    pub fn new() -> LinkedList {
        LinkedList { head: None, size: 0 }
    }

    pub fn get_size(&self) -> usize {
        self.size
    }

    pub fn is_empty(&self) -> bool {
        self.size == 0
    }

    pub fn push(&mut self, value : u32) {
				/* why use self.head.take() ?
				* because self.head is Option<Box<Node>>, and LinkedList has its ownership
				* now we want to give self.head's ownership to Node::new()
				* so we could use .take(), to give self.head's ownership to Node::new()
				* btw, take() is for Option<>, is self.head.is_some(), take() return Some()
				* leave None to self.head, is self.head.is_none(), take() return None
				*/
        let new_node = Box::new(Node::new(value, self.head.take()));
        self.head = Some(new_node);
        self.size += 1;
    }
    
    pub fn pop(&mut self) -> Option<u32> {
        let node = self.head.take()?; // if self.head is None, return None, else continue
        self.head = node.next;
        self.size -= 1;
        Some(node.value)
    }

    pub fn display(&self) {
        let mut current = &self.head;
        let mut result = String::new();
        loop {
            match current {
                Some(node) => {
                    result = format!("{} {}", result, node.value);
                    current = &node.next;
                }, 
                None => break,
            }
        }
        println!("{}", result);
    }
}

fn main() {
    let mut list = LinkedList::new();
    for i in 1..10 {
        list.push(i);
    }
    list.display();
    list.pop();
    list.display();
    assert!(list.get_size() == 8);
}
```

## Explain: When to use unwrap | expect

Example:

First, you’ll need to open the file by calling `File::open(filename)`. `File::open` returns a `Result`, since opening the file may fail (e.g. if the filename is invalid).

You could open the file like this:

```rust
let file = File::open(filename).unwrap();
```

However, it’s generally good style to avoid calling `unwrap` or `expect` unless the error absolutely should never occur, or unless you’re writing the code in `main` or some other high-level function that won’t be reused in your program. If you use `unwrap` or `expect` in helper functions, then code might use those helper functions without realizing they could cause panics.

The idiomatic way to deal with this is to write something like the following:

```rust
let file = File::open(filename)?;
```

The [`?` operator](https://doc.rust-lang.org/edition-guide/rust-2018/error-handling-and-panics/the-question-mark-operator-for-easier-error-handling.html)  is commonly used in Rust to propagate errors without having to repeatedly write a lot of code to check a `Result`. That line of code really expands to this:

```rust
let file = match File::open(filename) {
    Ok(file) => file,
    Err(err) => return Err(err),
};
```

If the `Result` was `Ok`, then the `?` gives you the returned value, but if the result was an `Err`, it causes your function to return that `Err` (propagating the error through the call chain).

Once you have your file open, you can read the lines like so:

```rust
for line in io::BufReader::new(file).lines() {
    let line_str = line?;
    // do something with line_str*
}
```

In this code, `line` is a `Result<String, io::Error>`. We can safely unwrap its `Ok` value using `?`, as we did when opening the file. Then, you can add the string to a vector.

> Written by Jiacheng Hu, at Zhejiang University, Hangzhou, China.
