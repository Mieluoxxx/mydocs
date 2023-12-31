---
title: Rust基础6 特征trait
---

### 特征定义

Trait(特征),Rust 编译器一个特定类型具有并且可以与其他类型共享的功能。我们可以使用特征以抽象的方式定义共享行为。我们可以使用特征边界来指定泛型类型可以是具有特定行为的任何类型。

> 类似其他语言的接口

现在用写文章，博客或者微博的总结来做一个例子：

```rust
pub trait Summary {
    fn summarize(&self) -> String;
}
```

定义一个总结的特征。接下来把他分别应用到一篇文章和twitter上去看看。

```rust
pub struct NewsArticle {
    pub headline: String,
    pub location: String,
    pub author: String,
    pub content: String,
}

impl Summary for NewsArticle {
    fn summarize(&self) -> String {
        format!("{}, by {} ({})", self.headline, self.author, self.location)
    }
}

pub struct Tweet {
    pub username: String,
    pub content: String,
    pub reply: bool,
    pub retweet: bool,
}

impl Summary for Tweet {
    fn summarize(&self) -> String {
        format!("{}: {}", self.username, self.content)
    }
}
```

调用的代码如下:

```rust
use aggregator::{Summary, Tweet};

fn main() {
    let tweet = Tweet {
        username: String::from("horse_ebooks"),
        content: String::from(
            "of course, as you probably already know, people",
        ),
        reply: false,
        retweet: false,
    };

    println!("1 new tweet: {}", tweet.summarize());
}
```

显示结果如下:



> 1 new tweet: horse_ebooks: of course, as you probably already know, people.



### 默认实现

默认实现 你可以在特征中定义具有默认实现的方法，这样其它类型无需再实现该方法，也可以选择重载该方法实现定制：

```rust
pub trait Summary {
    fn summarize(&self) -> String {
        String::from("(Read more...)")
    }
}
```

上面为 Summary 定义了一个默认实现，下面我们编写段代码来测试下：

```rust
pub struct Post {
    pub title: String, // 标题
    pub author: String, // 作者
    pub content: String, // 内容
}

pub struct Weibo {
    pub username: String,
    pub content: String
}

impl Summary for Post {}

impl Summary for Weibo {
    fn summarize(&self) -> String {
        format!("{}发表了微博{}", self.username, self.content)
    }
}
```

可以看到，Post 选择了默认实现，而 Weibo 重载了该方法，调用和输出如下：





```rust
fn main() {
    let post = Post{title: "Rust语言简介".to_string(),author: "Sunface".to_string(), content: "Rust棒极了!".to_string()};
    let weibo = Weibo{username: "sunface".to_string(),content: "好像微博没Tweet好用".to_string()};

    println!("{}",post.summarize());
    println!("{}",weibo.summarize());
}
```

结果如下：

```bash
(Read more...)
sunface发表了微博好像微博没Tweet好用
```



还可以把默认方向写成下面的形式:

```rust
pub trait Summary {
    fn summarize_author(&self) -> String;

    fn summarize(&self) -> String {
        format!("(Read more from {}...)", self.summarize_author())
    }
}
```

重载summarize_author

```rust
impl Summary for Weibo {
    fn summarize_author(&self) -> String {
        format!("{}", self.username)
    }
}
....
println!("{}",weibo.summarize()); //调用的时候还是调用summarize
 ....
```



### 特征作为参数

参考代码:

```rust
pub fn notify(item: &impl Summary) {
    println!("Breaking news! {}", item.summarize());
}
```

要适应更长的代码，建议定义成绑定形式：

```rust
pub fn notify<T: Summary>(item: &T) {
    println!("Breaking news! {}", item.summarize());
}
```

这样长语句，就可以这样写:

```rust
pub fn notify(item1: &impl Summary, item2: &impl Summary) {
pub fn notify<T: Summary>(item1: &T, item2: &T) {
```



#### 还可以使用+来使用多个特征



```rust
pub fn notify(item: &(impl Summary + Display)) {
pub fn notify<T: Summary + Display>(item: &T) {
```



使用过多的 trait bound  有其缺点。每个泛型都有自己的特征边界，因此具有多个泛型类型参数的函数可以在函数名称和参数列表之间包含大量特征绑定信息，从而使函数签名难以阅读。where出于这个原因，Rust 有替代语法用于在函数签名之后的子句内指定特征边界。所以不要写这个：



```rust
fn some_function<T: Display + Clone, U: Clone + Debug>(t: &T, u: &U) -> i32 {
```

我们可以使用一个where子句，像这样：





```rust
fn some_function<T, U>(t: &T, u: &U) -> i32
    where T: Display + Clone,
          U: Clone + Debug
{
```

这个函数的签名不那么杂乱：函数名、参数列表和返回类型很接近，类似于没有很多特征边界的函数。



#### 函数返回值可以是实现了特征的类型

本章节前面的例子:

```rust
impl Summary for Tweet {
```

Tweet实现了Summary特征。因此函数返回值可以写成下面的形式:

```rust
fn returns_summarizable() -> impl Summary {
    Tweet {
        username: String::from("horse_ebooks"),
        content: String::from(
            "of course, as you probably already know, people",
        ),
        reply: false,
        retweet: false,
    }
}
```



但是，只能impl Trait在返回单一类型时使用。例如，此代码返回 aNewsArticle或 Tweet且返回类型指定为  impl Summary将不起作用：

此代码无法编译！

```rust
fn returns_summarizable(switch: bool) -> impl Summary {
    if switch {
        NewsArticle {
            headline: String::from(
                "Penguins win the Stanley Cup Championship!",
            ),
            location: String::from("Pittsburgh, PA, USA"),
            author: String::from("Iceburgh"),
            content: String::from(
                "The Pittsburgh Penguins once again are the best \
                 hockey team in the NHL.",
            ),
        }
    } else {
        Tweet {
            username: String::from("horse_ebooks"),
            content: String::from(
                "of course, as you probably already know, people",
            ),
            reply: false,
            retweet: false,
        }
    }
}
```

这时候即使Tweet和NewsArticle 都实现了Summary特征编译器也是不让通过的。 后面的章节会有解决办法。或者按照提示都用Box来解决。