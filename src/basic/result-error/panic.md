# 不可恢复错误panic!

在正式开始之前，先来思考一个问题: 假设我们想要从文件读取数据，如果失败，你有没有好的办法通知调用者为何失败？如果成功，你有没有好的办法把读取的结果返还给调用者？

## panic!与不可恢复错误
上面的问题在真实场景，其实挺复杂的，让我们先做一个假设：文件读取操作发生在系统启动阶段。那么可以轻易得出一个结论,一旦文件读取失败，那么系统启动也将失败，这意味着该失败是不可恢复的错误，无论是因为文件不存在还是操作系统硬盘的问题，这些只是错误的原因不同，但是归根到底都是不可恢复的错误(梳理清楚当前场景的错误类型非常重要).

既然是不可恢复错误，那么一旦发生，只需让程序崩溃即可。对此，Rust为我们提供了`panic!`宏，当调用执行该宏时，**程序会打印出一个错误信息，展开报错点往前的函数调用堆栈，最后退出程序**. 

切记，一定是不可恢复的错误，才调用`panic!`处理，你总不想系统仅仅因为用户随便传入一个非法参数就崩溃吧？所以，**只有当你不知道该如何处理时，再去调用panic!**.

## 调用panic!
首先，来调用一下`panic!`，这里使用了最简单的代码实现，实际上你在程序的任何地方都可以这样调用：
```rust
fn main() {
    panic!("crash and burn");
}
```

运行后输出:
```console
thread 'main' panicked at 'crash!!1', src/main.rs:3:5
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

以上信息包含了两条重要信息: 
- `main`函数所在的线程崩溃了，发生的代码位置是`src/main.rs`中的第3行第5个字符(去除该行前面的空字符)
- 在使用时加上一个环境变量可以获取更详细的栈展开信息: `RUST_BACKTRACE=1 cargo run`

下面让我们针对第二点进行详细展开讲解。

## backtrace栈展开
在真实场景中，错误往往涉及到很长的调用链甚至会深入第三方库，如果没有栈展开技术，错误将难以跟踪处理，下面我们来看一个真实的崩溃例子：
```rust
fn main() {
    let v = vec![1, 2, 3];

    v[99];
}
```
上面的代码很简单，数组只有`3`个元素，我们却尝试去访问它的第`100`号元素(数组索引从`0`开始)，那自然会崩溃。

我们的读者里不乏正义之士，此时肯定要质疑，一个简单的数组越界访问，为何要直接让程序崩溃？是不是有些大题小做了？

如果有过C语言的经验，即使你越界了，问题不大，我依然尝试去访问，至于这个值是不是你想要的(`100`号内存地址也有可能有值，只不过是其它变量或者程序的！)，抱歉，不归我管,我只负责取，你要负责管理好自己的索引访问范围。上面这种情况被称为**缓冲区溢出**，并可能会导致安全漏洞，例如攻击者可以通过索引来访问到数组后面不被允许的数据。

说实话，我宁愿程序崩溃，为什么？当你取到了一个不属于你的值，这在很多时候会导致程序上的逻辑bug! 有编程经验的人都知道这种逻辑上的bug是多么难发现和修复！因此程序直接崩溃，然后告诉我们问题发生的位置，最后我们对此进行修复，这才是最合理的软件开发流程，而不是把问题藏着掖着:
```console
thread 'main' panicked at 'index out of bounds: the len is 3 but the index is 99', src/main.rs:4:5
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

好的，现在成功知道问题发生的位置，但是如果我们想知道该问题之前经过了哪些调用环节，该怎么办？那就按照提示使用`RUST_BACKTRACE=1 cargo run`来再一次运行程序:
```console
thread 'main' panicked at 'index out of bounds: the len is 3 but the index is 99', src/main.rs:4:5
stack backtrace:
   0: rust_begin_unwind
             at /rustc/59eed8a2aac0230a8b53e89d4e99d55912ba6b35/library/std/src/panicking.rs:517:5
   1: core::panicking::panic_fmt
             at /rustc/59eed8a2aac0230a8b53e89d4e99d55912ba6b35/library/core/src/panicking.rs:101:14
   2: core::panicking::panic_bounds_check
             at /rustc/59eed8a2aac0230a8b53e89d4e99d55912ba6b35/library/core/src/panicking.rs:77:5
   3: <usize as core::slice::index::SliceIndex<[T]>>::index
             at /rustc/59eed8a2aac0230a8b53e89d4e99d55912ba6b35/library/core/src/slice/index.rs:184:10
   4: core::slice::index::<impl core::ops::index::Index<I> for [T]>::index
             at /rustc/59eed8a2aac0230a8b53e89d4e99d55912ba6b35/library/core/src/slice/index.rs:15:9
   5: <alloc::vec::Vec<T,A> as core::ops::index::Index<I>>::index
             at /rustc/59eed8a2aac0230a8b53e89d4e99d55912ba6b35/library/alloc/src/vec/mod.rs:2465:9
   6: world_hello::main
             at ./src/main.rs:4:5
   7: core::ops::function::FnOnce::call_once
             at /rustc/59eed8a2aac0230a8b53e89d4e99d55912ba6b35/library/core/src/ops/function.rs:227:5
note: Some details are omitted, run with `RUST_BACKTRACE=full` for a verbose backtrace.
```

上面的代码就是一次栈展开(也称栈回溯)，它包含了函数调用的顺序，当然按照逆序排列：最近调用的函数排在列表的最上方. 因为咱们的`main`函数基本是最先调用的函数了，所以排在了倒数第二位，还有一个关注点，排在最顶部最后一个调用的函数是`rust_begin_unwind`，该函数的目的就是进行栈展开，呈现这些列表信息给我们。

