# match和if let

先来看一个关于`match`的简单例子:
```rust
enum Direction {
    East,
    West,
    North,
    South,
}

fn main() {
    let dire = Direction::South;
    match dire {
        Direction::East => println!("East"),
        Direction::North | Direction::South => {
            println!("South or North");
        },
        _ => println!("West"),
    };
}
```

这里我们想去匹配`dire`对应的枚举类型，因此在match中用三个匹配分支来完全覆盖枚举变量`Direction`的所有成员类型，有以下几点值得注意：
- `match`的匹配必须要穷举出所有可能,因此这里用`_`来代表其余的所有可能性
- `match`的每一个分支都必须是一个表达式，且所有分支的表达式最终返回值的类型必须相同
- **X | Y**, 是逻辑运算符`或`,代表该分支可以匹配`X`也可以匹配`Y`，只要满足一个即可


其实`match`跟其他语言中的`switch`非常像,例如`_`类似`switch`中的`default`。

## `match`匹配

`match`允许我们将一个值与一系列的模式相比较，并根据相匹配的模式执行对应的代码，下面让我们来一一详解，先看一个例子：
```rust
enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter,
}

fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny =>  {
            println!("Lucky penny!");
            1
        },
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter => 25,
    }
}
```

`value_in_cents`函数根据匹配到的硬币类似，返回对应的美分数值，`match`后紧跟着的是一个表达式，跟`if`很像，但是`if`后的表达式必须是一个布尔值，而`match`后的表达式返回值可以是任意类型，只要能跟后面的分支匹配起来即可，这里的`coin`是枚举`Coin`类型。

接下来是`match`的分支。一个分支有两个部分：一个模式和针对该模式的处理代码。第一个分支的模式是`Coin::Penny`而之后的`=>`运算符将模式和将要运行的代码分开。这里的代码就仅仅是表达式`1`, 不同分支之间使用逗号分隔。

当`match`表达式执行时，它将目标值`coin`按顺序与每一个分支的模式相比较, 如果模式匹配了这个值，那么模式之后的代码将被执行。如果模式并不匹配这个值，将继续执行下一个分支。

每个分支相关联的代码是一个表达式，而表达式的结果值将作为整个 match 表达式的返回值。如果分支有多行代码，那么需要用`{}`包裹，同时最后一行代码需要是一个表达式。

#### 使用`match`表达式赋值
还有一点很重要，`match`本身也是一个表达式，因此可以用它来赋值：
```rust
enum IpAddr {
   Ipv4,
   Ipv6
}

fn main() {
    // let d_panic = Direction::South;
    let ip1 = IpAddr::Ipv6;
    let ip_str = match ip1 {
        IpAddr::Ipv4 => "127.0.0.1",
        _ => "::1",
    };

    println!("{}", ip_str);
}
```
这里因为匹配到`_`分支，因此将`"::1"`赋值给了`ip_str`.

#### 模式绑定

匹配分支的另外一个重要功能是从模式中取出绑定的值，例如：
```rust
#[derive(Debug)]
enum UsState {
    Alabama,
    Alaska,
    // --snip--
}

enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter(UsState), // 25美分硬币
}
```
其中Coin::Quarter成员还存放了一个值：美国的某个州，因为在1999年到2008年间，美国在25美分(Quater)硬币的背后为50个州印刷了不同的设计。其它硬币都没有相关的设计。

接下来，我们希望在模式匹配中，获取到25美分硬币上刻印的州的名称：
```rust
fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => 1,
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter(state) => { 
            println!("State quarter from {:?}!", state);
            25
        },
    }
}
```
上面代码中，在匹配`Coin::Quarter`模式时，我们把它内部存储的值绑定到了`state`变量上，因此`state`变量就是对应的`UsState`枚举类型。

例如有一个印了阿拉斯加州标记的25分硬币：`Coin::Quarter(UsState::Alaska))`, 它在匹配时，`state`变量将被绑定`UsState::Alaska`的枚举值。

再来看一个更复杂的例子：
```rust
enum Action {
    Say(String),
    MoveTo(i32, i32),
    ChangeColorRGB(u16, u16, u16),
}

fn main() {
    let actions = [ 
        Action::Say("Hello Rust".to_string()),
        Action::MoveTo(1,2),
        Action::ChangeColorRGB(255,255,0),
    ];
    for action in actions {
        match action {
            Action::Say(s) => {
                println!("{}", s);
            },
            Action::MoveTo(x, y) => {
                println!("point from (0, 0) move to ({}, {})", x, y);
            },
            Action::ChangeColorRGB(r, g, _) => {
                println!("change color into '(r:{}, g:{}, b:0)', 'b' has been ignored",
                    r, g,
                );
            }
        }
    }
}
```

