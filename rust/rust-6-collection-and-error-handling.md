---
description: 书接上回
cover: ../.gitbook/assets/01.jpg
coverY: 0
---

# Rust-6 Collection & Error Handling

## Common Collections 集合类型  <a href="#ji-he-lei-xing" id="ji-he-lei-xing"></a>

最常见的三种集合类型：

* _vector_ 即动态数组，存储可变数量的值
* _string_ 字符的集合
* _hashmap_ 将值与特定键关联起来

### Vectors 动态数组 <a href="#storing-lists-of-values-with-vectors" id="storing-lists-of-values-with-vectors"></a>



<pre class="language-rust"><code class="lang-rust">
fn main() {
    let v: Vec&#x3C;i32> = Vec::new();//v 被显式地声明了类型 Vec&#x3C;i32>
    //使用 Vec::new 创建动态数组,调用了 Vec 中的 new 关联函数
    
    //还可以使用宏 vec! 来创建数组，同时给予初始化值
    let v = vec![1, 2, 3];
    
    //更新Vectors：
    let mut v = Vec::new();//与其它类型一样，必须将 v 声明为 mut 后，才能进行修改
<strong>    v.push(1);//使用 push 方法向数组尾部添加元素
</strong>}
</code></pre>

{% hint style="info" %}
动态数组意味着我们增加元素时，如果**容量不足就会导致 vector 扩容**（目前的策略是重新申请一块 2 倍大小的内存，再将所有元素拷贝到新的内存位置，同时更新指针数据），显然，当频繁扩容或者当元素数量较多且需要扩容时，大量的内存拷贝会降低程序的性能。

如果预先知道要存储的元素个数，可以使用 `Vec::with_capacity(capacity)` 创建动态数组，这样可以避免因为插入大量新数据导致频繁的内存分配和拷贝，提升性能

```rust
fn main() {
    let mut v = Vec::with_capacity(10);
    //在初始化时就指定一个实际的预估容量，尽量减少可能的内存拷贝
    v.extend([1, 2, 3]);    // 附加数据到 v
    println!("Vector 长度是: {}, 容量是: {}", v.len(), v.capacity());

    v.reserve(100);        // 调整 v 的容量，至少要有 100 的容量
    println!("Vector（reserve） 长度是: {}, 容量是: {}", v.len(), v.capacity());

    v.shrink_to_fit();     // 释放剩余的容量，一般情况下，不会主动去释放容量
    println!("Vector（shrink_to_fit） 长度是: {}, 容量是: {}", v.len(), v.capacity());
}
```
{% endhint %}

读取指定位置的元素有两种方式可选：

* 通过下标索引访问。
* 使用 `get` 方法。

```rust
fn main() {
    let v = vec![1, 2, 3, 4, 5];
    
    let third: &i32 = &v[2]; //通过下标索引访问
    println!("第三个元素是 {}", third);
    
    match v.get(2) 
    //使用 get 方法，返回Option<&T>，还需要额外的 match 来匹配解构出具体的值
    { 
        Some(third) => println!("第三个元素是 {third}"),
        None => println!("去你的第三个元素，根本没有！"),
    }
    
    //迭代遍历：
    for i in &v {
        println!("{i}");
    }

}
```

Vector的一些常见方法示例

{% embed url="https://doc.rust-lang.org/std/vec/struct.Vec.html" %}

```rust

fn main() {
    let mut v =  vec![1, 2];
    assert!(!v.is_empty());         // 检查 v 是否为空
    
    v.insert(2, 3);                 // 在指定索引插入数据，索引值不能大于 v 的长度， v: [1, 2, 3] 
    assert_eq!(v.remove(1), 2);     // 移除指定位置的元素并返回, v: [1, 3]
    assert_eq!(v.pop(), Some(3));   // 删除并返回 v 尾部的元素，v: [1]
    assert_eq!(v.pop(), Some(1));   // v: []
    assert_eq!(v.pop(), None);      // pop 方法返回的是 Option 枚举值
    v.clear();                      // 清空 v, v: []
    
    let mut v1 = [11, 22].to_vec(); // append 操作会导致 v1 清空数据，增加可变声明
    v.append(&mut v1);              // 将 v1 中的所有元素附加到 v 中, v1: []
    v.truncate(1);                  // 截断到指定长度，多余的元素被删除, v: [11]
    v.retain(|x| *x > 10);          // 保留满足条件的元素，即删除不满足条件的元素
    
    let mut v = vec![11, 22, 33, 44, 55];
    // 删除指定范围的元素，同时获取被删除元素的迭代器, v: [11, 55], m: [22, 33, 44]
    let mut m: Vec<_> = v.drain(1..=3).collect();    
    
    let v2 = m.split_off(1);        // 指定索引处切分成两个 vec, m: [22], v2: [33, 44]
    
    //像数组切片的方式获取 vec 的部分元素
    let v3 = vec![11, 22, 33, 44, 55];
    let slice = &v3[1..=3];
    assert_eq!(slice, &[22, 33, 44]);
}
```



