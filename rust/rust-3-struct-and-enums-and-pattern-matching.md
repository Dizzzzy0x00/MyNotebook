---
description: 书接上回，本篇有关Slice、结构体、枚举和匹配控制流
cover: ../.gitbook/assets/01.jpg
coverY: 0
---

# Rust-3 Struct & Enums & Pattern Matching

### [Slice ](https://doc.rust-lang.org/book/ch04-03-slices.html#the-slice-type) 类型 <a href="#the-slice-type" id="the-slice-type"></a>

slice 允许引用某个集合中一段连续的元素序列，而不用引用整个集合。slice 是一类引用，所以它没有所有权。

#### **字符串 slice**

```rust
    let s = String::from("hello world");

    let hello = &s[0..5];
    let world = &s[6..11];
```

不同于整个 String 的引用，hello 是一个部分 String 的引用，由一个额外的 \[0..5] 部分指定。可以使用一个由中括号中的 \[starting\_index..ending\_index] 指定的 range 创建一个 slice，其中 starting\_index 是 slice 的第一个位置，ending\_index 则是 slice 最后一个位置的后一个值。在其内部，slice 的数据结构存储了 slice 的开始位置和长度，长度对应于 ending\_index 减去 starting\_index 的值

<figure><img src="../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

```rust
//slice一些关于开头和结尾的省略：
let slice = &s[0..2];
let slice = &s[..2];


let len = s.len();
let slice = &s[3..len];
let slice = &s[3..];

let slice = &s[0..len];
let slice = &s[..];
```

[**String Literals as Slices**](https://doc.rust-lang.org/book/ch04-03-slices.html#string-literals-as-slices) **字符串字面值就是Slices**

之前提到的字符串字面值：`let s = "Hello, world!";`其实就是一个Slices，这样就只是对字符串的引用，也就可以理解为什么是不可边的变量

#### Other Slices <a href="#other-slices" id="other-slices"></a>

除了字符串，也可以对其他类型进行slice，例如数组：

```rust
fn main() {
    let a = [1, 2, 3, 4, 5];
    
    let slice = &a[1..3];
    
    assert_eq!(slice, &[2, 3]);
}

```

## [结构体](https://doc.rust-lang.org/book/ch05-01-defining-structs.html) <a href="#articlecontentid" id="articlecontentid"></a>

<figure><img src="../.gitbook/assets/de432db1bfe04d05948d94a402804bdb.png" alt="" width="188"><figcaption><p>Rust可爱捏</p></figcaption></figure>

### 声明结构体

使用关键字struct

```rust
struct User {
    active: bool,
    username: String,
    email: String,
    sign_in_count: u64,
}

fn main() {
    //使用结构体：
    let mut user1 = User {
        active: true,
        username: String::from("someusername123"),
        email: String::from("someone@example.com"),
        sign_in_count: 1,
    };
    //注意整个实例必须是可变的；
    //Rust 并不允许只将某个字段标记为可变。
    //另外可以在函数体的最后一个表达式中构造一个结构体的新实例隐式地返回这个实例
    user1.email = String::from("anotheremail@example.com");
}
```

结构体初始化函数，如果函数参数名与字段名都完全相同，可以使用**字段初始化简写语法**（field init shorthand）：

```rust
fn build_user(email: String, username: String) -> User {
    User {
        active: true,
        username,
        email,
        sign_in_count: 1,
    }
}
```

### 结构体更新语法

```rust
fn main() {
    let user2 = User {
        email: String::from("another@163.com"),
        ..user1
        //..user1 必须放在最后
        //指定其余的字段应从 user1 的相应字段中获取其值
    };
    
    //这段代码等价于:
    let user2 = User {
        active: user1.active,
        username: user1.username,
        email: String::from("another@163.com"),
        sign_in_count: user1.sign_in_count,
    };

}
```

### 类单元结构体

定义一个没有任何字段的结构体——**类单元结构体**（unit-like structs）类似于 `()`，即“元组类型”中提到的 unit 类型，类单元结构体常常在你想要在某个类型上实现 trait 但不需要在类型中存储数据的时候发挥作用。

```rust
struct AlwaysEqual;

fn main() {
    let subject = AlwaysEqual;
}

```

### 使用派生特征添加功能

想要打印一个结构体时，不能直接调用println!宏，对于结构体， `println!`格式化输出的方式不太明确，要逗号吗？想打印花括号吗？是否应该显示所有字段？由于这种歧义，Rust 不会试图猜测我们想要什么，并且结构体没有提供的实现来与占位符`Display`一起使用

```rust
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let rect1 = Rectangle {
        width: 30,
        height: 50,
    };

    println!("rect1 is {}", rect1);
}
//报错：
error[E0277]: `Rectangle` doesn't implement `std::fmt::Display`
   = help: the trait `std::fmt::Display` is not implemented for `Rectangle`
   = note: in format strings you may be able to use `{:?}` (or {:#?} for pretty-print) instead

```

一种修改是使用派生特征添加功能，在结构体定义时添加outer属性`#[derive(Debug)]`

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let rect1 = Rectangle {
        width: 30,
        height: 50,
    };

    println!("rect1 is {rect1:?}");
}
```



### 方法

方法（method）与函数类似：它们使用 fn 关键字和名称声明，可以拥有参数和返回值，同时包含在某处调用该方法时会执行的代码。不过方法与函数是不同的，因为它们在结构体的上下文中被定义（或者是枚举或 trait 对象的上下文），并且它们第一个参数总是 self，它代表调用该方法的结构体实例，示例如下：

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }
}

fn main() {
    let rect1 = Rectangle {
        width: 30,
        height: 50,
    };

    println!(
        "The area of the rectangle is {} square pixels.",
        rect1.area()
    );
}
```

