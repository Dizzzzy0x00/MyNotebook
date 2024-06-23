---
description: 书接上回
cover: ../.gitbook/assets/01.jpg
coverY: 0
---

# Rust-4

## Packages\&Crates\&Modules <a href="#managing-growing-projects-with-packages-crates-and-modules" id="managing-growing-projects-with-packages-crates-and-modules"></a>

~~我勒个豆，这里看官方文档真的看了半天看不懂，转战《Rust圣经》总算理解了一点~~

项目 `Package` 和包 `Crate`

**包 Crate**

对于 Rust 而言，包是一个独立的可编译单元，它编译后会**生成一个可执行文件或者一个库**。

一个包会将相关联的功能打包在一起，使得该功能可以很方便的在多个项目中分享。例如标准库中没有提供但是在三方库中提供的 `rand` 包，它提供了随机数生成的功能，我们只需要将该包通过 `use rand;` 引入到当前项目的作用域中，就可以在项目中使用 `rand` 的功能：`rand::XXX`。
