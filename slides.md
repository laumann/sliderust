% Memory, ownership and lifetimes

<div style="text-align: center; margin-top: 120px">
<img src="http://www.rust-lang.org/logos/rust-logo-256x256-blk.png">
</div>

<span style="font-size: 16pt">Thomas Bracht Laumann Jespersen</span>
<br />
<span style="font-size: 14pt">March 18, 2015</span>

# The dangers of C++

```rust
{
  map<int, Record> m;
  m[1] = ...;
  ...
  Record &r = m[i];
  return r;
}
```

# Why Rust?

* Want to guarantee memory safety

# Goals of Rust

* Support **efficient patters**
* Prevent **dangling pointers**, **memory leaks** and **data races**
*

# Memory management

* Want control, like `malloc()` and `free()`, but...
* avoid problems such as double frees, dangling pointers, etc
* GC solves these problems, but takes away control
* The Rust Way: Ownership and borrowing

It's so easy!

```rust
fn main() {
    println!("Hello, world!");
}
```

# Rust is

a systems programming language that

* runs blazingly fast,
* prevents almost all crashes, and
* eliminates data races.

Show me more!

# A true C++ replacement

```text
error: mismatched types: expected `&'a html5ever::tokenizer::Tokenizer<html5ever::tree_builder::TreeBuilder<dom::node::TrustedNodeAddress,dom::servohtmlparser::Sink>>`, found `&core::cell::RefCell<html5ever::tokenizer::Tokenizer<html5ever::tree_builder::TreeBuilder<dom::node::TrustedNodeAddress,dom::servohtmlparser::Sink>>>`
```

# Very good then

Empty slide
