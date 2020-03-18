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


