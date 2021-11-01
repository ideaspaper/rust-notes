# Rust Notes - Part 3

## Contents

- [Structs](#structs)
- [Enums](#enums)
- [Match](#match)

## Structs

Struct is a custom data type. We can name and package related things together. Like tuples, the pieces of a struct can be different types. Unlike with tuples, you’ll name each piece of data so it’s clear what the values mean. You don’t have to rely on the order of the data to specify or access the values of an instance.

```rust
// defining a struct
// structs use CamelCase for naming convention
struct User {
    active: bool,
    username: String,
    email: String,
    sign_in_count: u64,
}

fn main() {
    // declaring a User instance
    let user_1 = User {
        email: String::from("someone@example.com"),
        username: String::from("someusername123"),
        active: true,
        sign_in_count: 1,
    };
    // accessing User instance's specific field
    println!("{}", user_1.email);
}
```

Rust doesn’t allow us to mark only certain fields as mutable. We have to mark the struct declaration as `mut`.

```rust
struct User {
    active: bool,
    username: String,
    email: String,
    sign_in_count: u64,
}

fn main() {
    let mut user_1 = User {
        email: String::from("someone@example.com"),
        username: String::from("someusername123"),
        active: true,
        sign_in_count: 1,
    };
    user_1.email = String::from("aperson@example.com");
}
```

### Field Init Shorthand

```rust
// without shorthand
fn build_user(email: String, username: String) -> User {
    User {
        email: email,
        username: username,
        active: true,
        sign_in_count: 1,
    }
}

// with shorthand
fn build_user(email: String, username: String) -> User {
    User {
        email,
        username,
        active: true,
        sign_in_count: 1,
    }
}
```

### Struct Update Syntax

```rust
// defining a struct
// structs use CamelCase for naming convention
struct User {
    active: bool,
    username: String,
    email: String,
    sign_in_count: u64,
}

fn main() {
    // declaring a User instance, user_1
    let user_1 = User {
        email: String::from("someone@example.com"),
        username: String::from("someusername123"),
        active: true,
        sign_in_count: 1,
    };
    
    // creating User instance, user_2, with some values taken from user_1
    let user_2 = User {
        active: user1.active,
        username: user1.username,
        email: String::from("another@example.com"),
        sign_in_count: user1.sign_in_count,
    };
    
    // creating User instance, user_3, with some values taken from user_1
    // update syntax implementation
    let user_3 = User {
        email: String::from("another@example.com"),
        ..user_1
    };
}
```

The `..user_1` must come last to specify that any remaining fields should get their values from the corresponding fields in `user_1`. Note that the struct update syntax is like assignment with `=` because it moves the data. In this example, we can no longer use `user_1` after creating `user_3` because the `String` in the `username` field of `user_1` was moved into `user_2`. If we had given `user_2` new `String` values for both `email` and `username`, and thus only used the `active` and `sign_in_count` values from `user_1`, then `user_1` would still be valid after creating `user_3`.

### Tupple Structs

Tuple structs have the added meaning the struct name provides but don’t have names associated with their fields; rather, they just have the types of the fields. Tuple structs are useful when you want to give the whole tuple a name and make the tuple be a different type from other tuples, and naming each field as in a regular struct would be verbose or redundant.

```rust
#[derive(Debug)]
struct Color(i32, i32, i32);
#[derive(Debug)]
struct Point(i32, i32, i32);

fn main() {
    let black = Color(0, 0, 0);
    let origin = Point(0, 0, 0);
    println!("{:?}", black);
    println!("{:?}", origin);
}
```

`black` is of type `Color`, `origin` is of type `Point`. Even though the count and types of the  fields are the same, `black` is not the same as `origin`. They are instances of different tupple structs.

### Unit-Like Structs

Structs that do not have any fields are called unit-like structs. They behave similiarly to `()`.

```rust
// defining a unit-like struct, SomeStruct
struct SomeStruct;
fn main() {
    // creating an instance of SomeStruct
    let a = SomeStruct;
}
```

We will use unit-like in `traits` later.

### Methods

We can define methods in struct.

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
    let rect_1 = Rectangle {
        width: 30,
        height: 50,
    };
    println!("{:#?}", rect_1); // # to make the message prettier
    println!("Area: {}", rect_1.area());
}
```

Parameter `&self` is actually a short-form of `self: &Self`. Methods must have a parameter named `self` of type `Self` for their first parameter, so Rust lets you abbreviate this with only the name `self` in the first parameter spot. Within an `impl` block, the type `Self` is an alias for the type that the `impl` block is for. Methods can take ownership of `self`, borrow `self` immutably as the code above, or borrow `self` mutably.

In the code above we don't want to take ownership, therefore we use `&self` to borrow instead. If there is a case when we need to change the valie, we can use `&mut self` as the first parameter. Having a method that takes ownership of the instance by using just `self` as the first parameter is rare; this technique is usually used when the method transforms `self` into something else and you want to prevent the caller from using the original instance after the transformation.

[back to top](#contents)

## Enums

Enumerations allow you to define a type by enumerating its **possible variants**. Each variant can hold any kind of data, including structs or enums. Writing variants are "similiar" to writing structs.

```rust
// defining an enum, Message
#[derive(Debug)]
enum Message {
    Quit,                       // variant Quit holds no data, like unit-like struct
    Move { x: i32, y: i32 },    // variant Move holds data with label, like a normal struct
    Write(String),              // variant Write holds a single String without label, like a tupple struct
    ChangeColor(i32, i32, i32), // variant ChangeColor holds three i32s data without label, like a tupple struct
}

fn main() {
    let message_1 = Message::Quit;
    let message_2 = Message::Move { x: 30, y: 30 };
    let message_3 = Message::Write(String::from("This is a message"));
    let message_4 = Message::ChangeColor(255, 0, 0);
    println!("{:#?}", message_1);
    println!("{:#?}", message_2);
    println!("{:#?}", message_3);
    println!("{:#?}", message_4);
}
```

### Methods

Just as we’re able to define methods on structs using `impl`, we're able to define methods on enums.

```rust
// defining an enum, Message
#[derive(Debug)]
enum Message {
    Quit,                       // variant Quit holds no data, like unit-like struct
    Move { x: i32, y: i32 },    // variant Move holds data with label, like a normal struct
    Write(String),              // variant Write holds a single String without label, like a tupple struct
    ChangeColor(i32, i32, i32), // variant ChangeColor holds three i32s data without label, like a tupple struct
}

impl Message {
    fn call(&self) {
        // do something here
    }
}

fn main() {
    let message_1 = Message::Quit;
    let message_2 = Message::Move { x: 30, y: 30 };
    let message_3 = Message::Write(String::from("This is a message"));
    let message_4 = Message::ChangeColor(255, 0, 0);
    println!("{:#?}", message_1);
    println!("{:#?}", message_2);
    println!("{:#?}", message_3);
    println!("{:#?}", message_4);
}
```

The `self` inside `call` method will be the value of the variable we've created. For example, when calling `message_3.call()`, the `self` would be `Message::Write(String::from("This is a message"))`.

### Option Enum

Rust doesn’t have the null feature that many other languages have. Null is a value that means there is no value there. In languages with null, variables can always be in one of two states: null or not-null. In Rust we have `Option` enum which can accomodate null concept.

```rust
// Option definition in standard library
enum Option<T> {
    None,
    Some(T),
}
```

```rust
fn main() {
    let some_number = Some(5);             // some_number type is Option<i32>, because integer type by default is i32
    let some_string = Some("a string");    // some_string type is Option<&str>
    let absent_number: Option<i32> = None; // absent_number type is Option<i32>
}
```

In case of `absent_number` we have to define what type `Some` will hold, therefore we added the type annotation.

Because `Option<T>` and `T` (where `T` can be any type) are different types, we cannot use `Option<T>` value as if it were definitely a valid value. The code below won't work.

```rust
fn main() {
    let some_number = Some(5);              // some_number type is Option<i32>, because integer type by default is i32
    let another_number = 10;                // another_number type is i32
    let sum = some_number + another_number; // will fail because of different data types
}
```

If we want do operation on `some_number` and `another_number`, first we have to convert the `Option<i32>` type to `i32`. Therefore we are "forced" to handle the case when the value is actually `None`. An example on how to convert `Option<i32>` to `i32` is as below.

```rust
fn main() {
    let some_number = Some(5);              // some_number type is Option<i32>, because integer type by default is i32
    let another_number = 10;                // another_number type is i32

    // if some_number contains None, then 0 will be used instead
    let sum = some_number.unwrap_or(0) + another_number;
    
    println!("Result: {}", sum);
}
```

More information about available methods for `Option<T>` can be found [here](https://doc.rust-lang.org/std/option/enum.Option.html).

[back to top](#contents)

## Match

Aside from `if`, Rust has another conditional flow control which is `match`. In `if` the condition has to return `bool`, but in `match` it can be any type.

```rust
enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter,
}

fn value_in_cents(coin: Coin) -> u8 {
    // match will compares a value against each pattern in order
    // if a match found, then patterns below it will be skipped
    match coin {
        // arm => code
        Coin::Penny => 1,
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter => 25,
    }
}

fn main() {
    let my_coin = Coin::Penny;
    let value = value_in_cents(my_coin);
    println!("Coin value: {} cents", value);
}
```

### Bind to Values

Match arms can bind to the parts of the values that match the pattern.

```rust
#[derive(Debug)] // so we can inspect the state in a minute
enum UsState {
    Alabama,
    Alaska,
}

enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter(UsState),
}

fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => 1,
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter(state) => {
            println!("State quarter from {:?}!", state);
            25
        }
    }
}

fn main() {
    let temp = value_in_cents(Coin::Quarter(UsState::Alaska));
}
```

When the argument of `value_in_cents` match `Coin::Quarter(state)`, `println!` will be executed. This way we can extract the inner state value out of the `Coin` enum variant for `Quarter`.

Another example with `Quarter` variant written as struct.

```rust
// snipped
enum Coin {
    // snipped
    Quarter { state: UsState },
}

fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        // snipped
        Coin::Quarter { state } => {
            println!("State quarter from {:?}!", state);
            25
        }
    }
}

fn main() {
    let temp = value_in_cents(Coin::Quarter {
        state: UsState::Alaska,
    });
}
```

### Match with Option

```rust
fn plus_one(x: Option<i32>) -> Option<i32> {
    match x {
        None => None,
        Some(i) => Some(i + 1),
    }
}

fn main() {
    let five = Some(5);
    let six = plus_one(five);
    let none = plus_one(None);    
}
```

### Matches are Exhaustive

Matches in Rust are exhaustive: we must exhaust every last possibility in order for the code to be valid.

```rust
fn plus_one(x: Option<i32>) -> Option<i32> {
    match x {
        Some(i) => Some(i + 1),
    }
}

fn main() {
    // do something here
}
```

The code above will not compile because we haven't handle the case when `x` is `None`. We can modify code above as below.

```rust
fn plus_one(x: Option<i32>) -> Option<i32> {
    match x {
        Some(i) => Some(i + 1),
        other => Some(0),
    }
}

fn main() {
    let temp = plus_one(None);
    println!("{:?}", temp);
}
```

`other` will match any `x` value and bind to it. But in case we don't need the value, we can replace it with `_`. This tells Rust we aren’t going to use the value, so Rust won’t warn us about an unused variable.

```rust
fn plus_one(x: Option<i32>) -> Option<i32> {
    match x {
        Some(i) => Some(i + 1),
        _ => Some(0),
    }
}

fn main() {
    let temp = plus_one(None);
    println!("{:?}", temp);
}
```

[back to top](#contents)