#### Vector数组排序

在 rust 里，实现了两种排序算法，分别为稳定的排序 `sort` 和 `sort_by`，以及非稳定排序 `sort_unstable` 和 `sort_unstable_by`

```rust
fn main() {
    //整数数组的排序
    let mut vec = vec![1, 5, 10, 2, 15];    
    vec.sort_unstable();    
    assert_eq!(vec, vec![1, 2, 5, 10, 15]);
    //浮点数数组的排序
    let mut vec_float = vec![1.0, 5.6, 10.3, 2.0, 15f32];    
    //使用vec_float.sort_unstable()会无法编译：
    //在浮点数当中，存在一个 NAN 的值，这个值无法与其他的浮点数进行对比
    //浮点数类型并没有实现全数值可比较 Ord 的特性，而是实现了部分可比较的特性 PartialOrd
    //如果确定浮点数数组中不包含 NAN 值，可以使用 partial_cmp
    vec_float.sort_unstable_by(|a, b| a.partial_cmp(b).unwrap());    
    assert_eq!(vec_float, vec![1.0, 2.0, 5.6, 10.3, 15f32]);
}
```

#### `From<T>` 特征的类型转换Vector

只要为 `Vec` 实现了 `From<T>` 特征，那么 `T` 就可以被转换成 `Vec`

```rust
fn main() {
    // array -> Vec
    // impl From<[T; N]> for Vec
    let arr = [1, 2, 3];
    let v1 = Vec::from(arr);
    let v2: Vec<i32> = arr.into();
 
    assert_eq!(v1, v2);
 
    
    // String -> Vec
    // impl From<String> for Vec
    let s = "hello".to_string();
    let v1: Vec<u8> = s.into();

    let s = "hello".to_string();
    let v2 = s.into_bytes();
    assert_eq!(v1, v2);

    // impl<'_> From<&'_ str> for Vec
    let s = "hello";
    let v3 = Vec::from(s);
    assert_eq!(v2, v3);

    // 迭代器 Iterators 可以通过 collect 变成 Vec
    let v4: Vec<i32> = [0; 10].into_iter().collect();
    assert_eq!(v4, vec![0; 10]);

    println!("Success!")
 }
```

### HashMap 键值对

Rust 中哈希类型（哈希映射）为 `HashMap<K,V>`，跟创建动态数组 `Vec` 的方法类似，可以使用 `new` 方法来创建 `HashMap`，然后通过 `insert` 方法插入键值对，使用HashMap需要引用标准库`std::collections::HashMap`

```rust
use std::collections::HashMap;
fn main{
    let mut kv_map = HashMap::new();
    kvmap.insert("k",v);
}
```

HashhMap的所有权：

对于实现了特征的类型`Copy`，比如`i32`，值会被复制到哈希映射中。对于拥有的值，比如`String`，值会被移动，而哈希映射将成为这些值的所有者

{% hint style="info" %}
跟 `Vec` 一样，如果预先知道要存储的 `KV` 对个数，可以使用 `HashMap::with_capacity(capacity)` 创建指定大小的 `HashMap`，避免频繁的内存分配和拷贝，提升性能。

可以用方法`shrink_to_fit()`让 Rust 自行调整到一个合适的值

```rust
use std::collections::HashMap;
fn main() {
    let mut map: HashMap<i32, i32> = HashMap::with_capacity(100);
    map.insert(1, 2);
    map.insert(3, 4);
    // 事实上，虽然我们使用了 100 容量来初始化，但是 map 的容量很可能会比 100 更多
    assert!(map.capacity() >= 100);

    // 对容量进行收缩，你提供的值仅仅是一个允许的最小值，实际上，Rust 会根据当前存储的数据量进行自动设置，当然，这个值会尽量靠近你提供的值，同时还可能会预留一些调整空间

    map.shrink_to(50);
    assert!(map.capacity() >= 50);

    // 让 Rust  自行调整到一个合适的值，剩余策略同上
    map.shrink_to_fit();
    assert!(map.capacity() >= 2);
    println!("Success!")
}
```
{% endhint %}

