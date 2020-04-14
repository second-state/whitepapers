---
description: WebAssembly could improve security and efficiency of web services
---

# The case for server-side WebAssembly

\*\*\*\*[**This article is outdated. Please see the latest version here.**](https://cloud.secondstate.io/server-side-webassembly/why)\*\*\*\*

{% embed url="https://www.youtube.com/watch?v=RjYLHxNO4nM" caption="" %}

You can bootstrap a WebAssembly virtual machine from your server side application, such as a Node.js app, and then execute high performance and potentially unsafe code inside the virtual machine. The major use cases are web applications that require native performance or must execute user-submitted code.

{% hint style="info" %}
Node.js and Rust developers [get started here](../webassembly-on-the-server-side/)! Create high performance Rust + JavaScript hybrid apps on Node.js.
{% endhint %}

**Native code** is often used where high performance and efficiency are required. The popular Node.js runtime is written in native C/C++. Native code runs AI inference, big data analytics, image and video processing, and scientific computing. WebAssembly is a light, fast, and cross-platform container. It is a safe and managed alternative to native code. [Learn more here](https://medium.com/wasm/webassembly-on-the-server-side-c584f874b4a3).

**User submitted code** is the cornerstone of cloud native or serverless computing. For SaaS providers, users want to customize their experiences using code, or to create applications for their peers. Examples include Function as a Service, workflow apps, smart contracts, plugins, and extensions. WebAssembly supports multiple programming languages, and can provide a high performance sandbox for user submitted code with little resource consumption.

[Second State](https://www.secondstate.io/) provides an open source WebAssembly implementation \(Second State Virtual Machine, or [SSVM](https://github.com/second-state/SSVM)\) that is specifically optimized for server side applications. It is

* High performance with support for JIT and AOP optimizations.
* Seamlessly supports server application frameworks, such as NodeJS. You can build high performance NodeJS apps with SSVM.
* Supports safe access to external resources, such as databases, message queues, and even new AI hardware
* Allows precise metering of computational resources for serverless apps.

Visit our [web site](https://www.secondstate.io/) or follow us on social media \[[Twitter](https://twitter.com/secondstateinc), [LinkedIn](https://www.linkedin.com/company/second-state/), and [Medium](https://medium.com/wasm)\], and learn how WebAssembly could improve your web applications and services!

