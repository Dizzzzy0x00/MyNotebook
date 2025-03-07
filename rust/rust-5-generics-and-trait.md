---
description: 书接上回
cover: ../.gitbook/assets/01.jpg
coverY: 0
---

# Rust-5 Generics & Trait

## Generics 泛型

在开始讲解 Rust 的泛型之前，先来看看什么是多态

多态的引入是为了解决我们在编程中经常出现的一种需求：**用同一功能的函数处理不同类型的数据**，例如两个数的加法，无论是整数还是浮点数，甚至是自定义类型，都能进行支持。

实际上，泛型就是一种多态。泛型主要目的是为程序员提供编程的便利，减少代码的臃肿，同时可以极大地丰富语言本身的表达能力，假设我们有一个泛型函数 `chop<T>(item: T)`，这个函数的作用是将输入的食材切好备用。泛型 `<T>` 就是任何可以被切割的食材，比如胡萝卜、土豆、洋葱等等。同样的切菜方式，可以适用于各种各样的食材，所以不用写不同的切胡萝卜、切土豆、切洋葱等函数，一个泛型函数 `chop<T>(item: T)` 就能搞定。

**泛型参数**

在之前提到的`chop<T>(item: T)`的 `T` 就是**泛型参数，**&#x51FA;于惯例，用 `T` ( `T` 是 `type` 的首字母)来作为首选，这个名称越短越好，除非需要表达含义。使用泛型参数，有一个先决条件，**必需在使用前对其进行声明，**&#x7136;后才能在函数参数中使用这个泛型参数`item: T`

下面看一个错误示例

```rust
fn largest<T>(list: &[T]) -> T {
    let mut largest = list[0];

    for &item in list.iter() {
        if item > largest {
            largest = item;
        }
    }

    largest
}

fn main() {
    let number_list = vec![34, 50, 25, 100, 65];

    let result = largest(&number_list);
    println!("The largest number is {}", result);

    let char_list = vec!['y', 'm', 'a', 'q'];

    let result = largest(&char_list);
    println!("The largest char is {}", result);
}
```

这段代码编译报错，因为 `T` 可以是任何类型，但不是所有的类型都能进行比较

```bash
error[E0369]: binary operation `>` cannot be applied to type `T` // `>`操作符不能用于类型`T`
 --> src/main.rs:5:17
  |
5 |         if item > largest {
  |            ---- ^ ------- T
  |            |
  |            T
  |
help: consider restricting type parameter `T` // 考虑对T进行类型上的限制 :
  |
1 | fn largest<T: std::cmp::PartialOrd>(list: &[T]) -> T {
  |             ++++++++++++++++++++++
#编译器建议我们给 T 添加一个类型限制：
#使用 std::cmp::PartialOrd 特征（Trait）对 T 进行限制

```

正确的写法：

```rust
fn largest<T: std::cmp::PartialOrd>(list: &[T]) -> T {
    let mut largest = list[0];

    for &item in list.iter() {
        if item > largest {
            largest = item;
        }
    }

    largest
}
```

方法上使用泛型示例代码，注意使用泛型参数前，依然需要提前声明：`impl<T>`，只有提前声明了，我们才能在`Point<T>`中使用它，这样 Rust 就知道 `Point` 的尖括号中的类型是泛型而不是具体类型：

```rust
struct Point<T, U> {
    x: T,
    y: U,
}

impl<T, U> Point<T, U> {
    fn mixup<V, W>(self, other: Point<V, W>) -> Point<T, W> {
        Point {
            x: self.x,
            y: other.y,
        }
    }
}
//不仅能定义基于 T 的方法，还能针对特定的具体类型，进行方法定义
impl Point<f32> {
    fn distance_from_origin(&self) -> f32 {
        (self.x.powi(2) + self.y.powi(2)).sqrt()
    }
}

fn main() {
    let p1 = Point { x: 5, y: 10.4 };
    let p2 = Point { x: "Hello", y: 'c'};

    let p3 = p1.mixup(p2);

    println!("p3.x = {}, p3.y = {}", p3.x, p3.y);
}
```

### const 泛型 <a href="#const-fan-xing-rust151-ban-ben-yin-ru-de-zhong-yao-te-xing" id="const-fan-xing-rust151-ban-ben-yin-ru-de-zhong-yao-te-xing"></a>

以在数组为例，Rust认为：`[i32; 2]` 和 `[i32; 3]` 是不同的数组类型，如果想要对数组长度进行泛型，就需要引入const 泛型

