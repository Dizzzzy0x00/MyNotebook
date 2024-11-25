---
description: 好久不见 书接上回
cover: ../.gitbook/assets/01.jpg
coverY: 0
---

# Rust-7 Closure

_**闭包（closures）**_&#x662F;一种匿名函数，它可以赋值给变量也可以作为参数传递给其它函数，不同于函数的是，它允许捕获调用者作用域中的值，并在需要时调用。闭包提供了一种方便的方式来封装行为。

### 定义和语法

闭包使用 `||` 符号来定义，类似于匿名函数

```rust
fn main() {
    let add = |a, b| a + b; //定义了一个名为 add 的闭包
    //接受两个参数 a 和 b，并返回它们的和
    let result = add(2, 3);
    println!("The result is: {}", result);
}
```

### 闭包捕获变量

闭包可以捕获其环境中的变量，并在闭包的主体中使用。有三种方式可以捕获变量：

* `Fn` 闭包：通过引用捕获变量，不可变借用。
* `FnMut` 闭包：通过可变引用捕获变量，可变借用。
* `FnOnce` 闭包：通过值捕获变量，所有权转移。

下面示例定义三个闭包 `add`、`multiply` 和 `divide`，分别通过**引用、可变引用和值**来捕获变量 `x` 和 `y`。通过不同的捕获方式，闭包对变量的访问权限也不同。

```rust
fn main() {
    let x = 5;
    let y = 10;

    // Fn 闭包：通过引用捕获变量
    let add = |a| a + x;

    // FnMut 闭包：通过可变引用捕获变量
    let mut multiply = |a| {
        x * y * a
    };

    // FnOnce 闭包：通过值捕获变量
    let divide = move |a| {
        a / y
    };

    let result1 = add(3);
    let result2 = multiply(2);
    let result3 = divide(10);

    println!("The results are: {}, {}, {}", result1, result2, result3);
}

```

### 闭包作为参数和返回值

&#x20;下面的示例中定义`apply` 函数，它接受一个闭包作为参数，并在函数内部调用该闭包。定义`create_closure` 函数，返回一个闭包

```rust
fn apply<F>(f: F)
where
    F: FnOnce(),
{
    f();
}

fn create_closure() -> impl Fn() {
    let x = 5;
    move || println!("The value of x is: {}", x)
}

fn main() {
    let print_hello = || println!("Hello, world!");

    apply(print_hello);

    let closure = create_closure();
    closure();
}

```



