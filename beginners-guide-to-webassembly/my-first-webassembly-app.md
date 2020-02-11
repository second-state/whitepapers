---
description: 'Create a simple WebAssembly app in Rust, and then call it from JavaScript!'
---

# My first WebAssembly app

{% embed url="https://github.com/second-state/wasm-learning/tree/master/browser/triple" caption="Click on the box above to go to Github repo of the tutorial source code" %}

In this tutorial, we will create a very simple but complete WebAssembly application. A WebAssembly application typically has two parts.

* The bytecode program that runs inside the WebAssembly virtual machine to perform computing tasks
* The host application that provides UI, networking, database, and calls the WebAssembly program to perform key computing tasks or business logic

In this tutorial, the host application is written in JavaScript and runs inside a web browser. The source code of the tutorial is [here](https://github.com/second-state/wasm-learning/blob/master/browser/triple.md). The WebAssembly bytecode program is written in Rust. Now, let’s check out the Rust program. 

#### **WebAssembly program in Rust**

In this example, our Rust program simply triples an input number and returns the result. Let’s first install WebAssembly tools to the Rust compiler. 

```text
# Install Rust

$ sudo apt-get update
$ sudo apt-get -y upgrade
$ curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
$ source $HOME/.cargo/env
```

```text
# Install WebAssembly tools

$ rustup target add wasm32-wasi
$ rustup override set nightly
$ rustup target add wasm32-wasi --toolchain nightly
```

Next, create a new `cargo` project. Since this program is intended to be called from a host application, not to run as a stand-alone executable, we will create a `lib` project.

```text
$ cargo new --lib triple
$ cd triple
```

Edit the `Cargo.toml` file to add a `[lib]` section. It tells the compiler where to find the source code for the library and how to generate the bytecode output.

```text
[lib]
name = "triple_lib"
path = "src/lib.rs"
crate-type =["cdylib"]
```

Below is the content of the Rust program `src/lib.rs`. You can actually define multiple external functions in this library file, and all of them will be available to the host JaveScript app via WebAssembly. 

```text
#[no_mangle]
pub extern fn triple(x: i32) -> i32 {
  return 3 * x;
}
```

Next, you can compile the Rust source code into WebAssembly bytecode using the command below. 

```text
$ cargo +nightly build --target wasm32-wasi --release
```

The WebAssembly bytecode file is `target/wasm32-wasi/release/triple_lib.wasm`.

#### **The JavaScript host**

We use JavaScript to load the WebAssembly bytecode program and call its functions. Since most web browsers already support WebAssembly, this JavaScript can actually run as a web page. Without further ado, here is the relevant part of a JavaScript module to load, export, and call WebAssembly functions. The complete web page source file is [here](https://github.com/second-state/wasm-learning/blob/master/browser/triple/html/index.html). 

```text
<script>
  if (!('WebAssembly' in window)) {
    alert('you need a browser with wasm support enabled :(');
  }
  (async () => {
      const response = await fetch('triple_lib.wasm');
      const buffer = await response.arrayBuffer();
      const module = await WebAssembly.compile(buffer);
      const instance = await WebAssembly.instantiate(module);
      const exports = instance.exports;
      const triple = exports.triple;
      
      var buttonOne = document.getElementById('buttonOne');
      buttonOne.value = 'Triple the number';
      buttonOne.addEventListener('click', function() {
        var input = $("#numberInput").val();
        alert(input + ' tripled equals ' + triple(input));
      }, false);
  })();
</script>
```

You can see that the JavaScript code loads the `triple_lib.wasm` file in the WebAssembly virtual machine, exports its external functions, and then calls the functions as needed.

Put this HTML file and the `triple_lib.wasm` file on a web server and you can now access the web page to triple any number you enter on the page.

#### **What about strings?**

Now, you have noticed that this example is not really a hello world. The WebAssembly function computes numbers but does not manipulate strings as a real hello world would do. Why is that? We will answer in the next tutorial and give a real hello world example. 

