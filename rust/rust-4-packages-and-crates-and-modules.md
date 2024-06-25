---
description: 书接上回
cover: ../.gitbook/assets/01.jpg
coverY: 0
---

# Rust-4 Packages & Crates & Modules

## Packages\&Crates\&Modules <a href="#managing-growing-projects-with-packages-crates-and-modules" id="managing-growing-projects-with-packages-crates-and-modules"></a>

~~我勒个豆，这里看官方文档真的看了半天看不懂，转战《Rust圣经》总算理解了一点~~

项目 `Package` 和包 `Crate` 和模块 `Module`

### **包 Crate**

对于 Rust 而言，包是一个独立的可编译单元，它编译后会**生成一个可执行文件或者一个库**。

一个包会将相关联的功能打包在一起，使得该功能可以很方便的在多个项目中分享。例如标准库中没有提供但是在三方库中提供的 `rand` 包，它提供了随机数生成的功能，我们只需要将该包通过 `use rand;` 引入到当前项目的作用域中，就可以在项目中使用 `rand` 的功能：`rand::XXX`。

### **项目 Package**

理解为工程、软件包，包含有独立的 `Cargo.toml` 文件，以及因为功能性被组织在一起的一个或多个包。一个 `Package` 只能包含**一个**库(library)类型的包，但是可以包含**多个**二进制可执行类型的包，

创建二进制package：

```bash
$ cargo new my-project
     Created binary (application) `my-project` package
$ ls my-project
Cargo.toml
src
$ ls my-project/src
main.rs
$ cargo run
```

创建库类型package：

```bash
$ cargo new my-lib --lib
     Created library `my-lib` package
$ ls my-lib
Cargo.toml
src
$ ls my-lib/src
lib.rs
#库类型的 Package 只能作为三方库被其它项目引用
#不能独立运行，只有二进制 Package 才可以运行$ cargo run
```

Package结构：

```
.
├── Cargo.toml
├── Cargo.lock
├── src
│   ├── main.rs
│   ├── lib.rs
│   └── bin
│       └── main1.rs
│       └── main2.rs
├── tests
│   └── some_integration_tests.rs
├── benches
│   └── simple_bench.rs
└── examples
    └── simple_example.rs

```

* 唯一库包：`src/lib.rs`
* 默认二进制包：`src/main.rs`，编译后生成的可执行文件与 `Package` 同名
* 其余二进制包：`src/bin/main1.rs` 和 `src/bin/main2.rs`，它们会分别生成一个文件同名的二进制可执行文件
* 集成测试文件：`tests` 目录下
* 基准性能测试 `benchmark` 文件：`benches` 目录下
* 项目示例：`examples` 目录下

### **模块 Module** <a href="#mo-kuai-module" id="mo-kuai-module"></a>

使用 `cargo new --lib restaurant` 创建一个小餐馆，注意，这里创建的是一个库类型的 `Package`，然后将以下代码放入 `src/lib.rs`

```rust
#![allow(unused)]
fn main() {
// 餐厅前厅，用于吃饭
mod front_of_house { //使用 mod 关键字来创建新模块，后面紧跟着模块名称
    mod hosting { //模块可以嵌套
        fn add_to_waitlist() {}
        //模块中可以定义各种 Rust 类型，例如函数、结构体、枚举、特征等

        fn seat_at_table() {}
    }

    mod serving {
        fn take_order() {}

        fn serve_order() {}

        fn take_payment() {}
    }
}
}
```

上面的代码产生了一个模块树：

```
crate
 └── front_of_house
     ├── hosting
     │   ├── add_to_waitlist
     │   └── seat_at_table
     └── serving
         ├── take_order
         ├── serve_order
         └── take_payment

```

想要调用一个函数，就需要知道它的路径，在 Rust 中，这种路径有两种形式：

* **绝对路径**，从包根开始，路径名以包名或者 `crate` 作为开头
* **相对路径**，从当前模块开始，以 `self`，`super` 或当前模块的标识符作为开头

```rust
mod front_of_house {
    mod hosting {
        fn add_to_waitlist() {}
    }
}

pub fn eat_at_restaurant() {
    // 绝对路径
    crate::front_of_house::hosting::add_to_waitlist();

    // 相对路径
    front_of_house::hosting::add_to_waitlist();
}
```

Rust 出于安全的考虑，默认情况下，**所有的类型都是私有化的**，包括函数、方法、结构体、枚举、常量，是的，就连模块本身也是私有化的。在 Rust 中，**父模块完全无法访问子模块中的私有项，但是子模块却可以访问父模块、父父..模块的私有项**，想要实现跨模块的访问需要`pub` 关键字控制模块和模块中指定项的可见性，注意**模块可见性不代表模块内部项的可见性**，模块的可见性仅仅是允许其它模块去引用它，但是**想要引用它内部的项，还需要继续将对应的项标记为 `pub`**

```rust
#![allow(unused)]
fn main() {
mod front_of_house {
    pub mod hosting { //模块可见性
        pub fn add_to_waitlist() {} //引用内部的项也需要设置可见性
    }
}

/*--- snip ----*/
}
```

#### use关键字

&#x20;Rust 中，可以使用 `use` 关键字把路径提前引入到当前作用域中，随后的调用就可以省略该路径，极大地简化了代码。同时可以使用as进行别名设置，下面是两段示例代码

```rust
#![allow(unused)]
fn main() {
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

use crate::front_of_house::hosting;

pub fn eat_at_restaurant() {
    hosting::add_to_waitlist();
    hosting::add_to_waitlist();
    hosting::add_to_waitlist();
}
}
//###########################################################################//
#![allow(unused)]
fn main() {
use std::fmt::Result;
use std::io::Result as IoResult;

fn function1() -> Result {
    // --snip--
}

fn function2() -> IoResult<()> {
    // --snip--
}
}
```

[**限制可见性语法**](https://course.rs/basic/crate-module/use.html#%E9%99%90%E5%88%B6%E5%8F%AF%E8%A7%81%E6%80%A7%E8%AF%AD%E6%B3%95)

[**一个综合例子**](https://course.rs/basic/crate-module/use.html#%E4%B8%80%E4%B8%AA%E7%BB%BC%E5%90%88%E4%BE%8B%E5%AD%90)