除了使用new创建HashMap之外，还可以使用迭代器和collection方法进行创建，先将 `Vec` 转为迭代器，接着通过 `collect` 方法，将迭代器中的元素收集后，转成 `HashMap`，参考下面的示例代码：

<pre class="language-rust"><code class="lang-rust">use std::collections::HashMap;
<strong>fn main() {
</strong>    use std::collections::HashMap;

    let teams_list = vec![
        ("中国队".to_string(), 100),
        ("美国队".to_string(), 10),
        ("日本队".to_string(), 50),
    ];

    let teams_map: HashMap&#x3C;_,_> = teams_list.into_iter().collect();
    
    println!("{:?}",teams_map)
    
    let team_name = String::from("中国队");
    let score: Option&#x3C;&#x26;i32> = teams_map.get(&#x26;team_name);
    
    //通过循环的方式依次遍历 KV 对
    for (key, value) in &#x26;teams_map {
        println!("{}: {}", key, value);
    }
}
</code></pre>

在上面的例子也演示了通过 `get` 方法获取元素：

* `get` 方法返回一个 `Option<&i32>` 类型：当查询不到时，会返回一个 `None`，查询到时返回 `Some(&i32)`
* `&i32` 是对 `HashMap` 中值的借用，如果不使用借用，可能会发生所有权的转移，**如果我们想直接获得值类型的 `score`可以这么写：**`let score: i32 = scores.get(&team_name).copied().unwrap_or(0);`

值的更新：

```rust
fn main() {
    use std::collections::HashMap;

    let mut scores = HashMap::new();

    scores.insert("Blue", 10);

    // 覆盖已有的值
    let old = scores.insert("Blue", 20);
    assert_eq!(old, Some(10));

    // 查询新插入的值
    let new = scores.get("Blue");
    assert_eq!(new, Some(&20));

    // 查询Yellow对应的值，若不存在则插入新值
    let v = scores.entry("Yellow").or_insert(5);
    assert_eq!(*v, 5); // 不存在，插入5

    // 查询Yellow对应的值，若不存在则插入新值
    let v = scores.entry("Yellow").or_insert(50);
    assert_eq!(*v, 5); // 已经存在，因此50没有插入
    
    let text = "hello world wonderful world";


//查询某个 key 对应的值，
//若不存在则插入新值，若存在则对已有的值进行更新，例如在文本中统计词语出现的次数
    let mut map = HashMap::new();
    // 根据空格来切分字符串(英文单词都是通过空格切分)
    for word in text.split_whitespace() {
        let count = map.entry(word).or_insert(0);
        *count += 1;
    }
    
    println!("{:?}", map)
}
```

目前，`HashMap` 使用的哈希函数是 `SipHash`，它的性能不是很高，但是安全性很高。`SipHash` 在中等大小的 `Key` 上，性能相当不错，但是对于小型的 `Key` （例如整数）或者大型 `Key` （例如字符串）来说，性能还是不够好。若你需要极致性能，例如实现算法，可以考虑库[ahash](https://github.com/tkaitchuck/ahash)。



## [Error Handling](https://doc.rust-lang.org/book/ch09-00-error-handling.html#error-handling) <a href="#error-handling" id="error-handling"></a>

Rust 将错误分为两大类：

* **可恢复错误**，通常用于从系统全局角度来看可以接受的错误，例如处理用户的访问、操作等错误，这些错误只会影响某个用户自身的操作进程，而不会对系统的全局稳定性产生影响
* **不可恢复错误**，刚好相反，该错误通常是全局性或者系统性的错误，例如数组越界访问，系统启动时发生了影响启动流程的错误等等，这些错误的影响往往对于系统来说是致命的

很多编程语言，并不会区分这些错误，而是直接采用异常的方式去处理。Rust 没有异常，但是 Rust 也有自己的卧龙凤雏：`Result<T, E>` 用于可恢复错误，`panic!` 用于不可恢复错误。

### Panic
