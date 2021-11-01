# Rust Notes - Part 2

## Contents

- [Stack and Heap](#stack-and-heap)
- [Ownership](#ownership)
- [References and Borrowing](#references-and-borrowing)
- [Slice Type](#slice-type)

## Stack and Heap

- Both stack and heap are available at runtime.

- Stack is **last in, first out**. Adding data is called pushing onto the stack, and removing data is called popping off the stack.

- All data stored on the **stack** must have a **known, fixed size**.

- Data with an **unknown size** at compile time or a size that might change **must be stored on the heap**.

- When you put data on the heap, you request a certain amount of space. The memory allocator finds an empty spot in the heap that is big enough, marks it as being in use, and returns a pointer, which is the address of that location. This process is called allocating on the heap and is sometimes abbreviated as just allocating.

- Pushing to the stack is faster than allocating on the heap because the allocator never has to search for a place to store new data; that location is always at the top of the stack.

- When your code calls a function, the values passed into the function (including, potentially, pointers to data on the heap) and the function’s local variables get pushed onto the stack. When the function is over, those values get popped off the stack.

- ```rust
  fn main() {
      let string_literal = "some string value"; // stored into stack, since its size is known and fixed
      let string = String::from("some string value"); // stored into heap, since its size is not fixed
  }
  ```

[back to top](#contents)

## Ownership

### Rules

- Each value in Rust has a variable that’s called its owner.
- There can only be one owner at a time.
- When the owner goes out of scope, the value will be dropped.

### Scope

```rust
fn main() {
    // string literal will be stored in stack
    {                    // s is not valid here, it’s not yet declared
        let s = "hello"; // s is valid from this point forward
    }                    // this scope is now over, and s is no longer valid
    
    // String will be stored in heap
    {                                  // s is not valid here, it’s not yet declared
        let s = String::from("hello"); // s is valid from this point forward
    }                                  // this scope is now over, and s is no longer valid
}
```

Since `String` lives in the heap, we have to dealocate the memory when we are finished with it. In Java this will be done by garbage collector. In C, programmers have to dealocate the memory manually. In Rust, memory dealocation (drop) will be done when the variable goes out of scope.

### Move

```rust
fn main() {
    let x = 5;
    let y = x;
    
    let s1 = String::from("hello");
    let s2 = s1;
}
```

Variables which have simple values and known fixed size can be copied seamlessly. But in this case, a `String` is not as simple as integer. The `let s2 = s1;` is only copying the pointer `s1` to `s2`. The data that persist in memory (pointed by `s1`) will not be coppied. This concept is almost the same as shallow copy.

Since `s1` and `s2` are the same pointer, refering to the same memory, it will be a trouble when those two variables goes out of scope. Rust will drop the same memory twice that can lead to memory corruption. So instead of shallow copy, in Rust `let s2 = s1;` will be called as move. Once that line get executed, `s1` will be no longer valid. Therefore, Rust only needs to drop `s2`. Try the code below.

```rust
fn main() {
    let s1 = String::from("hello");
    let s2 = s1;
    println!("{}, world!", s1);
}
```

### Ownership and Functions

Calling a function with arguments has the same effect as assigning a value to a variable. Therefore, passing a variable as argument will move or copy it.

```rust
fn main() {
    let s = String::from("hello");  // s comes into scope

    takes_ownership(s);             // s's value moves into the function...
                                    // ... and so is no longer valid here

    let x = 5;                      // x comes into scope

    makes_copy(x);                  // x would move into the function,
                                    // but i32 is Copy, so it's okay to still
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

Returning values can also transfer ownership.

```rust
fn main() {
    let s1 = gives_ownership();         // gives_ownership moves its return
                                        // value into s1

    let s2 = String::from("hello");     // s2 comes into scope

    let s3 = takes_and_gives_back(s2);  // s2 is moved into
                                        // takes_and_gives_back, which also
                                        // moves its return value into s3
} // Here, s3 goes out of scope and is dropped. s2 was moved, so nothing
  // happens. s1 goes out of scope and is dropped.

fn gives_ownership() -> String {             // gives_ownership will move its
                                             // return value into the function
                                             // that calls it

    let some_string = String::from("yours"); // some_string comes into scope

    some_string                              // some_string is returned and
                                             // moves out to the calling
                                             // function
}

// This function takes a String and returns one
fn takes_and_gives_back(a_string: String) -> String { // a_string comes into
                                                      // scope

    a_string  // a_string is returned and moves out to the calling function
}
```

[back to top](#contents)

## References and Borrowing

When we call a function with arguments, we are transfering the ownership of the variables. Therefore, if we want to use the same variables again after the function end, we have to return the ownership. Taking ownership and then returning ownership with every function is a bit tedious. What if we want to let a function use a value but not take ownership? The answer to that is by using references.

```rust
fn main() {
    let s1 = String::from("hello");
    let len = calculate_length(&s1);
    println!("The length of '{}' is {}.", s1, len);
}

fn calculate_length(s: &String) -> usize { // s is a reference to a String
    s.len()
} // Here, s goes out of scope. But because it does not have ownership of what
  // it refers to, nothing happens.
```

The `&s1` syntax lets us create a reference that refers to the value of `s1` but does not own it. Because it does not own it, the value it points to will not be dropped when the reference stops being used. This is called borrowing.

### Mutable References

Just as variables are immutable by default, so are references. We’re not allowed to modify something we have a reference to. If we want to be able to mutate a variable through a reference, we have to use `&mut`. But the variable itself should be mutable in the first place.

```rust
fn main() {
    let mut s = String::from("hello");
    change(&mut s);
}

fn change(some_string: &mut String) {
    some_string.push_str(", world");
}
```

We **can only have one mutable reference to a particular piece of data at a time**. The code below will throw an error.

```rust
fn main() {
    let mut s = String::from("hello");
    let r1 = &mut s;
    let r2 = &mut s;
    println!("{}, {}", r1, r2);
}
```

The first mutable borrow is in `r1` and must last until it’s used in the `println!`, but between the creation of that mutable reference and its usage, we tried to create another mutable reference in `r2` that borrows the same data as `r1`. A trick is to use `{}` to create a new scope.

```rust
fn main() {
    let mut s = String::from("hello");
    {
        let r1 = &mut s;
    } // r1 goes out of scope here, so we can make a new reference with no problems.
    let r2 = &mut s;
}
```

The code below will also throw an error because users of an immutable reference don’t expect the values to suddenly change out from under them.

```rust
fn main() {
    let mut s = String::from("hello");
    let r1 = &s; // no problem
    let r2 = &s; // no problem
    let r3 = &mut s; // BIG PROBLEM
    println!("{}, {}, and {}", r1, r2, r3);
}
```

A reference’s scope starts from where it is introduced and continues through the last time that reference is used.

```rust
fn main() {
    let mut s = String::from("hello");
    let r1 = &s; // no problem
    let r2 = &s; // no problem
    println!("{} and {}", r1, r2);
    // variables r1 and r2 will not be used after this point
    let r3 = &mut s; // no problem
    println!("{}", r3);
}
```

### Conclusions

- At any given time, you can have either one mutable reference or any number of immutable references.
- References must always be valid.

[back to top](#contents)

## Slice Type

Slice is one of the data types that does not have ownership. Slices let you reference a contiguous sequence of elements in a collection rather than the whole collection.

```rust
fn main() {
    let s = String::from("hello world");
    let hello = &s[0..5]; // hello
    let world = &s[6..11]; // world
    
    let s2 = String::from("hello");
    let slice21 = &s2[0..2]; // he
    let slice22 = &s2[..2]; // he
    
    let s = String::from("hello");
    let len = s.len();
    let slice = &s[3..len]; // lo
    let slice = &s[3..]; // lo
}
```

[More](https://doc.rust-lang.org/book/ch04-03-slices.html)

[back to top](#contents)
