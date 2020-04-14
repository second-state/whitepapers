---
description: >-
  How can WebAssembly and Rust programs benefit your web and services
  application
---

# WebAssembly vs native code

\*\*\*\*[**This article is outdated. Please see the latest version here.**](https://cloud.secondstate.io/server-side-webassembly/why/webassembly-vs-native-code)\*\*\*\*

WebAssembly aims to drastically improve your application performance, safety, and developer productivity. It replaces native code with a managed container and finely grained security model. 

{% embed url="https://youtu.be/dxnTNe6Nmpw" %}



#### **Why do we program in native code in 2020?**

In the past several years, CPU speed has pretty much stopped improving. At the same time, AI, big data, and blockchain have all create huge demands for more computing power. So far, the solution has been more and more native code in our software. Native code is efficient, close to the hardware, and can access specialized hardware such as GPU and AI chips.

However, native code also has issues such as platform dependency and safety. The big trend in software engineering in the past 30 years has been to move away from native code into managed code running inside virtual machines or containers.

#### **How exactly is WebAssembly better than native code?**

WebAssembly is the next-generation virtual machine that will help us turn native code modules into managed services.

* WebAssembly programs can be **written in multiple programming languages**, not just C and C++. In particular, Rust is well supported on WebAssembly.
* WebAssembly programs can be **accessed or invoked from multiple programming frameworks**, such as JavaScript, Python, and PHP.
* WebAssembly programs are **cross-platform**. They can run without change on all major operating systems and hardware platforms.
* WebAssembly programs are **safe** as they are executed inside a virtual machine.
* WebAssembly programs are **highly efficient** and very fast due to its lightweight virtual machine design. It is on par with native code performance.
* WebAssembly provides an easy and secure **extension mechanism to access new hardware**.

#### **WebAssembly is safe, very fast, language-agnostic, and platform-independent. But, isn’t WebAssembly mostly used inside the web browser?**

WebAssembly started as a collaboration between Google, Mozilla, Apple, and Microsoft. It was envisioned to be a high-performance code execution engine inside browsers. The typical applications would be in-browser animated games that require performance, much like the Java Applet from the old days.

{% embed url="https://youtu.be/R7WB3gmtku8" %}



However, like Java and JavaScript before it, WebAssembly is finding success on the server-side. WebAssembly’s safety, performance, platform, and language independence, make it an ideal server-side runtime.

#### **Is it true that one must learn Rust in order to use WebAssembly?**

No. WebAssembly is language agnostic. You can invoke WebAssembly programs and functions from a variety of different host languages, such as Javascript, Rust, Go, Python or even PHP.

You can also write WebAssembly programs in a variety of different programming languages. However, it is also true that Rust is currently the most widely used language to create WebAssembly programs and modules.

Rust has been voted the most beloved programming language for the past 4 years in a row. It is the hottest programming language right now. It has many exciting features. For example, it is powerful and flexible like C, but much safer and without Java’s performance overhead. It supports both object-oriented and functional programming paradigms. It is one of the fastest-growing programming languages in the world and is now used in the entire software stack from front end to backend to infrastructure.

Our first tutorial is to get you set up with Rust tools to write WebAssembly programs!

{% page-ref page="../../related-technologies/a-rusty-hello-world.md" %}

