---
title: Rust实战1 -- 成绩判断
---

这里我们构造一个简单的学生成绩管理系统，通过定义学生信息，演示下Rust中基本类型以及枚举和结构体的使用。

```bash
cargo new stu_manager
```



## 变量定义

在 src/main.rs 中定义并打印学生信息

```rust
#[derive(Debug)]
enum Sex {
    Boy,
    Girl,
}

struct Student {
    name: String, // 姓名
    age: u16,     // 年龄
    sex: Sex,     // 性别
    score: f32,   // 成绩
}

fn main() {
    let students: [Student; 2] = [
        Student {
            name: String::from("boy01"),
            age: 18,
            sex: Sex::Boy,
            score: 61.5,
        },
        Student {
            name: String::from("girl01"),
            age: 16,
            sex: Sex::Girl,
            score: 91.5,
        },
    ];

    display(&students[0]);
    display(&students[1]);
}

fn display(stu: &Student) {
    println!(
        "name: {}, age: {}, sex:{:?}, score: {}",
        stu.name, stu.age, stu.sex, stu.score
    );
}
```

1. `#[derive(Debug)]`这个是为了打印enum，否则enum类型是不能直接打印的。
2. `struct Student`这个结构体中定义了常用的基本类型的使用方式。
3. `let students: [Student; 2]`Rust数组的定义方式。



## 流程控制

掌握了变量定义，可以组织我们的数据，再掌握Rust中的流程控制方法，那么，就能实现实际的业务功能了。

流程控制主要两种：分支和循环。



### 分支

Rust的分支语法有 `if`和`match`两种方式。

继续完善上面的例子，我们增加一个根据成绩区分优良中差的函数，用 `if`的方式来判断分支。

```rust
fn check_score(stu: &Student) {
    if stu.score >= 90.0 {
        println!("学员：{}, 成绩优秀", stu.name);
    } else if stu.score < 90.0 && stu.score >= 75.0 {
        println!("学员：{}, 成绩良好", stu.name);
    } else if stu.score < 75.0 && stu.score >= 60.0 {
        println!("学员：{}, 成绩中等", stu.name);
    } else {
        println!("学员：{}, 成绩不合格!!!", stu.name);
    }
}
```

再增加一个判断性别的函数，用`match`的方式来判断分支。

```rust
fn check_sex(stu: &Student) {
    match stu.sex {
        Sex::Boy => println!("学员: {} 是男生", stu.name),
        Sex::Girl => println!("学员: {} 是女生", stu.name),
    }
}
```



### 循环

Rust 循环主要有3种方式：

1. loop 无限循环，自己控制循环退出
2. while 条件循环
3. for 条件循环

下面用3种循环方式分别打印学生信息，学生成绩信息以及学生性别信息。

```rust
    // loop 循环示例
    let mut count = 0;
    loop {
        if count == students.len() {
            break;
        }

        display(&students[count]);
        count += 1;
    }

    // while 循环示例
    count = 0;
    while count < students.len() {
        check_score(&students[count]);
        count += 1;
    }

    // for 循环示例
    for stu in students {
        check_sex(&stu);
    }
```

3种循环中，还是 `for`循环最为见解，这也是我们使用最多的循环方式。



## 完整程序

```rust
#[derive(Debug)]
enum Sex {
    Boy,
    Girl,
}

struct Student {
    name: String, // 姓名
    age: u16,     // 年龄
    sex: Sex,     // 性别
    score: f32,   // 成绩
}

fn main() {
    let students: [Student; 2] = [
        Student {
            name: String::from("boy01"),
            age: 18,
            sex: Sex::Boy,
            score: 61.5,
        },
        Student {
            name: String::from("girl01"),
            age: 16,
            sex: Sex::Girl,
            score: 91.5,
        },
    ];

    // loop 循环示例
    let mut count = 0;
    loop {
        if count == students.len() {
            break;
        }

        display(&students[count]);
        count += 1;
    }

    // while 循环示例
    count = 0;
    while count < students.len() {
        check_score(&students[count]);
        count += 1;
    }

    // for 循环示例
    for stu in students {
        check_sex(&stu);
    }
}

fn display(stu: &Student) {
    println!(
        "name: {}, age: {}, sex:{:?}, score: {}",
        stu.name, stu.age, stu.sex, stu.score
    );
}

fn check_score(stu: &Student) {
    if stu.score > 100.0 {
        println!("学员：{}, 成绩错误", stu.name);
    } else if stu.score <= 100.0 && stu.score >= 90.0 {
        println!("学员：{}, 成绩优秀", stu.name);
    } else if stu.score < 90.0 && stu.score >= 75.0 {
        println!("学员：{}, 成绩良好", stu.name);
    } else if stu.score < 75.0 && stu.score >= 60.0 {
        println!("学员：{}, 成绩中等", stu.name);
    } else {
        println!("学员：{}, 成绩不合格!!!", stu.name);
    }
}

fn check_sex(stu: &Student) {
    match stu.sex {
        Sex::Boy => println!("学员: {} 是男生", stu.name),
        Sex::Girl => println!("学员: {} 是女生", stu.name),
    }
}
```

