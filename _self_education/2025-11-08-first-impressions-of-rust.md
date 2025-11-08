---
layout: single
title:  "First impressions of Rust as a Python dev"
date:   2025-11-08 04:00:00 +0800
classes: wide
toc: true
categories:
- self_education
- all_posts
permalink: /self_education/first-impressions-of-rust
---

# First impressions of Rust as a Python dev

I've been learning Rust for the past couple of weeks now. Here are some of my initial impressions with Rust, coming from a background of mostly Python.

### Wow, nothing compiles.

The hardest part is getting 'cargo run' to work. I kinda like that nothing compiles? It's really hard to make code compile, but I also feel much more secure that obvious bugs aren't going to happen. If I saw it happen enough times though, I can see how I'd likely come to dislike it.

The compiler tells me if a variable isn't necessary or if something is set as mutable when it really shouldn't be. I really appreciate that actually; the best you can do in Python is to add static type checking but it's not enforced at the compiler level. Again though, I can see how it can become annoying.

Even small things will stop the program from running. For example, `String` and `&str` aren't the same thing??? It makes sense why, but I'm just surprised since you can easily mutate strings and chars in Python and be lackadaisacal about how to define things and the code will still compile (though you'll hit either an explicit RunTime error or a silent bug that'll show up later, in production, with millions of users... oof...).

Because it's so hard to compile code, I see how it could be a pain. But wow this really does help prevent such a large swath of errors that you have to otherwise use static type checkers, add comprehensive unit tests, have control flow logic, include exception handling, and all the like in order to address them in Python.

### The idea of ownership seems like a really great way to make explicitly clear which references and variables own which bits of memory. 

I definitely prefer it to C's malloc and free. But it does mean that I can't just randomly create and move around variables like I do in Python, which is probably for the best.

### Rust seemingly makes you be very explicit about what you're implementing

For example, you can't even do something as basic as printing something without updating the interface to explicitly allow debug prints. In Python, if you create a new object and try to print it without adding `__repr__`, it'll at least print the address of the pointer.

The below doesn't work:

```rust
struct User {
    active: bool,
    username: String,
    email: String,
    sign_in_count: u64,
}

let mut user1 = User {
    active: true,
    username: String::from("someusername123"),
    email: String::from("someone@example.com"),
    sign_in_count: 1,
};

user1.email = String::from("anotheremail@example.com");
println!("user1: {user1}");
```

You end up with this error:

```rust
error[E0277]: `User` doesn't implement `std::fmt::Display`
   --> src/main.rs:260:23
    |
260 |     println!("user1: {user1}");
    |                      -^^^^^-
    |                      ||
    |                      |`User` cannot be formatted with the default formatter
    |                      required by this formatting parameter
    |
help: the trait `std::fmt::Display` is not implemented for `User`
```

You need to do the following:

```rust
#[derive(Debug)] // this allows us to print the struct using println
struct User {
    active: bool,
    username: String,
    email: String,
    sign_in_count: u64,
}

let mut user1 = User {
    active: true,
    username: String::from("someusername123"),
    email: String::from("someone@example.com"),
    sign_in_count: 1,
};

user1.email = String::from("anotheremail@example.com");
println!("user1: {:?}", user1);
```

This leads to our expected print statement.
```rust
user1: User { active: true, username: "someusername123", email: "anotheremail@example.com", sign_in_count: 1 }
```

- Coming from Python, where everything is mutable (even reserved keywords), I get tripped up trying to change variables in Rust like I do in Python. You have to be very clear if a variable is mutable, and things are immutable by default. It seems like it could be inconvenient, but I can see it working under the perspective of "the code author can do whatever they want, but they have to explicitly ask for it". For example, the below code doesn't work (though it would easily work in Python):

```rust
#[derive(Debug)] // this allows us to print the struct using println
struct User {
    active: bool,
    username: String,
    email: String,
    sign_in_count: u64,
}

let user1 = User {
    active: true,
    username: String::from("someusername123"),
    // a string literal (&str) is not the same as a String. &str is an 
    // immutable reference to a sequence of UTF-8 bytes, while String is a
    // growable, heap-allocated string type.
    // email: "someone@example.com",
    email: String::from("someone@example.com"),
    sign_in_count: 1,
};

// impossible here since user1 is immutable.
user1.email = String::from("anotheremail@example.com");
```

You end up getting this error.

```rust
error[E0594]: cannot assign to `user1.email`, as `user1` is not declared as mutable
   --> src/main.rs:249:5
    |
249 |     user1.email = String::from("anotheremail@example.com");
    |     ^^^^^^^^^^^ cannot assign
    |
help: consider changing this to be mutable
    |
237 |     let mut user1 = User {
    |         +++
```