要获取到栈回溯信息，你还需要开启`debug`标志，该标志在使用`cargo run`或者`cargo build`时自动开启(这两个方式是`Debug`运行方式). 同时，栈展开信息在不同操作系统或者Rust版本上也所有不同。

## panic时的两种终止方式
当出现`panic!`时，程序提供了两种方式来处理终止流程: **栈展开** 和 **直接终止**.

其中，默认的方式就是`栈展开`，这意味着Rust会回溯栈上数据和函数调用，因此也意味着更多的善后工作，好处是给于充分的报错信息和栈调用信息，便于事后的问题复盘。`直接终止`，顾名思义，不清理数据就直接推出程序，善后工作交与操作系统来负责。

对于绝大多数用户，使用默认选择是最好的，但是当你关心最终编译出的二进制可执行文件大小时，那么可以尝试去使用直接终止的方式,例如下面的配置修改`Cargo.toml`文件，实现在[`release`](../first-try/cargo.md#手动编译和运行项目)模式下遇到`panic`直接终止：
```rust
[profile.release]
panic = 'abort'
```

## 何时该使用panic!

下面让我们大概罗列下合适适合使用`panic`，虽然原则上，你理解了之前的内容后，会自己作出合适的选择，但是罗列出来可以帮助你强化这一点。

先来一点背景知识，在前面章节我们粗略讲过`Result<T,E>`这个枚举类型，它是用来表示函数的返回结果：
```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```
当没有错误发生时，函数返回一个用`Result`类型包裹的值`Ok(T)`，当错误时，返回一个`Err(E)`。对于`Result`返回我们有很多处理方法，最简单粗暴的就是`unwrap`和`expect`，这两个函数非常类似，我们以`unwrap`举例:
```rust
use std::net::IpAddr;
let home: IpAddr = "127.0.0.1".parse().unwrap();
```

上面的`parse`方法试图将字符串`"127.0.0.1"`解析为一个IP地址类型`IpAddr`，它返回一个`Result<IpAddr,E>`类型，如果解析成功，则把`Ok(IpAddr)`中的值赋给`home`,如果失败，则不处理`Err(E)`，而是直接`panic`。

因此`unwrap`简而言之：成功则返回值，失败则`panic`, 总之不进行任何错误处理。

#### 示例、原型、测试
这些场景，需要快速的搭建代码，错误处理反而会拖慢实现速度，也不是特别有必要，因此通过`unrap`、`expect`等方法来处理是最快的。

同时，当我们准备做错误处理时，全局搜索这些方法，也可以不遗漏的进行替换。

#### 你确切的知道你的程序是正确时，可以使用panic
因为`panic`的触发方式比错误处理要简单，因此可以让代码更清晰，可读性也更加好，当我们的代码注定是正确时，你可以用`unrawp`等方法直接进行处理，反正也不可能`panic`：
```rust
use std::net::IpAddr;
let home: IpAddr = "127.0.0.1".parse().unwrap();
```

例如上面的例子，`"127.0.0.1"`就是`ip`地址，因此我们知道`parse`方法一定会成功，那么就可以直接用`unwrap`方法进行处理。

当然，如果该字符串是来自于用户输入，那在实际项目中，就必须用错误处理的方式，而不是`unwrap`，否则你的程序一天要崩溃几十万次吧！

#### 可能导致全局有害状态时
有害状态大概分为几类：
- 非预期的错误
- 之后代码的运行会受到明显可见的影响
- 内存安全的问题

当错误预期会出现时，返回一个错误较为合适，例如解析器接收到格式错误的数据，HTTP请求接收到错误的参数甚至该请求内的任何错误(不会导致整个程序有问题，只影响该此请求)。 **因为错误是可预期的，因此也是可以处理的**。

当启动时某个流程发生了错误，导致了后续代码的允许造成影响，那么就应该使用`panic`，而不是处理错误后，继续运行，当然你可以通过重试的方式来继续。

上面提到过，数组访问越界，就要`panic`的原因，这个就是属于内存安全的范畴，一旦内存访问不安全，那么我们无法保证自己的程序会怎么运行下去，也无法保证逻辑和数据的正确性。


## panic原理剖析

Not sure exactly what you're asking, but I'll try to describe the panic mechanism as I understand it. I'm sure folks will correct my mistakes.
When you call the panic!() macro it formats the panic message and then calls std::panic::panic_any() with the message as the argument. panic_any() first checks to see if there's a "panic hook" installed by the application: if so, the hook function is called. Assuming that the hook function returns, the unwinding of the current thread begins with the parent stack frame of the caller of panic_any(). If the registers or the stack are messed up, likely trying to start unwinding will cause an exception of some kind, at which point the thread will be aborted instead of unwinding.
Unwinding is a process of walking back up the stack, frame-by-frame. At each frame, any data owned by the frame is dropped. (I believe things are dropped in reverse static order, as they would be at the end of a function.)
One exceptional case during unwinding is that the unwinding may hit a frame marked as "catching" the unwind via std::panic::catch_unwind(). If so, the supplied catch function is called and unwinding ceases: the catching frame may continue the unwinding with std::panic::resume_unwind() or not.
Another exceptional case during unwinding is that some drop may itself panic. In this case the unwinding thread is aborted.
Once unwinding of a thread is aborted or completed (no more frames to unwind), the outcome depends on which thread panicked. For the main thread, the operating environment's abort functionality is invoked to terminate the panicking process via core::intrinsics::abort(). Child threads, on the other hand, are simply terminated and can be collected later with std::thread::join()