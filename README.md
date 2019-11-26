# Introduction

* [Book link](https://doc.rust-lang.org/book/)
* [Rust cheatsheet](https://upsuper.github.io/rust-cheatsheet/)

* [ CHAPTER 2 - Programming a Guessing game ](https://github.com/slothyrulez/rust-book-summary/blob/master/CHAPTER_2.md)
* [ CHAPTER 3 - Common Programming Concepts ](https://github.com/slothyrulez/rust-book-summary/blob/master/CHAPTER_3.md)
* [ CHAPTER 4 - Understanding Ownership ](https://github.com/slothyrulez/rust-book-summary/blob/master/CHAPTER_4.md)
* [ CHAPTER 5 - Using Structs ](https://github.com/slothyrulez/rust-book-summary/blob/master/CHAPTER_5.md)
* [ CHAPTER 6 - Enums and Pattern matching ](https://github.com/slothyrulez/rust-book-summary/blob/master/CHAPTER_6.md)
* [ CHAPTER 7 - Managing Growing Projects with Packages, Crates, and Modules ](https://github.com/slothyrulez/rust-book-summary/blob/master/CHAPTER_7.md)
* [ CHAPTER 8 - Common Collections ](https://github.com/slothyrulez/rust-book-summary/blob/master/CHAPTER_8.md)
* [ CHAPTER 9 - Error Handling ](https://github.com/slothyrulez/rust-book-summary/blob/master/CHAPTER_9.md)
* [ CHAPTER 10 - Generic Types, Traits and Lifetimes ](https://github.com/slothyrulez/rust-book-summary/blob/master/CHAPTER_10.md)
* [ CHAPTER 11 - Writing Automated Tests ](https://github.com/slothyrulez/rust-book-summary/blob/master/CHAPTER_11.md)
* [ CHAPTER 13 - Functional Language Features: Iterators and Closures ](https://github.com/slothyrulez/rust-book-summary/blob/master/CHAPTER_13.md)
* [ CHAPTER 14 - More about Cargo and Crates.io ](https://github.com/slothyrulez/rust-book-summary/blob/master/CHAPTER_14.md)



# Chapter 15 - Smart Pointers

* A pointer is a general concept for a variable that contains an address
in memory. This address refers to, or “points at,” some other data.
* Smart pointers, on the other hand, are data structures that not only
  act like a pointer but also have additional metadata and
  capabilities.
* Some examples of smart pointers:
  - Reference counting smart pointer
  - String (metadata is capactiy and ensure that it is valid UTF-8)
  - Vec<T>

Smart pointers are usually implemented using structs. The
characteristic that distinguishes a smart pointer from an ordinary
struct is that smart pointers implement the `Deref` and `Drop` traits.

## Box<T>

* Boxes allow you to store data on the heap rather than the
  stack. What remains on the stack is the pointer to the heap data.

Usecase of Boxes:
* When you have a type whose size can’t be known at compile time.
* When you have a large amount of data and you want to transfer
  ownership but ensure the data won’t be copied when you do so
* When you want to own a value and you care only that it’s a type that
  implements a particular trait rather than being of a specific type

## Enabling recursive types with Boxes

``` rust
enum List {
    Cons(i32, Box<List>),
    Nil,
}

use crate::List::{Cons, Nil};

fn main() {
    let list = Cons(1,
        Box::new(Cons(2,
            Box::new(Cons(3,
                Box::new(Nil))))));
}
```

## Deref Trait

This program doesn't compile:

``` rust
struct MyBox<T>(T);

impl<T> MyBox<T> {
    fn new(x: T) -> MyBox<T> {
        MyBox(x)
    }
}

fn main() {
    let x = 5;
    let y = MyBox::new(x);

    assert_eq!(5, x);
    assert_eq!(5, *y); // The line which causes compile errors
}
```

This is the change required to make it compile:

``` rust
use std::ops::Deref;

impl<T> Deref for MyBox<T> {
    type Target = T;

    fn deref(&self) -> &T {
        &self.0
    }
}
```

## Implicit Deref Coercions with Functions and Methods

Deref coercion converts a reference to a type that implements `Deref`
into a reference to a type that `Deref` can convert the original type
into.

Deref coercion is a convenience that Rust performs on arguments to
functions and methods.

With deref coercion, a program like this will compile successfully:

``` rust
fn hello(name: &str) {
    println!("Hello, {}!", name);
}

fn main() {
    let m = MyBox::new(String::from("Rust"));
    hello(&m);
}
```

If you didn't have deref coercion, you have to write the above code
like this:

``` rust
fn hello(name: &str) {
    println!("Hello, {}!", name);
}

fn main() {
    let m = MyBox::new(String::from("Rust"));
    hello(&(*m)[..]);
}
```

## Deref Coercion and Mutability

Similar to how you use the `Deref` trait to override the * operator on
immutable references, you can use the `DerefMut` trait to override the *
operator on mutable references.

Rust does deref coercion when it finds types and trait implementations
in three cases:

* From `&T` to `&U` when `T: Deref<Target=U>`
* From `&mut T` to `&mut U` when `T: DerefMut<Target=U>`
* From `&mut T` to `&U` when `T: Deref<Target=U>`

The first two cases are the same except for mutability. In the third
one, Rust will also coerce a mutable reference to an immutable
one. But note that reverse is not possible.

## Drop trait

You can provide an implementation for the `Drop` trait on any type, and
the code you specify can be used to release resources like files or
network connections.

`Box<T>` customizes `Drop` to deallocate the space on the heap that
the box points to.

Example implementation:

``` rust
struct CustomSmartPointer {
    data: String,
}

impl Drop for CustomSmartPointer {
    fn drop(&mut self) {
        println!("Dropping CustomSmartPointer with data `{}`!", self.data);
    }
}

fn main() {
    let c = CustomSmartPointer { data: String::from("my stuff") };
    let d = CustomSmartPointer { data: String::from("other stuff") };
    println!("CustomSmartPointers created.");
}
```

You can also drop a value early by using `std::mem::drop`.

## Rc<T>, the Reference counted Smart Pointer

In the majority of cases, ownership is clear: you know exactly which
variable owns a given value. However, there are cases when a single
value might have multiple owners. To enable multiple ownership, Rust
has a type called `Rc<T>`.

The type `Rc<T>` provides shared ownership of a value of type T,
allocated in the heap. Invoking `clone` on `Rc` produces a new pointer to
the same value in the heap.

`Rc` uses non-atomic reference counting. This means that overhead is
very low, but an `Rc` cannot be sent between threads.

Example code:

``` rust
enum List {
    Cons(i32, Rc<List>),
    Nil,
}

use crate::List::{Cons, Nil};
use std::rc::Rc;

fn main() {
    let a = Rc::new(Cons(5, Rc::new(Cons(10, Rc::new(Nil)))));
    let b = Cons(3, Rc::clone(&a));
    let c = Cons(4, Rc::clone(&a));
}
```

## RefCell<T> and Interior mutability

* [Reddit summary on Cell and RefCell](https://www.reddit.com/r/rust/comments/755a5x/i_have_finally_understood_what_cell_and_refcell/)
* RefCell is a mutable memory location with dynamically checked borrow rules.
* Mutating the value inside an immutable value is the interior mutability pattern.

Let's actually check if it has dynamically checked borrow rules. In
Rust, that means a single variable cannot have two owners. Let's check it with `RefCell`:


``` rust
use std::cell::RefCell;

fn main() {
    let c = RefCell::new(5);
    println!("{:?}", c);
    let b = c.into_inner();
    println!("{:?}", b);
}
```


The above program works fine. But you can introduce a compile error like this:

``` rust
use std::cell::RefCell;

fn main() {
    let c = RefCell::new(5);
    println!("{:?}", c);
    let b = c.into_inner();
    println!("{:?}", b);
    println!("{:?}", c); // offending line
}
```

or like this:

``` rust
use std::cell::RefCell;

fn main() {
    let c = RefCell::new(5);
    println!("{:?}", c);
    let b = c.into_inner();
    println!("{:?}", b);
    let b = c.into_inner(); // offending line
}
```

But both the above are compile errors. What does it mean by
dynamically checked ? Let's see an example of mixing mutable and
immutable reference.

``` rust
use std::cell::RefCell;

fn main() {
    let c = RefCell::new(5);
    {
        let mut b = c.borrow_mut();
        *b = 6;
        *b = 7;
    }
    println!("{:?}", c); // prints 7
}
```

The above problem works fine. But let's have two mutable reference at
once:

``` rust
use std::cell::RefCell;

fn main() {
    let c = RefCell::new(5);
    {
        let mut b = c.borrow_mut();
        *b = 6;
        *b = 7;
        let mut d = c.borrow_mut();
        *d = 8;
    }
    println!("{:?}", c);
}
```

``` shellsession
$ ./rust4
thread 'main' panicked at 'already borrowed: BorrowMutError', src/libcore/result.rs:1084:5
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace.
```

Now that causes panic as expected. Another way to cause panic is to
mix mutable and immutable reference. Let's do that:

``` rust
use std::cell::RefCell;

fn main() {
    let c = RefCell::new(5);
    {
        let mut b = c.borrow_mut();
        *b = 6;
        *b = 7;
        let d = c.borrow();
        println!("{:?}", d);
    }
    println!("{:?}", c);
}
```

And bam, even that crashes at runtime.

[Sample usecase of RefCell<T>](https://stackoverflow.com/questions/36413364/as-i-can-make-the-vector-is-mutable-inside-struct)

## Combining Rc<T> and RefCell<T>

A common way to use RefCell<T> is in combination with Rc<T>. Recall
that Rc<T> lets you have multiple owners of some data, but it only
gives immutable access to that data. If you have an Rc<T> that holds a
RefCell<T>, you can get a value that can have multiple owners and that
you can mutate!

``` rust
#[derive(Debug)]
enum List {
    Cons(Rc<RefCell<i32>>, Rc<List>),
    Nil,
}

use crate::List::{Cons, Nil};
use std::rc::Rc;
use std::cell::RefCell;

fn main() {
    let value = Rc::new(RefCell::new(5));

    let a = Rc::new(Cons(Rc::clone(&value), Rc::new(Nil)));

    let b = Cons(Rc::new(RefCell::new(6)), Rc::clone(&a));
    let c = Cons(Rc::new(RefCell::new(10)), Rc::clone(&a));

    *value.borrow_mut() += 10;

    println!("a after = {:?}", a);
    println!("b after = {:?}", b);
    println!("c after = {:?}", c);
}
```

## Reference cycle example

``` rust
use std::rc::Rc;
use std::cell::RefCell;
use crate::List::{Cons, Nil};

#[derive(Debug)]
enum List {
    Cons(i32, RefCell<Rc<List>>),
    Nil,
}

impl List {
    fn tail(&self) -> Option<&RefCell<Rc<List>>> {
        match self {
            Cons(_, item) => Some(item),
            Nil => None,
        }
    }
}

fn main() {
    let a = Rc::new(Cons(5, RefCell::new(Rc::new(Nil))));

    println!("a initial rc count = {}", Rc::strong_count(&a));
    println!("a next item = {:?}", a.tail());

    let b = Rc::new(Cons(10, RefCell::new(Rc::clone(&a))));

    println!("a rc count after b creation = {}", Rc::strong_count(&a));
    println!("b initial rc count = {}", Rc::strong_count(&b));
    println!("b next item = {:?}", b.tail());

    if let Some(link) = a.tail() {
        *link.borrow_mut() = Rc::clone(&b);
    }

    println!("b rc count after changing a = {}", Rc::strong_count(&b));
    println!("a rc count after changing a = {}", Rc::strong_count(&a));

    // Uncomment the next line to see that we have a cycle;
    // it will overflow the stack
    // println!("a next item = {:?}", a.tail());
}
```

The reference cycle happens because of this:

```
a = 5, Nil
b = 10, a
```

Now after the initialization `let Some(link) = a.tail()`, the above
structure changes into this:

```
a = 5, b
b = 10, a
```

## Weak

Weak is a version of `Rc` that holds a non-owning reference to the
managed value. The value is accessed by calling `upgrade` on the `Weak`
pointer, which returns an `Option<Rc<T>>`.

Some experiments:

use std::rc::Rc;

``` rust
fn main() {
    let c = Rc::new(5);
    println!("{}", Rc::strong_count(&c)); // 1
    let f = Rc::clone(&c);
    println!("{}", Rc::strong_count(&c)); // 2
    println!("{}", Rc::weak_count(&c));   // 0
    let weak_f = Rc::downgrade(&c);
    println!("{}", Rc::strong_count(&c)); // 2
    println!("{}", Rc::weak_count(&c));   // 1
}
```

Usecase for Weak:

``` rust
struct Node {
    value: i32,
    parent: RefCell<Weak<Node>>,
    children: RefCell<Vec<Rc<Node>>>,
}
```

A node will be able to refer to its parent node but doesn’t own its parent.

# Chapter 16 - Fearless Concurrency

Problems writing multithreaded code:
* Race conditions, where threads are accessing data or resources in an
  inconsistent order
* Deadlocks, where two threads are waiting for each other to finish
  using a resource the other thread has, preventing both threads from
  continuing

This model where a language calls the operating system APIs to create
threads is sometimes called 1:1, meaning one operating system thread
per one language thread.

Programming language-provided threads are known as green threads, and
languages that use these green threads will execute them in the
context of a different number of operating system threads. For this
reason, the green-threaded model is called the M:N model: there are M
green threads per N operating system threads, where M and N are not
necessarily the same number.

Rust standard library only provides an implementation of 1:1
threading. But there are various libraries which provides M:N model.

## Thread Primitives

* spawn
* join

``` rust
use std::thread;
use std::time::Duration;

fn main() {
    let handle = thread::spawn(|| {
        for i in 1..10 {
            println!("hi number {} from the spawned thread!", i);
            thread::sleep(Duration::from_millis(1));
        }
    });

    for i in 1..5 {
        println!("hi number {} from the main thread!", i);
        thread::sleep(Duration::from_millis(1));
    }

    handle.join().unwrap();
}
```

You will use the `move` keyword to make the closure take ownership of
the values in threads:

``` rust
use std::thread;

fn main() {
    let v = vec![1, 2, 3];

    let handle = thread::spawn(move || {
        println!("Here's a vector: {:?}", v);
    });

    handle.join().unwrap();
}
```

The above code won't work without using `move` as you can very well
write invalid code like this:

``` rust
use std::thread;

fn main() {
    let v = vec![1, 2, 3];

    let handle = thread::spawn(|| {
        println!("Here's a vector: {:?}", v);
    });

    drop(v); // oh no!

    handle.join().unwrap();
}
```

## Message passing between Threads

One major tool Rust has for accomplishing message-sending concurrency
is the `channel`.

A channel in programming has two halves: a transmitter and a
receiver. One part of your code calls methods on the transmitter with
the data you want to send, and another part checks the receiving end
for arriving messages. A channel is said to be closed if either the
transmitter or receiver half is dropped.

``` rust
use std::thread;
use std::sync::mpsc;
use std::time::Duration;

fn main() {
    let (tx, rx) = mpsc::channel();

    thread::spawn(move || {
        let vals = vec![
            String::from("hi"),
            String::from("from"),
            String::from("the"),
            String::from("thread"),
        ];

        for val in vals {
            tx.send(val).unwrap();
            thread::sleep(Duration::from_secs(1));
        }
    });

    for received in rx {
        println!("Got: {}", received);
    }
}
```

* mpsc: multiple producer, single consumer
* tx: transmitter
* rx: receiver

## Shared state Concurrency

Mutexes are one of the concurrency primitives for shared memory.

Mutex is an abbreviation for mutual exclusion, as in, a mutex allows
only one thread to access some data at any given time. To access the
data in a mutex, a thread must first signal that it wants access by
asking to acquire the mutex’s lock. The lock is a data structure that
is part of the mutex that keeps track of who currently has exclusive
access to the data.

``` rust
use std::sync::{Mutex, Arc};
use std::thread;

fn main() {
    let counter = Arc::new(Mutex::new(0));
    let mut handles = vec![];

    for _ in 0..10 {
        let counter = Arc::clone(&counter);
        let handle = thread::spawn(move || {
            let mut num = counter.lock().unwrap();

            *num += 1;
        });
        handles.push(handle);
    }

    for handle in handles {
        handle.join().unwrap();
    }

    println!("Result: {}", *counter.lock().unwrap());
}
```

The result will be 10.

## Sync and Send Traits

The `Send` marker trait indicates that ownership of the type
implementing `Send` can be transferred between threads. Almost every
Rust type is `Send`

The `Sync` marker trait indicates that it is safe for the type
implementing `Sync` to be referenced from multiple threads. In other
words, any type T is Sync if &T (a reference to T) is Send, meaning
the reference can be sent safely to another thread.

# Chapter 17 - OOP Features of Rust

* Objects contains Data and Behavior

structs and enums have data, and impl blocks provide methods on
structs and enums. Even though structs and enums with methods aren’t
called objects, they provide the same functionality, according to the
Gang of Four’s definition of objects.

* Encapsulation that Hides Implementation Details

We can use the `pub` keyword to decide which modules, types,
functions, and methods in our code should be public, and by default
everything else is private. This provides encapsulation.

* Inheritance as a Type System and as Code Sharing

Rust doesn't have the usual inheritance property found in other OOP
langues which allows an object to inherit parent's object data and
behavior without having to define them again. But it has trait
mechanism and polymorphism to enable code reuse.

## Using Trait Objects That Allow for Values of Different Types

Objective: A library that iterates through a list of items and calls
`draw` method on each of them. Note that the some items may have been
created by the user of the library itself rather than the library.

OOP solution: Have a class named `Component` with a method named
`draw` on it. Other classes will inherit this class and may provide
custom behavior. How will Rust solve this kind of problem ?

A rust solution for the above problem:

```
pub trait Draw {
    fn draw(&self);
}

pub struct Screen {
    pub components: Vec<Box<dyn Draw>>,
}

impl Screen {
    pub fn run(&self) {
        for component in self.components.iter() {
            component.draw();
        }
    }
}
```

The above vector is of type `Box<dyn Draw>`, which is a trait object; it’s a
stand-in for any type inside a Box that implements the `Draw` trait.

You might be wondering why not a solution like this which involves
generic type and trait bounds:

```
pub struct Screen<T: Draw> {
    pub components: Vec<T>,
}

impl<T> Screen<T>
    where T: Draw {
    pub fn run(&self) {
        for component in self.components.iter() {
            component.draw();
        }
    }
}
```

The above style won't work in all scenarios. It will only work for
homogenous collections.

## Object safety is required for Trait Object

A trait is object safe if all the methods defined in the trait have
the following properties:

* The return type isn’t `Self`.
* There are no generic type parameters.

## Implementing an OODP

Desired behavior we want:

``` rust
use blog::Post;

fn main() {
    let mut post = Post::new();

    post.add_text("I ate a salad for lunch today");
    assert_eq!("", post.content());

    post.request_review();
    assert_eq!("", post.content());

    post.approve();
    assert_eq!("I ate a salad for lunch today", post.content());
}
```

The implementation for the above behavior:

``` rust
pub struct Post {
    state: Option<Box<dyn State>>,
    content: String,
}

impl Post {
    pub fn new() -> Post {
        Post {
            state: Some(Box::new(Draft {})),
            content: String::new(),
        }
    }

    pub fn approve(&mut self) {
        if let Some(s) = self.state.take() {
            self.state = Some(s.approve())
        }
    }

   pub fn add_text(&mut self, text: &str) {
        self.content.push_str(text);
    }

    pub fn content(&self) -> &str {
        self.state.as_ref().unwrap().content(&self)
    }

    pub fn request_review(&mut self) {
        if let Some(s) = self.state.take() {
            self.state = Some(s.request_review())
        }
    }


}

trait State {
 fn request_review(self: Box<Self>) -> Box<dyn State>;
 fn approve(self: Box<Self>) -> Box<dyn State>;
 fn content<'a>(&self, post: &'a Post) -> &'a str {
        ""
 }
}

struct Draft {}

impl State for Draft {
    fn request_review(self: Box<Self>) -> Box<dyn State> {
        Box::new(PendingReview {})
    }
    fn approve(self: Box<Self>) -> Box<dyn State> {
        self
    }
}

struct PendingReview {}

impl State for PendingReview {
    fn request_review(self: Box<Self>) -> Box<dyn State> {
        self
    }
    fn approve(self: Box<Self>) -> Box<dyn State> {
        Box::new(Published {})
    }
}

struct Published {}

impl State for Published {
    fn request_review(self: Box<Self>) -> Box<dyn State> {
        self
    }

    fn approve(self: Box<Self>) -> Box<dyn State> {
        self
    }

    fn content<'a>(&self, post: &'a Post) -> &'a str {
        &post.content
    }
}
```

## Chapter 18: Patterns and matching

Patterns come in two forms: refutable and irrefutable. Patterns that
will match for any possible value passed are irrefutable. An example
would be `x` in the statement `let x = 5;` because `x` matches
anything and therefore cannot fail to match. Patterns that can fail to
match for some possible value are refutable. An example would be
`Some(x)` in the expression `if let Some(x) = a_value` because if the
value in the `a_value` variable is `None` rather than `Some`, the
`Some(x)` pattern will not match.

Function parameters, let statements, and for loops can only accept
irrefutable patterns, because the program cannot do anything
meaningful when values don’t match. The if let and while let
expressions only accept refutable patterns, because by definition
they’re intended to handle possible failure: the functionality of a
conditional is in its ability to perform differently depending on
success or failure.

## Chapter 19: Advanced Features

### Unsafe Superpowers

To switch to unsafe Rust, use the `unsafe` keyword and then start a new
block that holds the unsafe code. You can take four actions in unsafe
Rust, called unsafe superpowers, that you can’t in safe Rust. Those
superpowers include the ability to:

* Dereference a raw pointer
* Call an unsafe function or method
* Access or modify a mutable static variable
* Implement an unsafe trait

#### Dereferencing a raw pointer

Raw pointers can be immutable or mutable and are written as:

* Immutable: *const T
* Mutable: *mut T

The asterisk isn’t the dereference operator; it’s part of the type
name.

Different from references and smart pointers, raw pointers:

* Are allowed to ignore the borrowing rules by having both immutable
  and mutable pointers or multiple mutable pointers to the same
  location
* Aren’t guaranteed to point to valid memory
* Are allowed to be null
* Don’t implement any automatic cleanup

Example:

``` rust
let mut num = 5;

let r1 = &num as *const i32;
let r2 = &mut num as *mut i32;

unsafe {
    println!("r1 is: {}", *r1);
    println!("r2 is: {}", *r2);
}
```

Another example which will likely lead to segmentation fault:

``` rust
let address = 0x012345usize;
let r = address as *const i32;
```

#### Calling an Unsafe Function or method

Example:

``` rust
unsafe fn dangerous() {}

unsafe {
    dangerous();
}
```

The `unsafe` keyword in this context indicates the function has
requirements we need to uphold when we call this function, because
Rust can’t guarantee we’ve met these requirements. By calling an
`unsafe` function within an unsafe block, we’re saying that we’ve read
this function’s documentation and take responsibility for upholding
the function’s contracts.

#### FFI

``` rust
extern "C" {
    fn abs(input: i32) -> i32;
}

fn main() {
    unsafe {
        println!("Absolute value of -3 according to C: {}", abs(-3));
    }
}
```

Within the `extern "C"` block, we list the names and signatures of
external functions from another language we want to call. The `"C"`
part defines which application binary interface (ABI) the external
function uses: the ABI defines how to call the function at the
assembly level. The "C" ABI is the most common and follows the C
programming language’s ABI.

#### Accessing or Modifying a Mutable Static Variable

In Rust, global variables are called static variables.

``` rust
static HELLO_WORLD: &str = "Hello, world!";

fn main() {
    println!("name is: {}", HELLO_WORLD);
}
```

In the above example the variable type is `&'static str`. Since,
static variables can only store references with the `'static`
lifetime, you don't need to annotate it explicityly.

``` rust
static mut COUNTER: u32 = 0;

fn add_to_count(inc: u32) {
    unsafe {
        COUNTER += inc;
    }
}

fn main() {
    add_to_count(3);

    unsafe {
        println!("COUNTER: {}", COUNTER);
    }
}
```

#### Implementing an Unsafe Trait

 A trait is unsafe when at least one of its methods has some invariant
 that the compiler can’t verify. We can declare that a trait is unsafe
 by adding the unsafe keyword before trait and marking the
 implementation of the trait as unsafe too.

``` rust
unsafe trait Foo {
    // methods go here
}

unsafe impl Foo for i32 {
    // method implementations go here
}
```

## Advanced Traits

### Specifying Placeholder Types in Trait Definitions with Associated Types

Associated types connect a type placeholder with a trait such that the
trait method definitions can use these placeholder types in their
signatures.

``` rust
pub trait Iterator {
    type Item;

    fn next(&mut self) -> Option<Self::Item>;
}
```

And it's implementation:

``` rust
impl Iterator for Counter {
    type Item = u32;

    fn next(&mut self) -> Option<Self::Item> {
        // --snip--
```

### Default Generic Type Parameters and Operator Overloading

When we use generic type parameters, we can specify a default concrete
type for the generic type. . The syntax for specifying a default type
for a generic type is `<PlaceholderType=ConcreteType>` when declaring
the generic type.

``` rust
trait Add<RHS=Self> {
    type Output;

    fn add(self, rhs: RHS) -> Self::Output;
}
```

If we don’t specify a concrete type for `RHS` when we implement the
`Add` trait, the type of `RHS` will default to `Self`, which will be
the type we’re implementing `Add` on.

Operator overloading is customizing the behavior of an operator (such
as +) in particular situations.

Rust doesn’t allow you to create your own operators or overload
arbitrary operators. But you can overload the operations and
corresponding traits listed in `std::ops` by implementing the traits
associated with the operator.

``` rust
use std::ops::Add;

struct Millimeters(u32);
struct Meters(u32);

impl Add<Meters> for Millimeters {
    type Output = Millimeters;

    fn add(self, other: Meters) -> Millimeters {
        Millimeters(self.0 + (other.0 * 1000))
    }
}
```

### Fully Qualified Syntax for Disambiguation: Calling Methods with the Same Name

``` rust
trait Pilot {
    fn fly(&self);
}

trait Wizard {
    fn fly(&self);
}

struct Human;

impl Pilot for Human {
    fn fly(&self) {
        println!("This is your captain speaking.");
    }
}

impl Wizard for Human {
    fn fly(&self) {
        println!("Up!");
    }
}

impl Human {
    fn fly(&self) {
        println!("*waving arms furiously*");
    }
}

fn main() {
    let person = Human;
    Pilot::fly(&person);
    Wizard::fly(&person);
    person.fly();
}
```

Example without the '&self' argument:

``` rust
trait Animal {
    fn baby_name() -> String;
}

struct Dog;

impl Dog {
    fn baby_name() -> String {
        String::from("Spot")
    }
}

impl Animal for Dog {
    fn baby_name() -> String {
        String::from("puppy")
    }
}

fn main() {
    println!("A baby dog is called a {}", Dog::baby_name()); // A baby dog is called a Spot
    println!("A baby dog is called a {}", <Dog as Animal>::baby_name()); // A baby dog is called a puppy
}
```

### Using Supertraits to Require One Trait’s Functionality Within Another Trait

Sometimes, you might need one trait to use another trait’s
functionality. In this case, you need to rely on the dependent trait
also being implemented. The trait you rely on is a supertrait of the
trait you’re implementing.

``` rust
use std::fmt;

trait OutlinePrint: fmt::Display {
    fn outline_print(&self) {
        let output = self.to_string();
        let len = output.len();
        println!("{}", "*".repeat(len + 4));
        println!("*{}*", " ".repeat(len + 2));
        println!("* {} *", output);
        println!("*{}*", " ".repeat(len + 2));
        println!("{}", "*".repeat(len + 4));
    }
}
```

`to_string` is a function implemented for `Display` trait.

### Using the Newtype Pattern to Implement External Traits on External Types

Orphan rule: We’re allowed to implement a trait on a type as long as
either the trait or the type are local to our crate.

You can overcome the above rule using the newtype pattern.

``` rust
use std::fmt;

struct Wrapper(Vec<String>);

impl fmt::Display for Wrapper {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "[{}]", self.0.join(", "))
    }
}

fn main() {
    let w = Wrapper(vec![String::from("hello"), String::from("world")]);
    println!("w = {}", w);
}
```

## Advanced Types

### Using the Newtype Pattern for Type Safety and Abstraction

Example: The `Millimeters` and `Meters` structs wrapped `u32` values
in a newtype.

### Creating Type Synonyms with Type Aliases

Rust provides the ability to declare a type alias to give an existing
type another name. For this we use the `type` keyword.

``` rust
type Kilometers = i32;
```

### The Never Type that Never Returns

Rust has a special type named `!` that’s known in type theory lingo as
the empty type because it has no values. We prefer to call it the
never type because it stands in the place of the return type when a
function will never return.

``` rust
fn bar() -> ! {
    // --snip--
}
```

Functions that return never are called diverging functions.

Example usage:

``` rust
let guess: u32 = match guess.trim().parse() {
    Ok(num) => num,
    Err(_) => continue,
};
```

The `continue` has a `!` value.

### Dynamically Sized Types and the Sized Trait

Dynamically sized types or DSTs or unsized types let us write code
using values whose size we can know only at runtime.

The following code won't compile:

``` rust
let s1: str = "Hello there!";
let s2: str = "How's it going?";
```

Rust needs to know how much memory to allocate for any value of a
particular type, and all values of a type must use the same amount of
memory. If Rust allowed us to write this code, these two str values
would need to take up the same amount of space. But they have
different lengths: s1 needs 12 bytes of storage and s2 needs 15. This
is why it’s not possible to create a variable holding a dynamically
sized type.

We make the types of s1 and s2 a &str rather than a str to make it
work. So although a `&T` is a single value that stores the memory
address of where the `T` is located, a `&str` is two values: the
address of the str and its length. As such, we can know the size of a
`&str` value at compile time: it’s twice the length of a `usize`.

To work with DSTs, Rust has a particular trait called the `Sized` trait
to determine whether or not a type’s size is known at compile time.

That is, a generic function definition like this:

``` rust
fn generic<T>(t: T) {
    // --snip--
}
```

is actually treated as though we had written this:

``` rust
fn generic<T: Sized>(t: T) {
    // --snip--
}
```

By default, generic functions will work only on types that have a
known size at compile time. However, you can use the following special
syntax to relax this restriction:

``` rust
fn generic<T: ?Sized>(t: &T) {
    // --snip--
}
```

A trait bound on `?Sized` is the opposite of a trait bound on `Sized`: we
would read this as “T may or may not be Sized.” This syntax is only
available for Sized, not any other traits.

## Advanced Functions and Closures

### Function Pointers

We can pass regular functions to functions using function
pointers. Functions coerce to the type `fn` (with a lowercase f), not
to be confused with the `Fn` closure trait. The `fn` type is called a
function pointer.

``` rust
fn add_one(x: i32) -> i32 {
    x + 1
}

fn do_twice(f: fn(i32) -> i32, arg: i32) -> i32 {
    f(arg) + f(arg)
}

fn main() {
    let answer = do_twice(add_one, 5);

    println!("The answer is: {}", answer);
}
```

Function pointers implement all three of the closure traits (Fn,
FnMut, and FnOnce), so you can always pass a function pointer as an
argument for a function that expects a closure. It’s best to write
functions using a generic type and one of the closure traits so your
functions can accept either functions or closures.

### Returning Closures

Closures are represented by traits, which means you can’t return
closures directly. A way to make it work:

``` rust
fn returns_closure() -> Box<dyn Fn(i32) -> i32> {
    Box::new(|x| x + 1)
}
```

Another way to make it work (not mentioned in the book):

``` rust
fn returns_closure() -> impl (Fn(i32) -> i32) {
    |x| x + 1
}

fn main() {
    let f = returns_closure();
    let g = f(3);
    println!("hello world");
    println!("hello world, {}", g);
}
```

## Macros

Rust has two kinds of Macros:
* Declarative macros with `macro_rules!`
* Procedural macros

There are three kinds of procedural macros:
* Custom `#[derive]` macros
* Attribute-like macros that define custom attributes usable on any item
* Function-like macros that look like function calls but operate on
  the tokens specified as their argument

# Chapter 20

This is a project and I won't be covering it here.
