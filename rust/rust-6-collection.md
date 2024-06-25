---
description: 书接上回
cover: ../.gitbook/assets/01.jpg
coverY: 0
---

# Rust-6 Collection

## Common Collections 集合类型  <a href="#ji-he-lei-xing" id="ji-he-lei-xing"></a>

最常见的三种集合类型：

* _vector_ 即动态数组，存储可变数量的值
* _string_ 字符的集合
* _hash map_ 将值与特定键关联起来

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
如果预先知道要存储的元素个数，可以使用 `Vec::with_capacity(capacity)` 创建动态数组，这样可以避免因为插入大量新数据导致频繁的内存分配和拷贝，提升性能
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
