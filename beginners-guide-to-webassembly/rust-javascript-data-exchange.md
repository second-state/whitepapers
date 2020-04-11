---
description: >-
  Seamless integration between the high performance Rust and easy-to-use
  JavaScript
---

# Rust to JavaScript

The role of WebAssembly is not \(yet\) to replace JavaScript, but to enhance JavaScript by taking over computationally intensive tasks. Similarly, it is not \(yet\) to replace native applications or libraries, but to provide a safe sandbox for native and user submitted code. In server side Node.js applications, that translates to functions written in Rust that can be called from a JavaScript host program.

{% hint style="info" %}
Before you start, make sure that you have gone through the [previous tutorial to set up a Node.js application in Rust](webassembly-on-the-server-side/). You should have [`ssvmup`](https://www.npmjs.com/package/ssvmup) and [`ssvm`](https://www.npmjs.com/package/ssvm) NPM modules installed through the last tutorial.
{% endhint %}

In this tutorial, we will show more examples on how the Rust program in WebAssembly / SSVM can exchange data with the Node.js JavaScript host app. Currently, the SSVM's JavaScript to Rust bridge has the following constraints. 

* Rust call parameters can be any combo of `i32`, `String`, `&str`, `Vec<u8>`, and `&[u8]`
* Return value can be `i32` or `String` or `Vec<u8>`
* For complex data types, such as structs, you could use JSON strings to pass data

As you will see in the examples, we believe that SSVM can support the vast majority of Rust applications and libraries, and make them available in Node.js.

{% hint style="success" %}
The source code of the tutorial is [here](https://github.com/second-state/wasm-learning/tree/master/nodejs/functions). 
{% endhint %}

#### **WebAssembly program in Rust**

In the `cargo` project called `functions`, edit the `Cargo.toml` file to add a `[lib]` section and a `[dependencies]` section. Besides the `wasm-bindgen` dependency, notice the `serde` and `serde_json` dependencies. They allow us to serialize and deserialize complex Rust types to and from JSON strings, so that the data can be passed to and from JavaScript.

```text
[lib]
name = "functions_lib"
path = "src/lib.rs"
crate-type =["cdylib"]

[dependencies]
num-integer = "0.1"
sha3 = "0.8.2"
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0"
wasm-bindgen = "0.2.59"
```

Below is the content of the Rust program `src/lib.rs`. Perhaps the most interesting is the `create_line()` function. It takes two JSON strings, each representing a `Point` struct, and returns a JSON string representing a `Line` struct. Notice that both the `Point` and `Line` structs are annotated with `Serialize` and `Deserialize` so that the Rust compiler automatically generates necessary code to support their conversion to and from JSON strings.

```text
use wasm_bindgen::prelude::*;
use num_integer::lcm;
use sha3::{Digest, Sha3_256, Keccak256};
use serde::{Serialize, Deserialize};

#[derive(Serialize, Deserialize, Debug)]
struct Point {
  x: f32, 
  y: f32
}

#[derive(Serialize, Deserialize, Debug)]
struct Line {
  points: Vec<Point>,
  valid: bool,
  length: f32,
  desc: String
}

#[wasm_bindgen]
pub fn create_line (p1: &str, p2: &str, desc: &str) -> String {
  let point1: Point = serde_json::from_str(p1).unwrap();
  let point2: Point = serde_json::from_str(p2).unwrap();
  let length = ((point1.x - point2.x) * (point1.x - point2.x) + (point1.y - point2.y) * (point1.y - point2.y)).sqrt();
  
  let valid = if length == 0.0 { false } else { true };
  let line = Line { points: vec![point1, point2], valid: valid, length: length, desc: desc.to_string() };
  return serde_json::to_string(&line).unwrap();
}

#[wasm_bindgen]
pub fn say(s: &str) -> String {
  let r = String::from("hello ");
  return r + s;
}

#[wasm_bindgen]
pub fn obfusticate(s: String) -> String {
  (&s).chars().map(|c| {
    match c {
      'A' ..= 'M' | 'a' ..= 'm' => ((c as u8) + 13) as char,
      'N' ..= 'Z' | 'n' ..= 'z' => ((c as u8) - 13) as char,
      _ => c
    }
  }).collect()
}

#[wasm_bindgen]
pub fn lowest_common_denominator(a: i32, b: i32) -> i32 {
  let r = lcm(a, b);
  return r;
}

#[wasm_bindgen]
pub fn sha3_digest(v: Vec<u8>) -> Vec<u8> {
  return Sha3_256::digest(&v).as_slice().to_vec();
}

#[wasm_bindgen]
pub fn keccak_digest(s: &[u8]) -> Vec<u8> {
  return Keccak256::digest(s).as_slice().to_vec();
}
```

Next, you can compile the Rust source code into WebAssembly bytecode and generate the accompanying JavaScript module for the Node.js host environment.

```text
$ ssvmup build
```

The result are files in the `pkg/` directory. the `.wasm` file is the WebAssembly bytecode program, and the `.js` files are for the JavaScript module.

#### **The Node.js host application**

Next, let’s create a node folder for the Node.js web application. Copy over the generated JavaScript module files.

```text
$ mkdir node
$ cp pkg/* node/
```

Below is the node application `app.js`. It shows how to call the Rust functions. As you can see `String` and `&str` are simply strings in JavaScript, `i32` are numbers, and `Vec<u8>` or `&[8]` are JavaScript `Uint8Array`. JavaScript objects need to go through `JSON.stringify()` or `JSON.parse()` before being passed into or returned from Rust functions.

```text
const { say, obfusticate, lowest_common_denominator, sha3_digest, keccak_digest, create_line } = require('./functions_lib.js');

var util = require('util');
const encoder = new util.TextEncoder();
console.hex = (d) => console.log((Object(d).buffer instanceof ArrayBuffer ? new Uint8Array(d.buffer) : typeof d === 'string' ? (new util.TextEncoder('utf-8')).encode(d) : new Uint8ClampedArray(d)).reduce((p, c, i, a) => p + (i % 16 === 0 ? i.toString(16).padStart(6, 0) + '  ' : ' ') + c.toString(16).padStart(2, 0) + (i === a.length - 1 || i % 16 === 15 ?  ' '.repeat((15 - i % 16) * 3) + Array.from(a).splice(i - i % 16, 16).reduce((r, v) => r + (v > 31 && v < 127 || v > 159 ? String.fromCharCode(v) : '.'), '  ') + '\n' : ''), ''));

console.log( say("SSVM") );
console.log( obfusticate("A quick brown fox jumps over the lazy dog") );
console.log( lowest_common_denominator(123, 2) );
console.hex( sha3_digest(encoder.encode("This is an important message")) );
console.hex( keccak_digest(encoder.encode("This is an important message")) );

var p1 = {x:1.5, y:3.8};
var p2 = {x:2.5, y:5.8};
var line = JSON.parse(create_line(JSON.stringify(p1), JSON.stringify(p2), "A thin red line"));
console.log( line );
```

Run `app.js` in Node.js environment as follows.

```text
$ node app.js
hello SSVM
N dhvpx oebja sbk whzcf bire gur ynml qbt
246
000000  57 1b e7 d1 bd 69 fb 31 9f 0a d3 fa 0f 9f 9a b5  W.çÑ½iû1..Óú...µ
000010  2b da 1a 8d 38 c7 19 2d 3c 0a 14 a3 36 d3 c3 cb  +Ú..8Ç.-<..£6ÓÃË

000000  7e c2 f1 c8 97 74 e3 21 d8 63 9f 16 6b 03 b1 a9  ~ÂñÈ.tã!Øc..k.±©
000010  d8 bf 72 9c ae c1 20 9f f6 e4 f5 85 34 4b 37 1b  Ø¿r.®Á .öäõ.4K7.

{ points: [ { x: 1.5, y: 3.8 }, { x: 2.5, y: 5.8 } ],
  valid: true,
  length: 2.2360682,
  desc: 'A thin red line' }
```

#### **What’s next?**

In the next several tutorials, we will show more complex use cases. The JavaScript host app will delegate computationally intensive tasks such as cryptography, machine learning, and artificial intelligence to Rust modules inside the SSVM WebAssembly runtime.