I sometimes get confused how Python handles copy by value versus reference for function arguments (I think it's something like "it depends on the variable type", but I forget exactly). In Rust, the compiler doesn't allow you to mess it up. Consider, for example,

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

fn area(rectangle: &Rectangle) -> u32 {
    rectangle.width * rectangle.height
}

let rect1 = Rectangle { width: 30, height: 50 };
println!("area of rect1 is: {}", area(rect1));
```

This throws an error and makes it very clear that what should be passed in is a reference.

```rust
error[E0308]: mismatched types
   --> src/main.rs:316:43
    |
316 |     println!("area of rect1 is: {}", area(rect1));
    |                                      ---- ^^^^^ expected `&Rectangle`, found `Rectangle`
    |                                      |
    |                                      arguments to this function are incorrect
    |
note: function defined here
   --> src/main.rs:311:8
    |
311 |     fn area(rectangle: &Rectangle) -> u32 {
    |        ^^^^ ---------------------
help: consider borrowing here
    |
316 |     println!("area of rect1 is: {}", area(&rect1));
```

This applies even for something like a for-loop. For example, consider the following two loops:

```rust
let mut v = vec![1, 2, 3, 4, 5];
for i in v {
    println!("The element is {i}");
}
```

This looks fine:

```rust
The element is 1
The element is 2
The element is 3
The element is 4
The element is 5
The element is 6
```

But, as it turns out, if you iterate in this way, you're taking ownership of `v`, which means that once they fall out of scope (here, outside of the scope of the for-loop), `v` can no longer be accessed:

```rust
let mut v = vec![1, 2, 3, 4, 5];
for i in v {
    println!("The element is {i}");
}
println!("4th element of v: {:?}", v.get(3));
```

```rust
error[E0382]: borrow of moved value: `v`
   --> src/main.rs:617:40
    |
597 |     let mut v = vec![1, 2, 3, 4, 5];
    |         ----- move occurs because `v` has type `Vec<i32>`, which does not implement the `Copy` trait
...
614 |     for i in v {
    |              - `v` moved due to this implicit call to `.into_iter()`
...
617 |     println!("4th element of v: {:?}", v.get(3)); // this doesn't work bec...
    |                                        ^ value borrowed here after move
    |
note: `into_iter` takes ownership of the receiver `self`, which moves `v`
   --> /rustc/f8297e351a40c1439a467bbbb6879088047f50b3/library/core/src/iter/traits/collect.rs:310:18
    = note: borrow occurs due to deref coercion to `[i32]`
help: consider iterating over a slice of the `Vec<i32>`'s content to avoid moving into the `for` loop
    |
614 |     for i in &v {
    |              +
```

To do this, you have to explicitly borrow the elements in `v` rather than change ownership.

```rust
let mut v = vec![1, 2, 3, 4, 5];
for i in &v {
    println!("The element is {i}");
}
println!("4th element of v: {:?}", v.get(3));
```

Now the behavior works as intended:

```rust
The element is 1
The element is 2
The element is 3
The element is 4
The element is 5
4th element of v: Some(4)
```

To update the values in a vector, we can use the dereference operator to update the vector in-place:

```rust
for i in &mut v {
    *i += 50;
}
println!("v: {:?}", v);
```

```rust
v: [51, 52, 53, 54, 55]
```

In Python, this would look something like:

```python
for i in range(len(lst)):
    lst[i] += 50
```

In Rust, `for i in &mut v { *i += 50; }` mutably iterates over elements, modifying them in place. In Python, iterating directly as for i in lst: gives you the value, not a memory reference, so we'd need to use indices to update the values rather than do it in-place. Rust makes me define this sort of thing very explicitly, rather than making it easy to do and having lots of built-in default behaviors as in Python.

### You have to convert an Option<T> to a T before you can perform T operations with it.
There's no real Python equivalent for this; you'd have to use a static type checker, and even then it relies on you manually adding the correct type to your objects in the first place. I actually really like this feature, as it saves me having to double-check for null values (this ends up being an annoying feature that I have to consistently include in Python).

```rust
let x: i8 = 5;
let y: Option<i8> = Some(5);
let sum = x + y;
```

This causes the following error:

```rust
error[E0277]: cannot add `Option<i8>` to `i8`
   --> src/main.rs:415:17
    |
415 |     let sum = x + y;
    |                 ^ no implementation for `i8 + Option<i8>`
    |
    = help: the trait `Add<Option<i8>>` is not implemented for `i8`
    = help: the following other types implement trait `Add<Rhs>`:
              `&i8` implements `Add<i8>`
              `&i8` implements `Add`
              `i8` implements `Add<&i8>`
              `i8` implements `Add`
```

This is resolved if they're both of the same type.

### `panic!`

The `panic!` notation for errors is a fun name. I like that it's similar to a Python Exception, except you can't catch it or do exception handling and so the program just crashes altogether. I suppose not having exception handling fits under the theme of "specify everything", since you have to be very explicit with error and failure paths in Rust instead of masking them with exception handling like you do in Python. This combats lazy error handling, since it's easy to just have a bare `try...except` code chunk in Python without much care to the failure modes, whereas here you really have to be explicit. I actually really like this, because even though I can see how it would slow down development and force me to be more intentional, I would be more confident in the robustness of the code that I'm shipping.

For example, for the following:

```rust
let mut v = vec![1, 2, 3]; 
v.push(4);
v.push(5);
let does_not_exist = &v[100];
```

The program panicks with the following message:

```rust
thread 'main' (76070905) panicked at src/main.rs:580:28:
index out of bounds: the len is 5 but the index is 100
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

### errata

- I am a HUGE fan of `uv` in Python as a form of package management and that seems to be directly inspired by Rust crates. I find `uv` to be really comprehensive, replicable, and easy to understand, superior to both pip and conda environments.
- Rust is FAST (to be fair, everything is FAST compared to Python; you don't do low-latency things in Python).
- I now know the difference between the stack and the heap.
