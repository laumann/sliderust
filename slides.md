% Memory, ownership and lifetimes

# What's your killer Rust feature?

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

<pre>
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
Result:

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

Result:

<pre>
The number was 5
The number was 5
</pre>

* `&` is a "borrowed reference"

# Borrowing

* Borrowing prevents moving

```rust
let v = Vec::new();
let vref = &v;
work_with(v);
```

Result:

<pre>
&lt;anon&gt;:7:15: 7:16 error: cannot move out of `v` because it is borrowed
&lt;anon&gt;:7     work_with(v);
                       ^
&lt;anon&gt;:6:17: 6:18 note: borrow of `v` occurs here
&lt;anon&gt;:6     let vref = &v;
                         ^
</pre>

# Lifetimes
```rust
fn helper(x: &u32) {   // --+ Borrow exists
    println!("{}", x); //   | for duration
}                      // --+ of `helper`

fn main() {
    let mut x = 5;
	helper(&x);      // Borrow happens here
	x += 1;
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

# Thinking in scopes

```rust
let v = Vec::new();
{
	let vref = &v;       // --+
	// work with `vref`  //   | lifetime of `vref`
}                        // --+ ends here
work_with(v);

```

* Common idiom: Introduce scope to narrow lifetime

# Lifetimes
```rust
struct Data {
    x: u32
}

fn main() {
    let a = Data { x: 0 };
    let b = Data { x: 1 };

    if a.x > b.x {    // --+
		process(&a)   //   | We want to pack
	} else {          //   | all of this away
	    process(&b);  //   |
	}                 // --+
}
```

# Lifetimes
```rust
struct Data {
    x: u32
}

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
struct Data {
    x: u32
}

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


# Concurrency

```rust
let mut a = 3;

spawn(move|| { // Moving closure
    a += 1;    // executed in other thread
});
a += 1;        // error! `a` was moved
```
