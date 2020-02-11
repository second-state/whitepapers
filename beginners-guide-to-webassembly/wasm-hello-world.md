---
description: How to pass string values from JavaScript to WASM / Rust.
---

# WASM hello world

{% embed url="https://github.com/second-state/wasm-learning/tree/master/browser/hello" caption="Click on the box above to go to Github repo of the tutorial source code" %}

In the [previous tutorial](my-first-webassembly-app.md), we mentioned that a WebAssembly application typically has two parts.

* The bytecode program that runs inside the WebAssembly virtual machine
* The host application that calls the WebAssembly program

However, the lightweight WebAssembly virtual machine only supports very limited numeric data types. The host application, on the other hand, probably needs to handle complex data types. One such complex data type is the string. The string is complicated because it contains data of unknown size and of unknown structure \(i.e., encoding\). The host application cannot directly pass string data to and from WebAssembly. It must convert string values to and from numeric values and arrays. You can [read more about it here](https://medium.com/wasm/strings-in-webassembly-wasm-57a05c1ea333).

![](../.gitbook/assets/01_first_module.png)

In this tutorial, I will show you how to construct a real “hello world” that handles strings in WebAssembly. 

* The host application is written in JavaScript and runs inside a web browser. It passes strings to and from WebAssembly function calls.
* The WebAssembly bytecode program is written in Rust. It receives and returns strings as Rust String structs.

The source code of the tutorial is [here](https://github.com/second-state/wasm-learning/blob/master/browser/hello.md).

#### **Setup**

The important development tool we introduce in this example is the `wasm-pack`. It compiles Rust source code into a WebAssembly bytecode program, and then generates a JavaScript module that can interact with the WebAssembly program. The generated code takes care of input / output data encoding and management. This makes it much easier for JavaScript developer to call WebAssembly functions. Follow the steps below to install Rust and the `wasm-pack` tool.

```text
# Install Rust

$ sudo apt-get update
$ sudo apt-get -y upgrade
$ curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
$ source $HOME/.cargo/env
```

```text
# Install wasm-pack tools

$ curl https://rustwasm.github.io/wasm-pack/installer/init.sh -sSf | sh
```

#### **WebAssembly program in Rust**

In this example, our Rust program appends the input string after “hello”. Let’s create a new `cargo` project. Since this program is intended to be called from a host application, not to run as a stand-alone executable, we will create a `hello` project.

```text
$ cargo new --lib hello
$ cd hello
```

Edit the `Cargo.toml` file to add a `[lib]` section. It tells the compiler where to find the source code for the library and how to generate the bytecode output. We also need to add a dependency of `wasm-bindgen` here. It is the utility `wasm-pack` uses to generate the JavaScript binding for the Rust WebAssembly program.

```text
[lib]
name = "hello_lib"
path = "src/lib.rs"
crate-type =["cdylib"]

[dependencies]
wasm-bindgen = "0.2.50"
```

Below is the content of the Rust program `src/lib.rs`. You can actually define multiple external functions in this library file, and all of them will be available to the host JavaScript app via WebAssembly. The `#[wasm_bindgen]` tag instructs the build tools to generate communication interfaces both in Rust / WebAssembly and in the JavaScript module.

```text
use wasm_bindgen::prelude::*;

#[wasm_bindgen]
pub fn say(s: String) -> String {
  let r = String::from("hello ");
  return r + &s;
}
```

Next, you can compile the Rust source code into WebAssembly bytecode and generate the accompanying JavaScript module.

```text
$ wasm-pack build --target web
```

The results are the following two files. the `.wasm` file is the WebAssembly bytecode program, and the `.js` file is the JavaScript module.

```text
pkg/hello_lib_bg.wasm
pkg/hello_lib.js
```

#### **The JavaScript host**

Let’s switch back to the JavaScript host application. With the generated `hello_lib.js` module, it is very easy to write JavaScript to call WebAssembly functions. After the `import`, the WebAssembly `say()` function now becomes a JavaScript function with the same name. The complete web page source file is [here](https://github.com/second-state/wasm-learning/blob/master/browser/hello/html/index.html).

```text
<script type="module">
  import init, { say } from './hello_lib.js';
  async function run() {
    await init();
    var buttonOne = document.getElementById('buttonOne');
    buttonOne.addEventListener('click', function() {
      var input = $("#nameInput").val();
      alert(say(input));
    }, false);
  }
  run();
</script>
```

The `hello_lib_bg.wasm` program is loaded by the `hello_lib.js` module. Put this HTML file and the `hello_lib.js` and `hello_lib_bg.wasm` files on a web server and you can now access the web page to “say hello” to any name you enter on the page.

#### **What’s next?**

So far, we have seen how to access WebAssembly programs from JavaScript hosted in browsers. But as you know, we believe that WebAssembly’s real potential is on the server-side. From the next tutorial, we will focus on server-side WebAssembly examples. Stay tuned!

