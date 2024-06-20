---
description: 由于一些原因开始学习Rust，发现这个语言和c的区别并不小，于是打算开个笔记记录一下，哈哈就是一直在开坑的路上
cover: ../.gitbook/assets/微信图片_20240601094138.jpg
coverY: 0
layout:
  cover:
    visible: true
    size: full
  title:
    visible: true
  description:
    visible: true
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
---

# Rust-1

## Rust初识

{% embed url="https://www.rust-lang.org/learn" %}

{% embed url="https://www.rust-lang.org/zh-CN" %}

Rust是一种现代的、强类型的、系统级编程语言。它被设计用于编写高性能、并发、安全的系统，并且可以避免常见的内存安全问题，如空指针和数据竞争。Rust的特点包括：零成本抽象、无运行时开销、内存安全、并发安全、线程安全、高性能等。Rust 的类型系统更为严格，包括了所有权系统、借用检查、生命周期等概念，这使得 Rust 能够避免很多传统静态语言中的常见错误。总的来说Rust是一种少有的兼顾开发效率和执行效率的语言



## 基础语法



### Helloworld

学习一门语言还是从helloword开始：

```rust
fn main() {
    println!("Hello World!");
}
```

Rust 输出文字的方式主要有两种：`println!()` 和 `print!()`，这两个不是函数，而是宏规则，第一个参数是格式字符串，后面是一串可变参数，对应着格式字符串中的"占位符"，这一点与 C 语言中的 `printf` 函数很相似。但是，Rust 中格式字符串中的占位符不是 “% + 字母” 的形式，而是一对 `{}`，所以在输出{或}是需要进行转义：

```rust
println!("a is {0}, a again is {0}", a);
println!("{{}}"); 
```

### 一个Rust Program示例

```
$ cargo new guessing_game
$ cd guessing_game
```

Cargo.toml：

```
[package]
name = "guessing_game"
version = "0.1.0"
edition = "2021"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[dependencies]

```

src/main.rs：

```rust
use std::io; //io标准库

fn main() {
    println!("Guess the number!");

    println!("Please input your guess.");

    let mut guess = String::new();

    io::stdin()
        .read_line(&mut guess)
        .expect("Failed to read line");

    println!("You guessed: {}", guess);
}
```

Rust标准库_prelude_可以参见这个文档：

{% embed url="https://doc.rust-lang.org/std/prelude/index.html" %}

编译：

```
$ cargo build
$ cargo run
```

### 变量和可变性

Rust 是强类型语言，但具有自动判断变量类型的能力，声明变量，需要使用 `let` 关键字：

```
let a = 123;
```

但这不意味着 a 不是"变量"（英文中的 variable），官方文档称 a 这种变量为**不可变变量，**Rust 语言为了高并发安全而做的设计：在语言层面尽量少的让变量的值可以改变。所以 a 的值不可变。所以在上面的声明以后，下面三个语句都是不合法的：

```rust
a = "abc";
a = 4.56; 
a = 456;
```

如果要让变量"可变"（mutable），需要 `mut` 关键字

```rust
let mut a = 123;
//此时下面的赋值是合法：
a = 456;

let a: u64 = 123;//声明 a 为无符号 64 位整型变量，
//如果没有声明类型，a 将自动被判断为有符号 32 位整型变量
```

#### **重影（Shadowing）**

变量的值可以"重新绑定"，但在"重新绑定"以前不能私自被改变，之前提到一个“不可变变量”，获取有一些迷惑，既然不可变，那不就是常量吗？但是这两个是不同的，因为不可变变量是可以重新绑定的：

```rust
// 合法
let a = 123;   // 可以编译，但可能有警告，因为该变量没有被使用
let a = 456;

//不合法
const a: i32 = 123;
let a = 456;

```

### 数据类型

#### [**Integer Types**](https://doc.rust-lang.org/book/ch03-02-data-types.html#integer-types) **整型**

| Length  | Signed  | Unsigned |
| ------- | ------- | -------- |
| 8-bit   | `i8`    | `u8`     |
| 16-bit  | `i16`   | `u16`    |
| 32-bit  | `i32`   | `u32`    |
| 64-bit  | `i64`   | `u64`    |
| 128-bit | `i128`  | `u128`   |
| arch    | `isize` | `usize`  |

| Number literals  | Example       |
| ---------------- | ------------- |
| Decimal          | `98_222`      |
| Hex              | `0xff`        |
| Octal            | `0o77`        |
| Binary           | `0b1111_0000` |
| Byte (`u8` only) | `b'A'`        |

{% hint style="info" %}
#### Rust中的整型溢出问题

假设一个变量`u8`，其值可以取在 0 到 255 之间。如果尝试将变量更改为该范围之外的值（例如 256），则会发生_整数溢出_，这可能导致以下两种行为之一。在调试模式下编译时，Rust 会检查整数溢出，如果发生此行为，则会导致程序在运行时_崩溃_。Rust 使用术语“_panicking”来表示程序因错误退出_