```rust
fn display_array<T: std::fmt::Debug, const N: usize>(arr: [T; N]) {
    println!("{:?}", arr);
}
fn main() {
    let arr: [i32; 3] = [1, 2, 3];
    display_array(arr);

    let arr: [i32; 2] = [1, 2];
    display_array(arr);
}
```

上面的示例代码定义了一个类型为 `[T; N]` 的数组，其中 `T` 是一个基于类型的泛型参数， `N` 是一个基于值的泛型参数，因为它用来替代的是数组的长度。

`N` 就是 const 泛型，定义的语法是 `const N: usize`，表示 const 泛型 `N` ，它基于的值类型是 `usize`。

### 泛型性能与Rust单态化

在 Rust 中泛型是零成本的抽象，意味着你在使用泛型时，完全不用担心性能上的问题，Rust损失编译速度和增大了最终生成文件的大小来换取性能。具体来说，Rust 通过在编译时进行泛型代码的 **单态化**(_monomorphization_)来保证效率。单态化是一个通过填充编译时使用的具体类型，将通用代码转换为特定代码的过程。

我们可以使用泛型来编写不重复的代码，而 Rust 将会为每一个实例编译其特定类型的代码。

## 特征 Trait <a href="#te-zheng-trait" id="te-zheng-trait"></a>

Rust的新的概念，特征定义了**一组可以被共享的行为，只要实现了特征，就能使用这组行为（**&#x7279;征跟接口很类&#x4F3C;**）**

如果不同的类型具有相同的行为，那么我们就可以定义一个特征，然后为这些类型实现该特征。**定义特征**是把一些方法组合在一起，目的是定义一个实现某些目标所必需的行为的集合。

例如，对于文章 `Post` 和微博 `Weibo` 两种内容载体，想对相应的内容进行总结，也就是无论是文章内容，还是微博内容，都可以在某个时间点进行总结，那么总结这个行为就是共享的，下面的代码声明一个特征叫Summary并且为两种类型实现这个特征

```rust
pub trait Summary {//使用 trait 关键字来声明一个特征
    fn summarize(&self) -> String;//只定义特征方法的签名，而不进行实现
    //特征只定义行为看起来是什么样的，而不定义行为具体是怎么样的
}
pub struct Post {
    pub title: String, // 标题
    pub author: String, // 作者
    pub content: String, // 内容
}

impl Summary for Post {
    fn summarize(&self) -> String {
        format!("文章{}, 作者是{}", self.title, self.author)
    }
}

pub struct Weibo {
    pub username: String,
    pub content: String
}

impl Summary for Weibo {
    fn summarize(&self) -> String {
        format!("{}发表了微博{}", self.username, self.content)
    }
}

//在类型上调用特征的方法
fn main() {
    let post = Post{title: "Rust语言简介".to_string(),author: "Sunface".to_string(), content: "Rust棒极了!".to_string()};
    let weibo = Weibo{username: "sunface".to_string(),content: "好像微博没Tweet好用".to_string()};

    println!("{}",post.summarize());
    println!("{}",weibo.summarize());
}
```

### **特征定义与实现的位置(孤儿规则)**

关于特征实现与定义的位置，有一条非常重要的原则：想要为类型 `A` 实现特征 `T`，那么 `A` 或者 `T` **至少有一个是在当前作用域中定义**

例如为之前的 `Post` 类型实现标准库中的 `Display` 特征，这是因为 `Post` 类型定义在当前的作用域中。同时，我们也可以在当前包中为 `String` 类型实现 `Summary` 特征，因为 `Summary` 定义在当前作用域中。

但是你无法在当前作用域中，为 `String` 类型实现 `Display` 特征，因为它们都定义在标准库中，其定义所在的位置都不在当前作用域，跟你半毛钱关系都没有，看看就行了。

该规则被称为**孤儿规则**，可以确保其它人编写的代码不会破坏你的代码，也确保了你不会莫名其妙就破坏了风马牛不相及的代码。

### 默认实现

之前的示例中，特征定义仅仅定义了方法的签名，但是也可以在特征定义具有默认实现的方法，其它类型无需再实现该方法，或者也可以选择重载该方法：

简单易懂，不再赘述

### 使用特征作为函数参数 <a href="#shi-yong-te-zheng-zuo-wei-han-shu-can-shu" id="shi-yong-te-zheng-zuo-wei-han-shu-can-shu"></a>

