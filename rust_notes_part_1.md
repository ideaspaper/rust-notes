# Rust Notes - Part 1

## Contents

- [Scalar Types](#scalar-types)
- [Compound Types](#compound-types)
- [Functions](#functions)
- [Code Block](#code-block)
- [If](#if)
- [Loop](#loop)
- [While](#while)
- [For](#for)

## Scalar Types

### Numbers

- Integers: `i8`, `i16`, `i32` (default), `i64`, `i128`
- Unsigned integers: `u8`, `u16`, `u32`, `u64`, `u128`
- Float: `f32` (default), `f64`

### Bool

`bool` type can only has two possible values: `true` and `false`.

### Character

`char` type value is a four bytes in size and represents a Unicode Scalar Value, which means it can represent a lot more than just ASCII.

[back to top](#contents)

## Compound Types

### Tupple

Tupple is used for grouping together a number of values with a variety of types into one compound type. Tuples have a fixed length.

```rust
fn main() {
    let x: (i32, f64, u8) = (500, 6.4, 1); // declaring a tupple
    let five_hundred = x.0; // accessing index-0
    let six_point_four = x.1; // accessing index-1
    let one = x.2; // accessing index-2
    
    let tup = (500, 6.4, 1); // declaring another tupple
    let (a, b, c) = tup; // destructuring
}
```

### Array

Every element in an array must have the same type. Arrays have a fixed length.

```rust
fn main() {
    let numbers_1 = [1, 2, 3, 4, 5]; // declaring an array of i32
    let months = [
        "January", "February","March", "April",
        "May", "June", "July", "August",
        "September", "October", "November", "December"
    ]; // declaring an array of &str
    let numbers_2: [i32; 5] = [1, 2, 3, 4, 5]; // declaring an array of i32 with explicit type and size
    let numbers_3 = [3; 5]; // [3, 3, 3, 3, 3]
    let first = numbers_1[0]; // accessing index-0 of numbers1
    let second = numbers_1[1]; // accessing index-1 of numbers1
}
```

[back to top](#contents)

## Functions

```rust
// function name uses snake_case as convention
fn function_name() {
    // your code here
}
```

### Parameters

```rust
fn multiplication(a: i32, b: i32) {
    let result = a * b;
    println!("The value of x is: {}", result);
}
```

### Return Value

```rust
fn multiplication(a: i32, b: i32) -> i32 {
    let result = a * b;
    return result;
}
```

### Function as an Expression

Rust is an **expression-based** language. Statements are instructions that perform some action and do not return a value. Expressions evaluate to a resulting value.

```rust
let num = 5;
// let num = 5; is a statement, thus does not return anything
// 5 itself is an expression
```

Function bodies are made up of a series of statements, optionally ending in an expression. Function definition is a statement. Function call is an expression. Ending an expression with `;` will turn it into a statement.

```rust
let num = add(4, 1);
// let num = add(4, 1); is a statement, thus does not return anything
// add(4, 1) itself is an expression
fn add(x: i32, y:i32) -> i32 {
    x + y // an expression that returns an i32 type value
}
```

[back to top](#contents)

## Code Block

We can create a code block with `{}`. This code block is an expression.

```rust
fn main() {
    let x = 5;
    let y = {
        let x = 3;
        x + 1
    };
    println!("The value of y is: {}", y);
}
```

[back to top](#contents)

## If

```rust
fn main() {
    let number = 3;
    // the condition must evaluates to a bool
    if number < 5 {
        println!("condition was true");
    } else {
        println!("condition was false");
    }
}
```

### If as an Expression

```rust
fn main() {
    let condition = true;
    let number = if condition { 5 } else { 6 };
    println!("The value of number is: {}", number);
    
    let condition_2 = true;
    let number_2 = if condition { 5 } else { "six" };
    // Error because the if and else arms have value types that are incompatible
    println!("The value of number is: {}", number_2);
}
```

[back to top](#contents)

## Loop

```rust
fn main() {
    let mut i = 0;
    loop {
        println!("{}", i);
        if i == 9 {
            break;
        }
        i += 1;
    }
}
```

### Label

If you have loops within loops, `break` and `continue` apply to the innermost loop at that point. You can optionally specify a loop label on a loop and then use the label with `break` or `continue` to have those keywords applied to the labeled loop instead of the innermost loop.

```rust
fn main() {
    let mut count = 0;
    'counting_up: loop {
        println!("count = {}", count);
        let mut remaining = 10;
        loop {
            println!("remaining = {}", remaining);
            if remaining == 9 {
                break;
            }
            if count == 2 {
                break 'counting_up;
            }
            remaining -= 1;
        }
        count += 1;
    }
    println!("End count = {}", count);
}
```

### Loop as an Expression

```rust
fn main() {
    let mut counter = 0;
    let result = loop {
        counter += 1;
        if counter == 10 {
            break counter * 2;
        }
    };
    println!("The result is {}", result);
}
```

[back to top](#contents)

## While

```rust
fn main() {
    let mut i = 0;
    while i < 10 {
        println!("{}", i);
        i += 1;
    }
}
```

We can use `break`, `continue` and `label` in `while`.

We cannot use `break` followed by a value in `while`.

[back to top](#contents)

## For

Looping through collection with `while`.

```rust
fn main() {
    let a = [10, 20, 30, 40, 50];
    let mut index = 0;
    while index < 5 {
        println!("the value is: {}", a[index]);
        index += 1;
    }
}
```

The code above is error prone because program will panic if the index value or test condition are incorrect.

The code above is slow because the compiler adds runtime code to perform the conditional check of whether the index is within the bounds of the array on every iteration through the loop.

We can use `for` to achive the same intended result.

```rust
fn main() {
    let a = [10, 20, 30, 40, 50];
    for element in a {
        println!("the value is: {}", element);
    }
}
```

### For with Range

```rust
fn main() {
    for i in (0..10) {
        println!("{}", i); // prints 0 to 9
    }
    for i in (0..=10) {
        println!("{}", i); // prints 0 to 10
    }
    for i in (0..10).rev() {
        println!("{}", i); // prints 9 to 0
    }
}
```

[back to top](#contents)