在release mode下进行编译时`--release`，Rust 不会检查会导致`panic`的整数溢出。相反，如果发生溢出，Rust 会执行_two’s complement wrapping_。简而言之，大于类型可以容纳的最大值的值会“包装”到类型可以容纳的最小值。对于`u8`，值 256 变为 0，值 257 变为 1，依此类推。程序不会panic，但变量的值可能不是期望的值。依赖整数溢出的包装行为被视为错误。

为了明确处理溢出的可能性，可以使用标准库为原始数字类型提供的这些方法系列：

* 使用方法包装所有模式`wrapping_*`，例如`wrapping_add`。
* `None`如果方法溢出，则返回值`checked_*`。
* 返回该值和一个布尔值，表示方法是否存在溢出`overflowing_*`。
* 使用方法在值的最小值或最大值处饱和`saturating_*` 。
{% endhint %}



[**Floating-Point Types**](https://doc.rust-lang.org/book/ch03-02-data-types.html#floating-point-types) **浮点数**

遵循IEEE-754

```rust
fn main() {
    let x = 2.0; // f64 单精度浮点数

    let y: f32 = 3.0; // f32 双精度浮点数
}
```

#### 算数运算

基本的加减乘除

```rust
fn main() {
    // addition
    let sum = 5 + 10;

    // subtraction
    let difference = 95.5 - 4.3;

    // multiplication
    let product = 4 * 30;

    // division
    let quotient = 56.7 / 32.2;
    let truncated = -5 / 3; // Results in -1

    // remainder
    let remainder = 43 % 5;
}
```

[**The Boolean Type**](https://doc.rust-lang.org/book/ch03-02-data-types.html#the-boolean-type) **布尔**

two possible values: `true` and `false`

[**The Character Type**](https://doc.rust-lang.org/book/ch03-02-data-types.html#the-character-type) **字符类型**

#### [Compound Types](https://doc.rust-lang.org/book/ch03-02-data-types.html#compound-types) 复合类型 <a href="#compound-types" id="compound-types"></a>

**Tuple 元组**

很好理解，直接看示例用法就行

```rust
fn main() {
    let tup0: (i32, f64, u8) = (500, 6.4, 1);
    let tup = (500, 6.4, 1);
    let (x, y, z) = tup;
    println!("The value of y is: {y}");
    let five_hundred = tup.0;
    let six_point_four = tup.1;
    let one = tup.2;
}
```

**Array 数组**

```rust
let a = [1, 2, 3, 4, 5];
let months = ["January", "February", "March", "April", "May", "June", "July",
              "August", "September", "October", "November", "December"];
let a: [i32; 5] = [1, 2, 3, 4, 5];
let b = [3; 5];//相当于let a = [3, 3, 3, 3, 3];
let first = a[0];
let second = a[1];
```



### 控制流

#### if分支

基本的用法一样的，给个示例过了

```rust
fn main() {
    let a=10;
    if a>0 {
        println!("a>0");
    }else if  a==0{
        println!("a==0");
    }else{
        println!("a<0");
    }
    
    let condition = true;
    let number = if condition { 5 } else { 6 };
}
```

#### 循环

`rust`中的循环语句共有三个：`for`、`while`以及`loop`

```rust
fn main() {
    let mut a=10;
    loop {
        println!("{}",a);
        a=a+1;
        if a==20{
            break;
        }
    }
    a = 10;
    while a!=20 {
        println!("{}",a);
        a=a+1;
    }
    a = 10;
    for i in 10..20{
    //范围运算符 ..生成的范围对象是左闭右开的，
    //10..20 ，i只会等于10到19
        println!("{}",i);
    }
    //可以遍历数组，但不能遍历元组：
    let arr=[10,20,30,40];
    let t=(10,'c',false);
    //正确，可以遍历数组，因为每个数组的元素类型都相同
    for i in arr{
        println!("{}",i);
    }
    //错误，由于元组元素类型可能相同，所以不能这样遍历
    // for i in t{
    //     println!("{}",i);
    // }
}


```

**循环标签**

如果存在嵌套循环，**`break` 和 `continue` 应用于此时最内层的循环**。选择在一个循环上指定一个循环标签（loop label），然后将标签与 `break` 或 `continue` 一起使用，使这些关键字应用于已标记的循环而不是最内层的循环。

```rust
fn main() {
    let mut count = 0;
    'counting_up: loop {
        println!("count = {count}");
        let mut remaining = 10;

        loop {
            println!("remaining = {remaining}");
            if remaining == 9 {
                break;
            }
            if count == 2 {
                break 'counting_up;
            }
            remaining -= 1;
        }

        count += 1;
    }
    println!("End count = {count}");
}

//外层循环有一个标签 counting_up，它将从 0 数到 2。没有标签的内部循环从 10 向下数到 9。
//第一个没有指定标签的 break 将只退出内层循环。break 'counting_up; 语句将退出外层循环。
//运行结果如下：
count = 0
remaining = 10
remaining = 9
count = 1
remaining = 10
remaining = 9
count = 2
remaining = 10
End count = 2

```