上面的示例代码使用关键字`impl` 启动一个实现 (implementation) 块`Rectangle`。此`impl`块中的所有内容都将与`Rectangle`类型相关联，在`impl`块中的`area`方法使用`&self`而不是`rectangle: &Rectangle`。`&self`实际上是 的缩写`self: &Self`。在块中`impl`， 类型`Self`是块所针对类型的别名`impl`。方法的第一个参数必须有一个名为`self`类型的参数`Self`

如果想在方法执行的过程中更改调用该方法的实例，使用`&mut self`作为第一个参数。这种第一个参数来取得实例所有权的方法很少见；这种方法转换`self`为其他内容并且在调用者在转换后禁止使用原始实例，对self的三种不同传参方式：

读取reading (`&self`)

转移mutating (`&mut self`)

消费consuming (`self`).

#### 关联函数

所有在 impl 块中定义的函数被称为关联函数（associated functions），因为它们与 impl 后面命名的类型相关。可以定义不以 self 为第一参数的关联函数（因此不是方法）之前使用的函数String::from 函数就是String的关联函数，使用结构体名和 `::` 语法来调用关联函数，函数位于结构体的命名空间中，`::` 语法用于关联函数和模块创建的命名空间。

关联函数通常用于返回结构体新实例的构造函数：

```rust
impl Rectangle {
    fn square(size: u32) -> Self {
        Self {
            width: size,
            height: size,
        }
    }
}
fn main(){
    let sq = Rectangle::square(3);
}
```

## [Enums and Pattern Matching](https://doc.rust-lang.org/book/ch06-00-enums.html#enums-and-pattern-matching) 枚举和模式匹配  <a href="#enums-and-pattern-matching" id="enums-and-pattern-matching"></a>

枚举通过**枚举可能的变体**来定义类型，结构体提供了一种将相关字段和数据组合在一起的方法，而枚举提供了一种表示值是一组可能值之一的方法。

```rust
enum IpAddrKind {
    V4,
    V6,
}
fn main(){
    let ipV4 = IpAddrKind::V4;
    let ipV6 = IpAddrKind::V6;
    //枚举的变体在其标识符下命名空间，使用双冒号将两者分开。
    //现在ipV4和ipV6都是同一类型：IpAddrKind
}
```

可以将**字符串、数字类型或结构，甚至可以另一个枚举**直接放入每个枚举变量中：

```rust
    enum MyIpAddr {
        V4(u8, u8, u8, u8),
        V6(String),
    }

    let home = MyIpAddr::V4(127, 0, 0, 1);

    let loopback = MyIpAddr::V6(String::from("::1"));
    
    
    //标准库定义的IpAddr
    struct Ipv4Addr {
        // --snip--
    }
    
    struct Ipv6Addr {
        // --snip--
    }
    
    enum IpAddr {
        V4(Ipv4Addr),
        V6(Ipv6Addr),
    }

```

结构体和枚举还有另一个相似点：就像可以使用 impl 来为结构体定义方法那样，也可以在枚举上定义方法。这是一个定义于我们 Message 枚举上的叫做 call 的方法：

```rust
fn main() {
    enum Message {
        Quit,
        Move { x: i32, y: i32 },
        Write(String),
        ChangeColor(i32, i32, i32),
    }

    impl Message {
        fn call(&self) {
            // method body would be defined here
        }
    }

    let m = Message::Write(String::from("hello"));
    m.call();
}
```



### Option Enum

`Option`是由Rust标准库定义的一个枚举变量，该`Option`类型编码了一种非常常见的情况，即值可以是某物，也可以是空

编程语言设计通常考虑要包含哪些功能，但要排除哪些功能也很重要。**Rust 没有许多其他语言所具有的 null 功能**。Null是一个表示没有值的值。在具有 null 的语言中，变量始终处于两种状态之一：null 或 not-null

Null 的发明者 Tony Hoare 在 2009 年的演讲“Null 引用：十亿美元的错误”中说道：

