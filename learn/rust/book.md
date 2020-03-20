https://doc.rust-lang.org/

# Installation

```
$ curl https://sh.rustup.rs -sSf | sh
$ rustup update
$ rustup doc
```

# Cargo

```
$ cargo new hello_cargo
$ cargo build
$ cargo run
$ cargo check
$ cargo build --release
```

# Basics

Rust code uses snake case as the conventional style for function and variable names.

## Variables

By default variables are immutable.
You can make them mutable by adding `mut` in front of the variable name.

You declare constants using the `const` keyword instead of the `let` keyword, and the type of the value must be annotated.
Constants can be declared in any scope, including the global scope, which makes them useful for values that many parts of code need to know about.
The last difference is that constants may be set only to a constant expression, not the result of a function call or any other value that could only be computed at runtime.

## Data Types

### Scalar Types

* Integer: `i8, u8, i16, u16, i32 (default), u32, i64, u64, i128, u128, isize, usize`
* Floating-point: `f32, f64 (default)`
* Boolean: `bool`
* Character: `char`

### Compound Types

* Tuple: `let tup: (i32, f64, u8) = (500, 6.4, 1); let first = tup.0;`
* Array: `let a = [1, 2, 3, 4, 5]; let first = a[0];` or `let a: [i32; 5] = [1, 2, 3, 4, 5];`

## Functions

```rust
fn plus_one(x: i32) -> i32 {
    x + 1 // shortcut for return x + 1;
}
```

## Control Flow

```rust
// if
let number = 6;
if number % 4 == 0 {
    println!("div by 4");
} else if number % 3 == 0 {
    println!("div by 3");
} else {
    println!("other");
}

// if in let
let number2 = if condition { 5 } else { 6 };

// loop
loop { println!("again!"); }
// breaking and returning from loop
let mut counter = 0;
let result = loop {
    counter += 1;
    if counter == 10 {
        break counter * 2;
    }
};

// while
while condition { ...  }

// for
let a = [...]
for element in a.iter() { ...  }
for number in (1..4).rev() { ... }
```

# Ownership

## `Copy` types

This will compile because stack-only data types are `Copy` types:

```rust
let x = 5;
let y = x;
println!("{}, {}", x, y)
```

Whereas this won't:

```rust
let s = String::from("hello");
let s2 = s;
println!("{} {}", s, s1);
                  ^ value used here after move
```

Use `let s2 = s.clone()` if you want to duplicate the string.

## Ownership and Functions

```rust
fn main() {
    let s = String::from("hello");  // s comes into scope

    takes_ownership(s);             // s's value moves into the function...
                                    // ... and so is no longer valid here

    let x = 5;                      // x comes into scope

    makes_copy(x);                  // x would move into the function,
                                    // but i32 is Copy, so it’s okay to still
                                    // use x afterward

} // Here, x goes out of scope, then s. But because s's value was moved, nothing
  // special happens.

fn takes_ownership(some_string: String) { // some_string comes into scope
    println!("{}", some_string);
} // Here, some_string goes out of scope and `drop` is called. The backing
  // memory is freed.

fn makes_copy(some_integer: i32) { // some_integer comes into scope
    println!("{}", some_integer);
} // Here, some_integer goes out of scope. Nothing special happens.
```

## Return Values and Scope

```rust
fn main() {
    let s1 = gives_ownership();         // gives_ownership moves its return
                                        // value into s1

    let s2 = String::from("hello");     // s2 comes into scope

    let s3 = takes_and_gives_back(s2);  // s2 is moved into
                                        // takes_and_gives_back, which also
                                        // moves its return value into s3
} // Here, s3 goes out of scope and is dropped. s2 goes out of scope but was
  // moved, so nothing happens. s1 goes out of scope and is dropped.

fn gives_ownership() -> String {             // gives_ownership will move its
                                             // return value into the function
                                             // that calls it

    let some_string = String::from("hello"); // some_string comes into scope

    some_string                              // some_string is returned and
                                             // moves out to the calling
                                             // function
}

// takes_and_gives_back will take a String and return one
fn takes_and_gives_back(a_string: String) -> String { // a_string comes into
                                                      // scope

    a_string  // a_string is returned and moves out to the calling function
}
```

## References

```rust
let s1 = String::from("hello");
let len = calculate_length(&s1);
fn calculate_length(s: &String) -> usize { // s is a reference to a String
    s.len()
} // Here, s goes out of scope. But because it does not have ownership of what
  // it refers to, nothing happens.
```

### Mutable references

```rust
let mut s = String::from("hello");

let r1 = &mut s;
let r2 = &mut s; // won't compile (cannot borrow s as mutable more than once at a time

println!("{}, {}", r1, r2);
```

## Slices

```rust
let s = String::from("hello world");

let hello : &str = &s[0..5];
let world = &s[6..11];
```

String literals are slices pointing to a specific point in the binary.

```rust
let a = [1, 2, 3];
let slice = &a[1..2]; // type is &[i32]
```

# Struct

