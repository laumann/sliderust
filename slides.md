% Memory, ownership and lifetimes

<div style="text-align: center; margin-top: 120px">
<img src="http://www.rust-lang.org/logos/rust-logo-256x256-blk.png">
</div>

<span style="font-size: 16pt">Thomas Bracht Laumann Jespersen</span>
<br>
<span style="font-size: 14pt">March 18, 2015</span>

# What's your killer Rust feature?

- What novel features does Rust bring?
- Asked on [reddit.com/r/rust](https://www.reddit.com/r/rust/comments/2x0h17/whats_your_killer_rust_feature/)
  * borrow checker!
  * _``I don't have to worry about aliasing anymore''_

# Memory Management (1)

* Want to guarantee memory safety
* Manual: Complete control, but error-prone
* GC: Less control, but safer. Requires a runtime
* Region-based: Cyclone

# Memory Management (2)
```
void foo() {
    int *x = (int*)malloc(sizeof(int));
	*x = 5;
	printf("%d\n", *x);
}
```

# Memory Management (3)
```
void foo() {
	int *x = (int*)malloc(sizeof(int));
	*x = 5;
	free(x);
	printf("%d\n", *x);
}

```

# Memory Management (3)
```
void foo() {
	int *x = (int*)malloc(sizeof(int));
	*x = 5;
	printf("%d\n", *x);
    free(x);
}

```

# (Almost) The Same in Rust

```rust
fn foo() {
    let x = Box::new(5); // Heap allocated
	println!("{}", x);
}
```
* `Box::new` allocates on heap
* No need for `free()`
* _lifetime_ of `x` is block of `foo`

# Ownership
```rust
fn add_one(x: Box<u32>) {
	*x += 1;
}

fn foo() {
    let x = Box::new(5);
	add_one(x);
	println!("{}", x);
}
```
Result:
<pre>
&lt;anon&gt;:8:19: 8:20 error: use of moved value: `x`
&lt;anon&gt;:8 	println!("{:?}", x);
                           &#94;
</pre>

* Rule #1: Exactly one owner to some allocated memory
* `add_one` "owns" the value.
* Ownership can be transferred


# Give it back
```rust
fn add_one(mut x: Box<u32>) -> Box<u32> {
    *x += 1;
	x
}

fn foo() {
    let x = Box::new(5);
	let y = add_one(x);
	println!("{}", y);
}
```
Result:
<pre>
6
</pre>

* Works (yay!), but is this really necessary?

# Just borrow it
```rust
fn add_one(v: &mut u32) {
    *x += 1;
}

fn foo() {
    let mut x = 5;
	add_one(&mut x);
	println!("{}", x);
}
```
* `&` is borrowed, immutable reference
* `&mut` is borrowed, mutable reference
* Enforced at compile-time
* Enough to prevent data races
* Compiler knows exactly when a given piece of memory can be dropped

# Lifetimes
```rust
fn add_one(v: &mut u32) { // -+ Borrow exists
    *x += 1;              //  | for duration
}                         // -+ of `add_one`

fn foo() {
    let mut x = 5;
	add_one(&mut x);      // Borrow happens here
	println!("{}", x);
}
```

# Lifetimes

```rust
fn add_one<'a>(v: &'a mut u32) {
    *x += 1;
}
```
* Most of the time `rustc` can infer lifetimes

# Lifetimes

```rust
struct Foo {
	x: &u32
}
```
Result:
<pre>
&lt;anon&gt;:2:8: 2:12 error: missing lifetime specifier [E0106]
&lt;anon&gt;:2     x: &amp;u32
                &#94;~~~
</pre>

Instead:

```rust
struct Foo<'a> {
	x: &'a u32
}
```

# Lifetimes
```rust
struct Foo<'a> {
	x: &'a u32
}

fn foo() {
    let y = &5;           // -+   lifetime of y starts here
	let f = Foo { x: y }; //  |-+ lifetime of f starts here
	                      //  | |
    // use values here    //  | |
}                         // -+-+ lifetimes of y and f end here

```
