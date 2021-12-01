# Rust编程指南

[Rust编程指南](about-book.md)
[进入Rust编程世界](into-rust.md)

## Getting started

- [安装运行](first-try/intro.md)
    - [安装Rust环境(todo)](first-try/installation.md)
    - [墙推VSCode!(todo)](first-try/editor.md)
    - [认识Cargo(todo)](first-try/cargo.md)
    - [你好，世界!(todo)](first-try/hello-world.md)

## Rust学习三部曲

- [基本语法](basic/intro.md)
    - [变量与绑定](basic/variable.md)
    - [基本类型(todo)](basic/type.md)
    - [复合类型(todo)](basic/compound-type.md)
    - [函数与方法(todo)](basic/function-method.md)
    - [格式化输出(todo)](basic/formatted-output.md)
    - [字符串、数组与切片(todo)](basic/string-array-slice.md)
    - [流程控制(todo)](basic/flow-control.md)
    - [返回、异常和错误(todo)](basic/exception-error.md)
    - [模式匹配(todo)](basic/match-pattern.md)
    - [文档注释(todo)](basic/comment.md)
    - [包和模块(todo)](basic/crate-module.md)
    - [语句与表达式(todo)](basic/statement-expression.md)

- [核心语法](core/intro.md)
    - [所有权(todo)](core/ownership.md)
    - [借用(todo)](core/borrowing.md)
    - [生命周期(todo)](core/lifetime.md)
    
- [进阶语法](advance/intro.md)
    - [泛型(todo)](advance/generitic.md)
    - [特征(todo)](advance/trait.md)
    - [迭代器(todo)](advance/interator.md)
    - [集合类型(todo)](advance/collection.md)
    - [函数式编程(todo)](advance/functional-programing.md)
    - [智能指针(todo)](advance/smart-pointer.md)


## 专题内容,每个专题都配套一个小型项目进行实践
- [智能指针](smart-pointer/intro.md)
    - [Box对象(todo)](smart-pointer/box.md)
    - [Deref和Drop特征(todo)](smart-pointer/deref-drop.md)
    - [Rc与RefCell(todo)](smart-pointer/rc-refcell.md)
    - [自引用与内存泄漏(todo)](smart-pointer/self-referrence.md)

- [深入内存](memory/intro.md)
    - [指针和引用(todo)](memory/pointer-ref.md)
    - [未初始化内存(todo)](memory/uninit.md)
    - [内存分配(todo)](memory/allocation.md)
    - [内存布局(todo)](memory/layout.md)
    - [虚拟内存(todo)](memory/virtual.md)

- [对抗编译检查](fight-with-compiler/intro.md)
    - [幽灵数据(todo)](fight-with-compiler/phantom-data.md)
    - [生命周期(todo)](fight-with-compiler/lifetime.md)
    - [类型未限制(todo)](fight-with-compiler/unconstrained.md)

- [多线程](multi-threads/intro.md)
    - [线程管理(todo)](multi-threads/thread.md)
    - [消息传递(todo)](multi-threads/message-passing.md)
    - [数据共享Arc、Mutex、Rwlock(todo)](multi-threads/ref-counter-lock.md)
    - [数据竞争(todo)](multi-threads/races.md)
    - [Send、Sync(todo)](multi-threads/send-sync.md)

- [网络和异步编程](networking/intro.md)
    - [TCP和网络原理(todo)](networking/tcp.md) 
    - [并发与并行(todo)](networking/concurrency-parallelism.md)
    - [异步编程](networking/async/intro.md)
        - [async/await语法](networking/async/async-await.md)
        - [future详解](networking/async/future/into.md)
            - [何为Future](networking/async/future/future.md)
            - [任务调度](networking/async/future/task-schedule.md)
            - [任务执行器](networking/async/future/task-excutor.md)
            - [系统IO](networking/async/future/system-io.md)
            - [执行多个Future](networking/async/future/multi-futures.md)
        - [Pin、Unpin(todo)](networking/async/pin-unpin.md)
        - [遇到不支持的异步特性?](networking/async/future/workarounds.md)
        - [HTTP Client/Server](networking/async/http.md)
        - [定海神针-tokio包](networking/async/tokio/intro.md)
            - [基本用法](networking/async/tokio/basic.md)
            - [异步消息流](networking/async/tokio/stream.md)

- [代码规范](style-guide/intro.md)
    - [命名规范](style-guide/naming.md)
    - [代码风格(todo)](style-guide/code.md)

- [面向对象](object-oriented/intro.md)
    - [OO所需的特性(todo)](object-oriented/characteristics.md)

- [Cargo详解](cargo/intro.md)
    - [常用命令](cargo/commands.md)
    - [项目结构](cargo/layout.md)
    - [Cargo.toml和Cargo.lock](cargo/cargo-toml-lock.md)
    - [构建缓存](cargo/cache.md)
    - [工作空间](cargo/workspace.md)
    - [配置参数(todo)](cargo/manifest.md)

- [性能调优](performance/intro.md)
    - [Benchmark性能测试(todo)](performance/benchmark.md)
    - [减少Runtime check(todo)](performance/runtime-check.md)

- [Unsafe](unsafe/intro.md)
    - [原始指针(todo)](unsafe/raw-pointer.md)

- [宏编程](macro/intro.md)
    - [过程宏(todo)](macro/procedure-macro.md)
   
- [常用三方库](libraries/intro.md)
    - [JSON](libraries/json/intro.md)
        - [serde(todo)](libraries/json/serde.md)
    - [HTTP](libraries/http/intro.md)
        - [reqwest(todo)](libraries/http/reqwest.md)
    - [命令行解析](libraries/command/intro.md)
        - [structopt(todo)](libraries/command/structopt.md)
        
## 场景模版
- [场景模版](templates/intro.md)
    - [文件操作](templates/files/intro.md)
        - [目录(todo)](templates/files/dir.md)
    - [Http请求(todo)](templates/http/intro.md)


## 附录
- [附录](appendix/intro.md)
    - [A-关键字](appendix/a-keywords.md)