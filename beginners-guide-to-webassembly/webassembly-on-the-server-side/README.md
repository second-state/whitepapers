---
description: Run high performance Rust code safely inside Node.js
---

# WebAssembly on the server-side

There are great use cases for [WebAssembly on the server-side](https://medium.com/wasm/webassembly-on-the-server-side-c584f874b4a3), especially for AI, blockchain, and big data applications. In this tutorial, I will show you how to incorporate WebAssembly functions, written in Rust, into Node.js applications on the server. We can therefore provide WebAssembly functions as a microservice \(FaaS\).

In this tutorial, we use the Second State Virtual Machine \(SSVM\) , an open source WebAssembly runtime optimized for server-side applications, together with Node.js. The SSVM provides not only a WebAssembly runtime in Node.js, but also a compiler toolchain for Rust and JavaScript.

> While Node.js comes with a default WebAssmebly runtime inside its V8 JavaScript engine, V8 is not designed to handle the performance and complex integration requirements of server-side applications. Compared with V8, the server-optimized SSVM is more performant, integrates better with JavaScript, provides access to external enterprise resources, and supports finely grained metering.

The demo application is structured as follows.

* The host application is a Node.js web application written in JavaScript. It makes WebAssembly function calls.
* The WebAssembly bytecode program is written in Rust. It runs inside the SSVM, and is called from the Node.js web application.

{% hint style="success" %}
The source code of the tutorial is [here](https://github.com/second-state/wasm-learning/tree/master/nodejs/hello).
{% endhint %}

{% hint style="info" %}
If you just want to try it out and do not wish to setup dev tools on your own machine, you can [fork this repository](https://github.com/second-state/ssvm-nodejs-starter) and have GitHub [build and test](https://github.com/second-state/ssvm-nodejs-starter/actions) your code for you! [See instructions here](the-no-software-approach.md).
{% endhint %}

#### **Setup**

The `ssvmup` npm module installs the Second State Virtual Machine \(SSVM\) into Node.js as a native `addon`, and provides the necessary compiler tools. Follow the steps below to install Rust and the `ssvmup` tool.

```text
# Prerequisite
$ apt-get update
$ apt install -y build-essential curl wget git vim libboost-all-dev

# Install rust
$ curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
$ source $HOME/.cargo/env

# Install nvm
$ curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.35.3/install.sh | bash
# Follow the on-screen instructions to logout and then log in

# Install node
$ nvm install v10.19.0
$ nvm use v10.19.0

# Install ssvmup toolchain
$ npm install -g ssvmup # Append --unsafe-perm if permission denied

# Install the nodejs addon for SSVM
$ npm install ssvm
```

#### **WebAssembly program in Rust**

In this example, our Rust program appends the input string after “hello”. Let’s create a new `cargo` project. Since this program is intended to be called from a host application, not to run as a stand-alone executable, we will create a `hello` project.

```text
$ cargo new --lib hello
$ cd hello
```

Edit the `Cargo.toml` file to add a `[lib]` section. It tells the compiler where to find the source code for the library and how to generate the bytecode output. We also need to add a dependency of `wasm-bindgen` here. It is the utility `ssvmup` uses to generate the JavaScript binding for the Rust WebAssembly program.

```text
[lib]
name = "hello_lib"
path = "src/lib.rs"
crate-type =["cdylib"]

[dependencies]
wasm-bindgen = "0.2.59"
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
$ ssvmup build
```

The result are files in the `pkg/` directory. the `.wasm` file is the WebAssembly bytecode program, and the `.js` files are for the JavaScript module.

#### **The Node.js host application**

Next, let’s create a `node` folder for the Node.js web application. Copy over the generated JavaScript module files.

```text
$ mkdir node
$ cp pkg/hello_lib_bg.wasm node/
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

Now we have seen a very simple example to call a Rust function from JavaScript in a Node.js application. In the next several tutorials, we will look into more complex examples of Rust JavaScript interaction using the SSVM. Let's start with [a review of all input output data types](../rust-javascript-data-exchange.md) supported in the SSVM Rust JavaScript bridge. After that, we will cover examples of cryptography, machine learning, data management, and artificial intelligence.