```rust
pub trait Summary {
    fn summarize(&self) -> String {
        String::from("(Read more...)")
    }
}

pub fn notify(item: &impl Summary) {
//可以使用任何实现了 Summary 特征的类型作为该函数的参数
//同时在函数体内，还可以调用该特征的方法
    println!("Breaking news! {}", item.summarize());
}
```

### 特征约束

虽然 `impl Trait` 这种语法非常好理解，但是实际上它只是一个语法糖，真正的完整书写形式如上所述，形如 `T: Summary` 被称为**特征约束**

```rust
pub fn notify<T: Summary>(item: &T) {
    println!("Breaking news! {}", item.summarize());
}
```

#### 多重约束

```rust
pub fn notify<T: Summary + Display>(item: &T) {}
```

#### **Where 约束**

```rust
fn some_function<T, U>(t: &T, u: &U) -> i32
    where T: Display + Clone,
          U: Clone + Debug
{}
```

### `derive` 派生特征 <a href="#tong-guo-derive-pai-sheng-te-zheng" id="tong-guo-derive-pai-sheng-te-zheng"></a>

在之前的示例中有形如 `#[derive(Debug)]` 的代码，这种是一种特征派生语法，被 `derive` 标记的对象会自动实现对应的默认特征代码，继承相应的功能。

例如 `Debug` 特征，它有一套自动实现的默认代码，当你给一个结构体标记后，就可以使用 `println!("{:?}", s)` 的形式打印该结构体的对象。

再如 `Copy` 特征，它也有一套自动实现的默认代码，当标记到一个类型上时，可以让这个类型自动实现 `Copy` 特征，进而可以调用 `copy` 方法，进行自我复制。

总之，`derive` 派生出来的是 Rust 默认给我们提供的特征，在开发过程中极大的简化了自己手动实现相应特征的需求，当然，如果你有特殊的需求，还可以自己手动重载该实现。

下面是学习了键值对之后写的作业题，放这里拿来当个例子吧

```rust
use std::collections::HashMap;
#[derive(Debug, PartialEq, Eq,Hash)]
struct Viking {
    name: String,
    country: String,
}

impl Viking {
    fn new(name: &str, country: &str) -> Viking {
        Viking {
            name: name.to_string(),
            country: country.to_string(),
        }
    }
}

fn main() {
    // 使用 HashMap 来存储 viking 的生命值
    let vikings = HashMap::from([
        (Viking::new("Einar", "Norway"), 25),
        (Viking::new("Olaf", "Denmark"), 24),
        (Viking::new("Harald", "Iceland"), 12),
    ]);

    // 使用 derive 的方式来打印 viking 的当前状态
    for (viking, health) in &vikings {
        println!("{:?} has {} hp", viking, health);
    }
}
```

在这个例子中，使用了 `#[derive(Debug, PartialEq, Eq, Hash)]` 来为 `Viking` 结构体自动实现了相应的特性。

* `Debug`：使得 `Viking` 结构体可以通过 `{:?}` 进行格式化打印。
* `PartialEq` 和 `Eq`：使得 `Viking` 结构体可以执行相等性比较。这就允许我们基于两个 `Viking` 结构体实例的 `name` 属性和 `country` 属性来判断这两个实例是否相等。
* `Hash`：使 `Viking` 结构体具备被哈希的能力，这样我们就可以将 `Viking` 结构体实例用作 `HashMap` 的键。

此外还有一些常见的derive派生：

* `Clone` 和 `Copy`：用于让类型支持复制和直接拷贝。
* `PartialOrd` 和 `Ord`：为类型实现（部分）顺序性测试。

### 特征对象

看示例代码，**特征对象**指向实现了 `Draw` 特征的类型的实例，也就是指向了 `Button` 或者 `SelectBox` 的实例，这种映射关系是存储在一张表中，可以在运行时通过特征对象找到具体调用的类型方法。

**可以通过 `&` 引用或者 `Box<T>` 智能指针的方式来创建特征对象。**

```rust
trait Draw {
    fn draw(&self) -> String;
}

impl Draw for u8 {
    fn draw(&self) -> String {
        format!("u8: {}", *self)
    }
}

impl Draw for f64 {
    fn draw(&self) -> String {
        format!("f64: {}", *self)
    }
}

// 若 T 实现了 Draw 特征
// 则调用该函数时传入的 Box<T> 可以被隐式转换成函数参数签名中的 Box<dyn Draw>
// Box<T>类似于一个引用，但是包裹的值会被强制分配在堆上
fn draw1(x: Box<dyn Draw>) {
    // 由于实现了 Deref 特征，Box 智能指针会自动解引用为它所包裹的值，然后调用该值对应的类型上定义的 `draw` 方法
    x.draw();
}

fn draw2(x: &dyn Draw) {
    x.draw();
}

```