```rust
// declaration
struct User {
    username: String,
    email: String,
    sign_in_count: u64,
    active: bool,
}

// immutable variable
let user1 = User {
    email: String::from("someone@example.com"),
    username: String::from("someusername123"),
    active: true,
    sign_in_count: 1,
};
// mutable
let mut user1 = User {
    email: String::from("someone@example.com"),
    username: String::from("someusername123"),
    active: true,
    sign_in_count: 1,
};
user1.email = String::from("anotheremail@example.com");

// Field init shorthand + Struct update syntax
let email = "toto@titi";
let username = "someOtherUsername";
let user2 = User {
    username, // field init shorthand
    email, // idem,
    ..user1 //struct update syntax
}
```

## Tuple Structs

```rust
struct Color(i32, i32, i32);
struct Point(i32, i32, i32);

let black = Color(0, 0, 0);
let origin = Point(0, 0, 0);
```

## Methods

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }
}

fn main() {
    let rect1 = Rectangle { width: 30, height: 50 };

    println!(
        "The area of the rectangle is {} square pixels.",
        rect1.area()
    );
}
```

# Enum

```rust
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}

impl Message {
    fn call(&self) {
        // method body would be defined here
    }
}

let m = Message::Write(String::from("hello"));
m.call();
```

## `Option<T>`

```rust
enum Option<T> {
    Some(T),
    None,
}
```

## `match`ing

```rust
enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter,
}

fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => {
            println!("Lucky penny!");
            1
        },
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter => 25,
    }
}

// _ placeholder
let some_u8_value = 0u8;
match some_u8_value {
    1 => println!("one"),
    3 => println!("three"),
    5 => println!("five"),
    7 => println!("seven"),
    _ => (),
}
```

## Concise `if let`

This

```rust
let some_u8_value = Some(0u8);
match some_u8_value {
    Some(3) => println!("three"),
    _ => (),
}
```

can be simplified as

```rust
if let Some(3) = some_u8_value {
    println!("three");
}
```

# Packages, Crates, Modules and Paths

* **Packages**: A Cargo feature that lets you build, test, and share crates
* **Crates**: A tree of modules that produces a library or executable
* **Modules** and **use**: Let you control the organization, scope, and privacy of paths
* **Paths**: A way of naming an item, such as a struct, function, or module

## Modules

```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

pub fn eat_at_restaurant() {
    // Absolute path
    crate::front_of_house::hosting::add_to_waitlist();

    // Relative path
    front_of_house::hosting::add_to_waitlist();
}

// super keyword
fn serve_order() {}

mod back_of_house {
    fn fix_incorrect_order() {
        cook_order();
        super::serve_order();
    }

    fn cook_order() {}
}
```

## `use` Keyword

```rust
use std::fmt::Result;
use std::io::Result as IoResult;

// re-exporting names
pub use crate::front_of_house::hosting;

// nested paths
use std::{cmp::Ordering, io};
use std::io::{self, Write};

// glob operator
use std::collections::*;
```

# Collections

## Vectors

```rust
let v: Vec<i32> = Vec::new();
let v = vec![1, 2, 3];

let mut v = Vec::new();
v.push(5);

let third: &i32 = &v[2];
println!("The third element is {}", third);

match v.get(2) {
    Some(third) => println!("The third element is {}", third),
    None => println!("There is no third element."),
}

// iterating over values
let v = vec![100, 32, 57];
for i in &v {
    println!("{}", i);
}
let mut v = vec![100, 32, 57];
for i in &mut v {
    *i += 50;
}
```

## Strings (UTF-8)

```rust
let mut s = String::new();
let s = "initial contents".to_string(); // string literals derive the Display trait

// concatenation
let s1 = String::from("Hello, ");
let s2 = String::from("world!");
let s3 = s1 + &s2; // note s1 has been moved here and can no longer be used
let s1 = String::from("tic");
let s2 = String::from("tac");
let s3 = String::from("toe");
let s = format!("{}-{}-{}", s1, s2, s3);

// getting element with [] of slices is impossible (or not simple)
// iterating is done with
for c in "नमस्ते".chars() {
    println!("{}", c);
}
// will print न म स ्त े
for b in "नमस्ते".bytes() {
    println!("{}", b);
}
// will print 224 164 ...
```

## Maps

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();

scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Yellow"), 50);

let teams  = vec![String::from("Blue"), String::from("Yellow")];
let initial_scores = vec![10, 50];

let scores: HashMap<_, _> = teams.iter().zip(initial_scores.iter()).collect();

```

For types that implement the `Copy` trait, like `i32`, the values are copied into
the hash map. For owned values like String, the values will be moved and the
hash map will be the owner of those values.

```rust
let mut scores = HashMap::new();

scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Yellow"), 50);

let team_name = String::from("Blue");
let score = scores.get(&team_name); // -> Some(&10)

// iterating over key/value pairs
for (key, value) in &scores {
    println!("{}: {}", key, value);
}

// updating map
scores.insert(String::from("Blue"), 25); // will overwrite previously set 10
scores.entry(String::from("Blue")).or_insert(50); // will not overwrite and return a reference to the value (25)
```

# Error Handling

## `panic!` Macro

```rust
fn main() {
    panic!("crash and burn");
}
```

