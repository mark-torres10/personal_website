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

- Wow, nothing compiles. The hardest part is getting 'cargo run' to work.
- I kinda like that nothing compiles? It's really hard to make code compile, but I also feel much more secure that obvious bugs aren't going to happen.
- The compiler tells me if a variable isn't necessary or if something is set as mutable when it really shouldn't be. I really appreciate that actually; the best you can do in Python is to add static type checking but it's not enforced at the compiler level.
- String and &str aren't the same thing??? It makes sense why, and it makes it clear so you don't shoot yourself in the foot accidentally.
- The idea of ownership seems like a really great way to make explicitly clear which references and variables own which bits of memory. I definitely prefer it to C's malloc and free. But it does mean that I can't just randomly create and move around variables like I do in Python, which is probably for the best.
- Coming from Python, where eerything is mutable (even keywords), I get tripped up trying to change variables in Rust like I do in Python.
- Rust seemingly makes you be very explicit about what you're implementing. For example, you can't even print something without updating the interface to explicitly allow debug prints. In Python, if you create a new object and try to print it without adding `__repr__`, it'll at least print the address of the pointer.
- I am a HUGE fan of `uv` in Python as a form of package management and that seems to be directly inspired by Rust crates. I find `uv` to be really comprehensive, replicable, and easy to understand, superior to both pip and conda environments.
- Rust is FAST (to be fair, everything is FAST compared to Python; you don't do low-latency things in Python).
- I now know the difference between the stack and the heap.