Rust 编译器需要知道一个函数的返回类型占用多少内存空间。由于特征的不同实现类型可能会占用不同的内存，因此通过 `impl Trait` 返回多个类型是不被允许的，但是我们可以返回一个 `dyn` 特征对象来解决问题。此外，也可以[在数组中使用特征对象](https://practice-zh.course.rs/generics-traits/trait-object.html#%E5%9C%A8%E6%95%B0%E7%BB%84%E4%B8%AD%E4%BD%BF%E7%94%A8%E7%89%B9%E5%BE%81%E5%AF%B9%E8%B1%A1)

```rust
trait Bird {
    fn quack(&self) -> String;
}

struct Duck;
impl Duck {
    fn swim(&self) {
        println!("Look, the duck is swimming")
    }
}
struct Swan;
impl Swan {
    fn fly(&self) {
        println!("Look, the duck.. oh sorry, the swan is flying")
    }
}

impl Bird for Duck {
    fn quack(&self) -> String{
        "duck duck".to_string()
    }
}

impl Bird for Swan {
    fn quack(&self) -> String{
        "swan swan".to_string()
    }
}

fn main() {
    // 填空
    let duck = Duck{};
    duck.swim();

    let bird = hatch_a_bird(2);
    // 变成鸟儿后，它忘记了如何游，因此以下代码会报错
    // bird.swim();
    // 但它依然可以叫唤
    println!("{}", bird.quack());
    assert_eq!(bird.quack(), "duck duck");

    let bird = hatch_a_bird(1);
    // 这只鸟儿忘了如何飞翔，因此以下代码会报错
    // bird.fly();
    // 但它也可以叫唤
    assert_eq!(bird.quack(), "swan swan");
    let birds: Vec<Box<dyn Bird>> = vec![
        Box::new(Duck{}),
        Box::new(Duck{}),
        Box::new(Swan{})
    ];

    for bird in birds {
        bird.quack();
        // 当 duck 和 swan 变成 bird 后，它们都忘了如何翱翔于天际，只记得该怎么叫唤了。。
        // 因此，以下代码会报错
        // bird.fly();
    }
    println!("Success!")
}  

fn hatch_a_bird(birdd:i32) -> Box<dyn Bird>{
    if birdd==1{
        Box::new(Swan {})
    }else {
        Box::new(Duck {})
    }
    
}
```

#### 特征对象的动态分发

当使用特征对象时，Rust 必须使用动态分发。编译器无法知晓所有可能用于特征对象代码的类型，所以它也不知道应该调用哪个类型的哪个方法实现。为此，Rust 在运行时使用特征对象中的指针来知晓需要调用哪个方法。动态分发也阻止编译器有选择的内联方法代码，这会相应的禁用一些优化

<figure><img src="../.gitbook/assets/image (80).png" alt=""><figcaption></figcaption></figure>

不是所有特征都能拥有特征对象，只有对象安全的特征才行。当一个特征的所有方法都有如下属性时，它的对象才是安全的：

* 方法的返回类型不能是 `Self`
* 方法没有任何泛型参数

### Rust的`self` 和 `Self`&#x20;

在Rust中，`self` 和 `Self` 有着完全不同的用途：

1. `self`：主要在关联函数（即方法）中使用，**用于表示实例本身的引用**。根据需要使用 `self`，`&self`，`&mut self` 或 `Box<Self>` 以表明实例的所有权，可以借用（引用）或者可变借用。例如：

```rust
struct User {
    name: String
}

impl User {
   fn print_name(&self) {
      println!("{}", self.name);
   }
}
```

在上述代码中，`&self` 表示 `print_name` 函数借用了 `User` 实例，不会用完后释放实例。

2. `Self`：在trait或impl块中使用，**表示实现该trait或者impl的类型本身**。它在写可以被多个类型实现的trait或者实现方法时十分可用。例如：

```rust
trait Animal {
   fn create(name: String) -> Self;
}

impl Animal for User {
    fn create(name: String) -> Self {     //这里的 Self 就是 User 类型
       User {name}
    }
}
```

在上述代码中，`Self`在`trait Animal`中表示任何实现`Animal`的类型。在`impl Animal for User`中，`Self`就代表了`User`类型。