## `Result` Enum

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}

use std::fs::File;

fn main() {
    let f = File::open("hello.txt");

    let f = match f {
        Ok(file) => file,
        Err(error) => {
            panic!("Problem opening the file: {:?}", error)
        },
    };
}

// Different errors ?
use std::fs::File;
use std::io::ErrorKind;

fn main() {
    let f = File::open("hello.txt");

    let f = match f {
        Ok(file) => file,
        Err(error) => match error.kind() {
            ErrorKind::NotFound => match File::create("hello.txt") {
                Ok(fc) => fc,
                Err(e) => panic!("Problem creating the file: {:?}", e),
            },
            other_error => panic!("Problem opening the file: {:?}", other_error),
        },
    };
}
```

### Panic on Error: `unwrap` and `expect`

If the `Result` value is the `Ok` variant, `unwrap` will return the value inside the `Ok`. If the `Result` is the `Err` variant, `unwrap` will call the `panic!` macro for us.
Another method, `expect`, which is similar to `unwrap`, lets us also choose the `panic!` error message.

```rust
let f = File::open("hello.txt").unwrap();
// thread 'main' panicked at 'called `Result::unwrap()` on an `Err` value: Error {
// repr: Os { code: 2, message: "No such file or directory" } }',
// src/libcore/result.rs:906:4

let f = File::open("hello.txt").expect("Failed to open hello.txt");
// thread 'main' panicked at 'Failed to open hello.txt: Error { repr: Os { code:
// 2, message: "No such file or directory" } }', src/libcore/result.rs:906:4
```

### Propagating Errors

```rust
fn read_username_from_file() -> Result<String, io::Error> {
    let f = File::open("hello.txt");

    let mut f = match f {
        Ok(file) => file,
        Err(e) => return Err(e),
    };

    let mut s = String::new();

    match f.read_to_string(&mut s) {
        Ok(_) => Ok(s),
        Err(e) => Err(e),
    }
}
```

`?` operator as a shortcut:

```rust
fn read_username_from_file() -> Result<String, io::Error> {
    let mut f = File::open("hello.txt")?;
    let mut s = String::new();
    f.read_to_string(&mut s)?;
    Ok(s)
}
// shorter
fn read_username_from_file() -> Result<String, io::Error> {
    let mut s = String::new();
    File::open("hello.txt")?.read_to_string(&mut s)?;
    Ok(s)
}
// even shorter ^^
fn read_username_from_file() -> Result<String, io::Error> {
    fs::read_to_string("hello.txt")
}
```

# Generics

## Generic Types

```rust
// functions
fn largest<T>(list: &[T]) -> T {...}

// structs
struct Point<T> {
    x: T,
    y: T,
}
struct Point<T, U> {
    x: T,
    y: U,
}

// enums
enum Option<T> {
    Some(T),
    None,
}
enum Result<T, E> {
    Ok(T),
    Err(E),
}

// methods
impl<T> Point<T> {
    fn x(&self) -> &T {
        &self.x
    }
}
```

## Traits

A `trait` is like an `interface` in other programming languages.

```rust
// Defining
pub trait Summary {
    fn summarize(&self) -> String;
}

// Implementing
pub struct NewsArticle {
    pub headline: String,
    pub location: String,
    pub author: String,
    pub content: String,
}
impl Summary for NewsArticle {
    fn summarize(&self) -> String {
        format!("{}, by {} ({})", self.headline, self.author, self.location)
    }
}
pub struct Tweet {
    pub username: String,
    pub content: String,
    pub reply: bool,
    pub retweet: bool,
}
impl Summary for Tweet {
    fn summarize(&self) -> String {
        format!("{}: {}", self.username, self.content)
    }
}

// Default implementations
pub trait Summary {
    fn summarize_author(&self) -> String;

    fn summarize(&self) -> String {
        format!("(Read more from {}...)", self.summarize_author())
    }
}

// As parameters
pub fn notify(item: impl Summary) {
// Trait bound syntax
pub fn notify<T: Summary>(item1: T, item2: T) {
// + syntax
pub fn notify(item: impl Summary + Display) {
pub fn notify<T: Summary + Display>(item: T) {
// where clauses
fn some_function<T, U>(t: T, u: U) -> i32
    where T: Display + Clone,
          U: Clone + Debug
{

// Return type : only works when returning a single type implementing the trait
fn returns_summarizable() -> impl Summary {
```

## Lifetimes

```rust
&i32        // a reference
&'a i32     // a reference with an explicit lifetime
&'a mut i32 // a mutable reference with an explicit lifetime

fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

### Elision rules

1. each parameter that is a reference gets its own lifetime parameter
1. if there is exactly one input lifetime parameter, that lifetime is assigned to all output lifetime parameters
1. if there are multiple input lifetime parameters, but one of them is `&self` or `&mut self` because this is a method, the lifetime of `self` is assigned to all output lifetime parameters

## All Together

```rust
use std::fmt::Display;

fn longest_with_an_announcement<'a, T>(x: &'a str, y: &'a str, ann: T) -> &'a str
    where T: Display
{
    println!("Announcement! {}", ann);
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```
