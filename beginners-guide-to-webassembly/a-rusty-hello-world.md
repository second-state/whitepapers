---
description: >-
  Getting started with the Rust programming language - Rust is the
  best-supported language on WebAssembly today.
---

# A Rusty hello world

While WebAssembly supports many programming languages, Rust by far has the best tooling. Rust is voted the most beloved programming language by StackOverflow users for the past 4 years in a row. It is one of the fastest-growing programming languages.

Rust is versatile and performant like C, but much safer than C due to its compiler design. Like C, it has a bit of a learning curve. In this tutorial, I will get you started with the Rust programming language. From here, you can learn much more about the Rust language through online resources like [the book](https://doc.rust-lang.org/book/).

#### **Install Rust**

On a typical Linux system, run the following commands to install Rust compiler and the `cargo` tool for build management.

```text
$ sudo apt-get update
$ sudo apt-get -y upgrade

$ curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
$ source $HOME/.cargo/env
```

#### **Hello world**

{% hint style="success" %}
The source code of the tutorial is [here](https://github.com/second-state/wasm-learning/tree/master/rust/hello).
{% endhint %}

First, let’s create a new project using `cargo`. 

```text
$ cargo new hello
     Created binary (application) `hello` package
$ cd hello
```

The `main()` function in the `src/main.rs` file is the entry point when we execute the Rust application. The `src/main.rs` file content is as follows. The code just prints a string “hello world” to the standard output.

```text
fn main() {
  println!(“Hello, world!”);
}
```

Next, build the binary executable file for your machine.

```text
$ cargo build --release
   Compiling hello v0.1.0 (/home/ubuntu/wasm-learning/rust/hello)
    Finished release [optimized] target(s) in 0.22s
```

You can now run your first Rust program and see “Hello World!” on the console. 

```text
$ target/release/hello
Hello, world!
```

#### **Interactive greeting**

{% hint style="success" %}
The source code of the tutorial is [here](https://github.com/second-state/wasm-learning/tree/master/rust/cli).
{% endhint %}

Again, let’s create a new project using `cargo`. 

```text
$ cargo new cli
     Created binary (application) `cli` package
$ cd cli
```

The content of the `src/main.rs` file is as follows. The `env::args()` holds the string values passed from the command line when we execute the program. Here you can also see that we first create a Rust string, and then append more string references to it. Why do we have to concatenate string references instead of string values? Well, that is how Rust makes programs safe. You can [read more here](https://doc.rust-lang.org/book/ch08-02-strings.html).

```text
use std::env;

fn main() {
    let args: Vec<String> = env::args().collect();
    println!("{}", String::from("Hello ") + &args[1]);
}
```

Next, build the binary executable file for your machine.

```text
$ cargo build --release
```

You can run the program and pass in a command-line argument. 

```text
$ target/release/cli Rust
Hello Rust
```

#### **What about WebAssembly?**

Now we have seen how to create build native executable programs from Rust source code. The executable program can only run on your build machine and could be unsafe. In the next tutorial, I will show you. 

* How to build WebAssembly bytecode programs instead of native executable files from Rust source code.
* How to interact with WebAssembly programs via a web browser instead of the cumbersome command line.

Stay tuned!