> I call it my billion-dollar mistake. At that time, I was designing the first comprehensive type system for references in an object-oriented language. My goal was to ensure that all use of references should be absolutely safe, with checking performed automatically by the compiler. But I couldn’t resist the temptation to put in a null reference, simply because it was so easy to implement. This has led to innumerable errors, vulnerabilities, and system crashes, which have probably caused a billion dollars of pain and damage in the last forty years.\
>

```rust
enum Option<T> {
    None,
    Some(T),
}
```

`Option`已经包含在了 prelude 之中，也就是不需要将其显式引入作用域。另外，它的成员也是如此，可以不需要 Option:: 前缀来直接使用 Some 和 None。即便如此 Option 也仍是常规的枚举，Some(T) 和 None 仍是 Option 的成员。

\<T>语法表示一个泛型类型参数

看下面的示例代码：

```rust
fn main() {
    let x: i8 = 5;
    let y: Option<i8> = Some(5);

    let sum = x + y;
    //Rust认为尝试将两个不同的变量进行相加，所以会报错
}
//无法编译：
$ cargo run
   Compiling enums v0.1.0 (file:///projects/enums)
error[E0277]: cannot add `Option<i8>` to `i8`
 --> src/main.rs:5:17
  |
5 |     let sum = x + y;
  |                 ^ no implementation for `i8 + Option<i8>`
  |
  = help: the trait `Add<Option<i8>>` is not implemented for `i8`
  = help: the following other types implement trait `Add<Rhs>`:
            <i8 as Add>
            <i8 as Add<&i8>>
            <&'a i8 as Add<i8>>
            <&i8 as Add<&i8>>

For more information about this error, try `rustc --explain E0277`.
error: could not compile `enums` (bin "enums") due to 1 previous error


```

### match 控制流 <a href="#the-match-control-flow-construct" id="the-match-control-flow-construct"></a>

Rust 有一个非常强大的控制流构造，称为`match`，允许**将一个值与一系列模式**进行比较，**然后根据匹配的模式执行代码**。模式可以由**文字值、变量名、通配符和许多其他内容**组成；[这里](https://doc.rust-lang.org/book/ch18-00-patterns.html)介绍了所有不同类型的模式及其作用。`match`的强大之处在于模式的表达能力以及编译器确认**所有可能的情况都得到处理**的事实。可以将`match`想象成一台硬币分拣机：

```rust
enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter,
}

fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => {
            println!("Lucky penny!");
            1
        }
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter => 25,
    }
}

fn main() {}
```

**注意match的穷尽性（exhaustive）**，例如在示例中必须match完Coin的四种类型——Penny,Nickel,Dime,Quarter，否则无法通过编译，如果对其他的值采取默认操作可以使用通配符方式：

```rust
enum Vehicle {
    Airplane,
    Train,
    Car,
    Ship,
}

fn fare(vehicle: Vehicle) -> i32 {
    match vehicle {
        Vehicle::Airplane => 800,
        Vehicle::Train => 100,
        //第一个没有显式地处理 Vehicle 枚举中的其余情况的被认为通配分支
        other => {
            -1
            //如果这样写Rust警告存在未使用的变量
        }
        //必须将通配分支放在最后，因为match是按顺序匹配的
        //如果我们在通配分支后添加其他分支
        //Rust 将会警告此后的分支永远不会被匹配到
    }
}

fn fare2(vehicle: Vehicle) -> i32 {
    match vehicle {
        Vehicle::Airplane => 800,
        Vehicle::Train => 100,
        _ => {
            //使用_通配符可以匹配任意值而不绑定到该值
            //不会警告存在未使用的变量
            -1
        }
        
    }
}
```

### [matches!宏](https://course.rs/basic/match-pattern/match-if-let.html#matches%E5%AE%8F) <a href="#matches-hong" id="matches-hong"></a>

Rust 标准库中提供了一个非常实用的宏：`matches!`，它可以将一个表达式跟模式进行匹配，然后返回匹配的结果 `true` or `false`

下面的示例代码使用match宏从动态数组中筛选类型是 `MyEnum::Foo` 的元素

```
enum MyEnum {
    Foo,
    Bar
}

fn main() {
    let v = vec![MyEnum::Foo,MyEnum::Bar,MyEnum::Foo];
    v.iter().filter(|x| matches!(x, MyEnum::Foo));
}

```

### if let 控制流

相较于match较为冗长的情况处理，if let控制流更为简洁，直接上两个对比代码：

```rust
fn main() {
    let config_max = Some(3u8);
    match config_max {
        Some(max) => println!("The maximum is configured to be {max}"),
        _ => (),
        //match必须有_ => ()保证穷尽性 
    }
}

fn main() {
    let config_max = Some(3u8);
    if let Some(max) = config_max {
        println!("The maximum is configured to be {max}");
    }
}
```

在if let之后可以匹配else表达式

```rust
fn main() {
    let coin = Coin::Penny;
    let mut count = 0;
    if let Coin::Quarter(state) = coin {
        println!("State quarter from {state:?}!");
    } else {
        count += 1;
    }
}
```

