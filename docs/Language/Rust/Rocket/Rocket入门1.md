---
title: Rocket光速入门1
---

## Rocket光速入门1

选择版本：0.4.2

### 1. Rust切换nightly

```sh
rustup default nightly
```

### 2. Hello World

在cargo.toml添加

```toml
[dependencies]
rocket = "0.4.2"
```

修改src/main.rs内容

```rust
#![feature(proc_macro_hygiene, decl_macro)]

#[macro_use] extern crate rocket;

#[get("/")]
fn index() -> &'static str {
    "Hello, world!"
}

fn main() {
    rocket::ignite().mount("/", routes![index]).launch();
}
```

创建一条`index`路由，将该路由 ***绑定*** 在 `/` 路径上，然后启动应用程序。

### 生命周期

Rocket的主要任务是侦听传入的Web请求，将请求分配给应用程序代码，并将响应返回给客户端。我们将从请求到响应的过程称为“生命周期”。我们将生命周期总结为以下步骤序列：

1. **路由**

    Rocket将传入的HTTP请求解析为本机结构，您的代码将在本机结构中间接操作。Rocket通过与您的应用程序中声明的路由属性进行匹配来确定要调用的请求处理程序。

2. **验证方式**

    Rocket会根据匹配路径中存在的类型和防护来验证传入的请求。如果验证失败，Rocket *会将*请求*转发*到下一个匹配的路由，或者调用*错误处理程序*。

3. **处理中**

    与路由关联的请求处理程序将使用经过验证的参数来调用。这是应用程序的主要业务逻辑。通过返回来完成处理`Response`。

4. **响应**

    返回的`Response`已处理。Rocket会生成适当的HTTP响应，并将其发送到客户端。这样就完成了生命周期。Rocket继续侦听请求，并为每个传入请求重新启动生命周期。



### 3. 路由

```rust
#[get("/world")]              // <- 路由属性
fn world() -> &'static str {  // <- 请求处理程序
    "Hello, world!"
}
```



### 4. 挂载

```rust
fn main() {
    rocket::ignite().mount("/hello", routes![world]);
}
```

`mount`方法接受以下输入：

1. 一个基本路径，作为一个其下包含路由列表的命名空间，这里的例子中为`"/hello"`。
2. 一个路由列表，通过`routes!`宏生成：这里的例子中为`routes![world]`，其中可包含多条路由：`routes![a, b, c]`。

这将通过`ignite`函数创建一个新的Rocket实例，并将`world`路由挂载到“/hello”路径，使Rocket知道该路径。对“/hello/world”的GET请求将指向world函数。

**注意：在许多情况下，基本路径将只是`"/"`**



### 5.启动

```rust
#![feature(proc_macro_hygiene, decl_macro)]

#[macro_use] extern crate rocket;

#[get("/world")]
fn world() -> &'static str {
    "Hello, world!"
}

fn main() {
    rocket::ignite().mount("/hello", routes![world]).launch();
}
```



## 请求

```rust
#[get("/world")]
fn handler() { .. }
```

这个路由指定了它仅匹配到 `/world` 的`GET`请求。



## 请求方法 Methods

Rocket route属性可以是`get`、`put`、`post`、`delete`、`head`、`patch`或`options`中的任意一个，每个属性都对应于要匹配的HTTP方法。

```rust
#[post("/")]
```



## 动态路径

```rust
#[get("/hello/<name>")]
fn hello(name: &RawStr) -> String {
    format!("Hello, {}!", name.as_str())
}
```

如果将路由挂载在根（`.mount("/", routes![hello])`）上，则具有两个非空段的路径的任何请求（第一个段为`hello`）将被分派到该`hello`路由。例如，如果我们要访问`/hello/John`, 这个程序将响应 `Hello, John!`。

```rust
#![feature(proc_macro_hygiene, decl_macro)]

#[macro_use] extern crate rocket;
use rocket::http::RawStr;

#[get("/hello/<name>")]
fn hello(name: &RawStr) -> String {
    format!("Hello, {}!", name.as_str())
}

fn main() {
    rocket::ignite().mount("/", routes![hello]).launch();
}
```

