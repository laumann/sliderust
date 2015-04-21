% Memory, ownership and lifetimes

<div style="text-align: center; margin-top: 120px">
<img src="http://www.rust-lang.org/logos/rust-logo-256x256-blk.png">
</div>

<span style="font-size: 16pt">Thomas Bracht Laumann Jespersen</span>
<br>
<span style="font-size: 14pt">March 18, 2015</span>

# What's your killer Rust feature?

Asked on [reddit.com/r/rust](https://www.reddit.com/r/rust/comments/2x0h17/whats_your_killer_rust_feature/)

 * The borrow checker!
 * It's the fourth flavor of memory management
 * _I don't have to worry about aliasing anymore_

One of the primary goals of Rust is _memory-safety_

# First: Why?

 * Why a new programming language?
 * Servo, a new experimental browser engine
 * Must have _safety_

# Memory Management (1)

* Manual: Complete control, but error-prone
* GC: Less control, but safer. Requires a runtime
* Region-based: Cyclone

# Memory Management (2)
```
void helper(int *slot) {
	printf("The number was %d\n", *slot);
	free(x);
}

int main() {
	int *slot = (int*)malloc(sizeof(int));
	*slot = 5;
	helper(slot);
	helper(slot);
}
```

# (Almost) The Same in Rust

```rust
fn helper(slot: Box<u32>) {
	println!("The number was {}", slot);
}
fn main() {
    let slot = Box::new(5); // Heap allocated
	helper(slot);           // `slot` moved here
	helper(slot);
}
```
Result:
<pre style="font-size: 18px">
&lt;anon&gt;:9:9: 9:13 error: use of moved value: `slot`
&lt;anon&gt;:9 	helper(slot);
                 &#94;~~~
&lt;anon&gt;:8:9: 8:13 note: `slot` moved here because it has type `Box<u32>`, which is non-copyable
&lt;anon&gt;:8 	helper(slot);
                 &#94;~~~
</pre>

# Ownership

* Rule #1: Exactly one owner to some allocated memory
* Compiler knows exactly when a given piece of memory can be dropped
* Ownership can be transferred

```rust
fn helper() -> Box<u32> {
	let slot = Box::new(5);
	slot
}

fn main() {
	let boxed = helper(); // Acquire ownership of return value
}
```

# Give it back!
```rust
fn helper(slot: Box<u32>) -> Box<u32> {
	println!("The number was {}", slot);
	slot
}

fn main() {
    let slot = Box::new(5);
	let slot = helper(slot);
	helper(slot);
}
```
Output:
<pre>
The number was 5
The number was 5
</pre>

# Borrowing
```rust
fn helper(slot: &Box<u32>) {
	println!("The number was {}", slot);
}

fn main() {
    let slot = Box::new(5);
	helper(&slot);
	helper(&slot);
}
```
* `&` is a "borrowed reference"

# Borrowing

* Borrowing prevents moving

```rust
let v = Vec::new();
let vref = &v;
work_with(v);
```
Output:
<pre style="font-size: 22px">
&lt;anon&gt;:7:15: 7:16 error: cannot move out of `v` because it is borrowed
&lt;anon&gt;:7     work_with(v);
                       ^
&lt;anon&gt;:6:17: 6:18 note: borrow of `v` occurs here
&lt;anon&gt;:6     let vref = &v;
                         ^
</pre>

# Borrowing

 * Also have `&mut` - a borrowed, mutable reference
 * Cannot have both `&` and `&mut` at the same time
 * At most `&mut` at any given time, but multiple `&` are fine

```rust
fn push4(v: &mut Vec<u32>) {
    v.push(4);
}

fn main() {
	let mut v = vec![1, 2, 3];
	push4(&mut v);
	println!("{:?}", v);
}
```

# Borrowing

 * All references exist for some _lifetime_
 * Borrow must not outlive owner

```
int *gimme_ref() {
	int a = 42;
	return &a;
}

int main() {
	int *a = gimme_ref();
	printf("%d\n", *a);
}
```

# Borrowing

```rust
fn gimme_ref<'a>() -> &'a u32 {
    let a = 42;
    &a
}
fn main() {
	let a: &u32 = gimme_ref();
}
```

<pre style="font-size: 18px">
&lt;anon&gt;:4:6: 4:7 error: `a` does not live long enough
&lt;anon&gt;:4     &a
              ^
&lt;anon&gt;:2:31: 5:2 note: reference must be valid for the lifetime 'a as defined on the block at 2:30...
&lt;anon&gt;:2 fn gimme_ref<'a>() -> &'a u32 {
&lt;anon&gt;:3     let a = 42;
&lt;anon&gt;:4     &a
&lt;anon&gt;:5 }
&lt;anon&gt;:3:15: 5:2 note: ...but borrowed value is only valid for the block suffix following statement 0 at 3:14
&lt;anon&gt;:3     let a = 42;
&lt;anon&gt;:4     &a
&lt;anon&gt;:5 }
</pre>