运行后输出：
```console
$ cargo run
   Compiling world_hello v0.1.0 (/Users/sunfei/development/rust/world_hello)
    Finished dev [unoptimized + debuginfo] target(s) in 0.16s
     Running `target/debug/world_hello`
Hello Rust
point from (0, 0) move to (1, 2)
change color into '(r:255, g:255, b:0)', 'b' has been ignored
```

#### 穷尽匹配
在文章的开头，我们简单总结过`match`的匹配必须穷尽所有情况，下面来举例说明，例如:
```rust
enum Direction {
    East,
    West,
    North,
    South,
}

fn main() {
    let dire = Direction::South;
    match dire {
        Direction::East => println!("East"),
        Direction::North | Direction::South => {
            println!("South or North");
        },
    };
}
```

我们没有处理`Direction::West`的情况，因此会报错：
```rust
error[E0004]: non-exhaustive patterns: `West` not covered // 非穷尽匹配，`West`没有被覆盖
  --> src/main.rs:10:11
   |
1  | / enum Direction {
2  | |     East,
3  | |     West,
   | |     ---- not covered
4  | |     North,
5  | |     South,
6  | | }
   | |_- `Direction` defined here
...
10 |       match dire {
   |             ^^^^ pattern `West` not covered // 模式`West`没有被覆盖
   |
   = help: ensure that all possible cases are being handled, possibly by adding wildcards or more match arms
   = note: the matched value is of type `Direction`
```

首先，不禁想感叹，`Rust`的编译器真**强大，忍不住爆粗口了，sorry，如果你以后进一步深入使用Rust也会像我这样感叹的。

其次，Rust知道`match`中没有覆盖的具体分支，甚至知道哪些模式被遗忘了。这种设计初心是为了保证我们处理所有的情况，特别是那种会造成十亿美金的空值问题。

#### `_` 通配符

Rust 也提供了一个**模式**用于不想列举出所有可能值的场景。例如，`u8` 可以拥有 0 到 255 的有效的值，如果我们只关心 1、3、5 和 7 这几个值，就并不想列出其它的 0、2、4、6、8、9 一直到 255 的值。所幸, 我们不必这么做, 因为可以使用使用特殊的模式 `_` 替代：

```rust
let some_u8_value = 0u8;
match some_u8_value {
    1 => println!("one"),
    3 => println!("three"),
    5 => println!("five"),
    7 => println!("seven"),
    _ => (),
}
```

通过将其放置于其他分支后，`_`将会匹配所有遗漏的值。`()`表示啥都不做的意思，所以当匹配到`_`后，什么也不会发生。

然后，在某些场景下，我们其实只关心**某一个值是否存在**，此时`match`就显得过于啰嗦，还好，Rust提供了`if let`.

## `if let`匹配
很多时候都会遇到只有一个模式的值需要被处理，其它值直接忽略的场景，用`match`处理如下：
```rust
    let v = Some(3u8);
    match v{
        Some(3) => println!("three"),
        _ => (),
    }
````

我们想要对 `Some(3)` 模式进行匹配, 同时不想处理任何其他 `Some<u8>` 值或 `None` 值。但是为了满足`match`表达式（穷尽性）的要求，必须在处理完这唯一的成员后加上 `_ => ()`，这样也要增加很多样板代码。

杀鸡焉用牛刀，可以用`if let`的方式来实现：
```rust
if let Some(3) = some_u8_value {
    println!("three");
}
```

这两种匹配对于新手来说，可能有些难以抉择，但是只要记住一点就好: **当你只要匹配一个条件，且忽略其他条件时就用`if let`，否则都用match**.


## 变量覆盖
无论是是`match`还是`if let`，他们都可以在模式匹配时覆盖掉老的值，绑定新的直:
```rust
fn main() {
   let age = Some(30);
   println!("在匹配前，age是{:?}",age);
   if let Some(age) = age {
       println!("匹配出来的age是{}",age);
   }

   println!("在匹配后，age是{:?}",age);
}
```

`cargo run`运行后输出如下：
```console
在匹配前，age是Some(30)
匹配出来的age是30
在匹配后，age是Some(30)
```

可以看出在`if let`中，`=`右边`Some(i32)`类型的`age`被左边`i32`类型的新`age`覆盖了，该覆盖一直持续到`if let`语句块的结束。因此第三个`println!`输出的`age`依然是`Some(i32)`类型。

对于`match`类型也是如此:
```rust
fn main() {
   let age = Some(30);
   println!("在匹配前，age是{:?}",age);
   match age {
       Some(age) =>  println!("匹配出来的age是{}",age),
       _ => ()
   }
   println!("在匹配后，age是{:?}",age);
}
```