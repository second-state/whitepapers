---
description: Call WebAssembly functions from a nodejs web application.
---

# WebAssembly and Node.js

\*\*\*\*[**This article is outdated. Please see the latest version here.**](https://cloud.secondstate.io/server-side-webassembly/getting-started)\*\*\*\*

In previous tutorials, we discussed how to access WebAssembly functions from JavaScript applications hosted inside web browsers. However, as we also noted, there are great use cases for [WebAssembly on the server-side](https://medium.com/wasm/webassembly-on-the-server-side-c584f874b4a3), especially for AI, blockchain, and big data applications. In this example, I will show you how to incorporate WebAssembly functions, written in Rust, into Node.js applications on the server. We can provide WebAssembly functions as a microservice.

The demo application is structured as follows.

* The host application is a Node.js web application written in JavaScript. It makes WebAssembly function calls.
* The WebAssembly bytecode program is written in Rust. It is called from the Node.js web application.

{% hint style="success" %}
The source code of the tutorial is [here](https://github.com/second-state/wasm-learning/tree/master/nodejs/hello-v8).
{% endhint %}

#### **Setup**

As in the previous tutorial, we use the `wasm-pack` tool to compile the Rust source code and generate the corresponding JavaScript module. The generated module makes it easy to pass complex and dynamic data between JavaScript and Rust functions. You can [learn more](https://medium.com/wasm/strings-in-webassembly-wasm-57a05c1ea333) about this issue here. Follow the steps below to install Rust and the `wasm-pack` tool.

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

Below is the content of the Rust program `src/lib.rs`. You can actually define multiple external functions in this library file, and all of them will be available to the host JaveScript app via WebAssembly.

```text
use wasm_bindgen::prelude::*;

#[wasm_bindgen]
pub fn say(s: String) -> String {
  let r = String::from("hello ");
  return r + &s;
}
```

Next, you can compile the Rust source code into WebAssembly bytecode and generate the accompanying JavaScript module for the Node.js host environment.

```text
$ wasm-pack build --target nodejs
```

The result are the following three files. the `.wasm` file is the WebAssembly bytecode program, and the `.js` files are for the JavaScript module.

```text
pkg/hello_lib_bg.wasm
pkg/hello_lib_bg.js
pkg/hello_lib.js
```

#### **The Node.js host application**

Next, let’s create a node folder for the Node.js web application. Copy over the generated JavaScript module files.

```text
$ mkdir node
$ cp pkg/hello_lib_bg.wasm node/
$ cp pkg/hello_lib_bg.js node/
$ cp pkg/hello_lib.js node/
```

With the generated `hello_lib.js` module, it is very easy to write JavaScript to call WebAssembly functions. Below is the node application `app.js`. It simply imports the `say()` function from the generated module. The node application takes the `name` parameter from incoming an HTTP GET request, and responds with “hello `name`”.

```text
const { say } = require('./hello_lib.js');

const http = require('http');
const url = require('url');
const hostname = '127.0.0.1';
const port = 8080;

const server = http.createServer((req, res) => {
  const queryObject = url.parse(req.url,true).query;
  res.statusCode = 200;
  res.setHeader('Content-Type', 'text/plain');
  res.end(say(queryObject['name']));
});

server.listen(port, hostname, () => {
  console.log(`Server running at http://${hostname}:${port}/`);
});
```

Start the Node.js application server as follows.

```text
$ node app.js
Server running at http://127.0.0.1:8080/
```

Then, you can test it.

```text
$ curl http://127.0.0.1:8080/?name=Wasm
hello Wasm
```

#### **What’s next?**

Rust developers rejoice! Web services can now offload computationally intensive, unsafe, and novel hardware access tasks to WebAssembly. More examples and use cases to come. Stay tuned!