# Lifetimes
```rust
fn helper(x: &u32) {   // --+ Borrow exists
    println!("{}", x); //   | for duration
}                      // --+ of `helper`

fn main() {
    let mut x = 5;
	helper(&x);      // Borrow starts and ends here
	x += 1;          // Otherwise this wouldn't work
	helper(&x);
}
```

# Lifetimes

```rust
fn helper<'a>(x: &'a u32) {
    println!("{}", x);
}

```
* Most of the time `rustc` can infer lifetimes

# Life & times

```rust
struct Foo {
	x: &u32
}
```
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

# Thinking in scopes

```rust
let v = Vec::new();
{
	let vref = &v;       // --+
	// work with `vref`  //   | lifetime of `vref`
}                        // --+
work_with(v);

```

* Common idiom: Introduce scope to narrow lifetime

# Lifetimes
```rust
struct Data { x: u32 }

fn main() {
    let a = Data { x: 0 };
    let b = Data { x: 1 };

    if a.x > b.x {    // --+
		process(&a);  //   | We want to hide `if`
	} else {          //   | and only have one
	    process(&b);  //   | invocation of `process`
	}                 // --+
}
```

# Lifetimes
```rust
struct Data { x: u32 }

fn pick_one(a: &Data, b: &Data) -> &Data {
	if a.x > b.x { a } else { b }
}

fn main() {
    let a = Data { x: 0 };
    let b = Data { x: 1 };

	let a_or_b = pick_one(&a, &b); // Points to either `a` or `b`

	process(a_or_b);               // borrow of `a` and `b`
}                                  // lasts until end
```

# Lifetimes
```rust
struct Data { x: u32 }

fn pick_one<'a>(a: &'a Data, b: &'a Data) -> &'a Data {
	if a.x > b.x { a } else { b }
}

fn main() {
    let a = Data { x: 0 };
    let b = Data { x: 1 };

	let a_or_b = pick_one(&a, &b); // Points to either `a` or `b`

	process(a_or_b);               // borrow of `a` and `b`
}                                  // lasts until end
```

# Special lifetime: `'static`

 * `'static` is the lifetime of the entire program

```rust
let x: &'static str = "Hello, world.";
```

# Ownership model

 * Prevents data races
 * Prevents a large class of memory errors
 * Does not require GC!
 * Has a learning curve...
 * Common for learners to "fight with the borrow checker"

# Great talks on Rust

 * [Talk by Niko Matsakis](mirror.linux.org.au/pub/linux.conf.au/2014/Friday/107-The_Rust_language_memory_ownership_and_lifetimes_-_Nicholas_Matsakis.mp4) on Memory, Ownership and Lifetimes
 * [Lars Bergstrom](https://vimeo.com/120512790) An Introduction to Systems Programming with Rust
