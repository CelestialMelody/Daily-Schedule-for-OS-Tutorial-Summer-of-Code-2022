# Day thirteen

用条件变量(Condvar)控制线程的同步

- 解决资源访问顺序的问题
- [`notify_one()`](https://rustwiki.org/zh-CN/std/sync/struct.Condvar.html#method.notify_one)：在此 condvar 上唤醒一个阻塞的线程，如果此条件变量上有阻塞的线程，则它将从其调用中唤醒到 [`wait`](https://rustwiki.org/zh-CN/std/sync/struct.Condvar.html#method.wait) 或 [`wait_timeout`](https://rustwiki.org/zh-CN/std/sync/struct.Condvar.html#method.wait_timeout)
- [`wait()`](https://rustwiki.org/zh-CN/std/sync/struct.Condvar.html#method.wait)：将自动解锁指定的互斥锁 (由 `guard` 表示) 并阻塞当前线程，在互斥锁解锁后逻辑上发生的任何对 [`notify_one`](https://rustwiki.org/zh-CN/std/sync/struct.Condvar.html#method.notify_one) 或 [`notify_all`](https://rustwiki.org/zh-CN/std/sync/struct.Condvar.html#method.notify_all) 的调用都可以唤醒该线程。 当此函数调用返回时，将重新获得指定的锁



#### [线程阻塞](https://www.cnblogs.com/JonaLin/p/11570858.html#circle=on)

>#### 什么是线程阻塞？
>
>在某一时刻某一个线程A在运行一段代码的时候，这时候另一个线程B也需要运行，但是在运行过程中的那个线程A执行完成之前，另一个线程B是无法获取到CPU执行权的，这个时候就会造成线程阻塞
>
>调用sleep方法是进入到睡眠暂停状态，但是CPU执行权并没有交出去，而调用wait方法则是将CPU执行权交给另一个线程



>#### 为什么会出现线程阻塞？
>
>**睡眠状态** sleep
>
>当一个线程A执行代码的时候调用了sleep方法后，线程A处于睡眠状态，需要设置一个睡眠时间，此时有其他线程B需要执行时就会造成线程阻塞，而且sleep方法被调用之后，线程A不会释放锁对象，但是锁还在线程A手里，等睡眠一段时间后，线程A就会进入就绪状态
>
>**等待状态 **wait
>
>当一个线程B正在运行时，调用了wait方法，此时该线程B需要交出CPU执行权，也就是将锁释放出去，交给另一个线程A，该线程B进入等待状态；
>
>但与睡眠状态不一样的是，进入等待状态的线程B不需要设置睡眠时间，但是需要其他线程A执行notify方法或者notifyall方法来对B唤醒，B自己是不会主动醒来的，等被唤醒之后，该线程B也会进入就绪状态，但是进入就绪状态的该线程B手里是没有执行权的，也就是没有锁，而睡眠状态的线程A一旦苏醒，进入就绪状态时是自己还拿着锁的
>
>**自闭状态** join
>
>当一个线程A正在运行时，调用了一个join方法，此时该线程A会进入阻塞状态，另一个线程B会运行，直到B运行结束后，原线程A才会进入就绪状态



> 一篇java文章：[线程阻塞和进程阻塞、挂起、sleep,wait()和notify()方法](https://blog.csdn.net/qq_22162093/article/details/108873592)



信号量 Semaphore

- 在多线程中，另一个重要的概念就是信号量，使用它可以让我们精准的控制当前正在运行的任务最大数量
- [`tokio::sync::Semaphore`](https://github.com/tokio-rs/tokio/blob/master/tokio/src/sync/semaphore.rs)

- 关键在于信号量的申请和归还：使用前需要申请信号量，如果容量满了，就需要等待；使用后需要释放信号量，以便其它等待者可以继续



线程同步：Atomic 原子类型与内存顺序

- `Mutex`用起来简单，但是无法并发读，`RwLock`可以并发读，但是使用场景较为受限且性能不够
- 原子指的是一系列不可被 CPU 上下文交换的机器指令，这些指令组合在一起就形成了原子操作
- 在多核 CPU 下，当某个 CPU 核心开始运行原子操作时，会先暂停其它 CPU 内核对内存的操作，以保证原子操作不会被其它 CPU 内核所干扰
- 由于原子操作是通过指令提供的支持，因此它的性能相比锁和消息传递会好很多。相比较于锁而言，原子类型不需要开发者处理加锁和释放锁的问题，同时支持修改，读取等操作，还具备较高的并发性能
- 原子类型是无锁类型，但是无锁不代表无需等待，因为原子类型内部使用了`CAS`循环，当大量的冲突发生时仍需要等待，但是总归比锁要好
- `fetch_add`

-  **和`Mutex`一样，`Atomic`的值具有内部可变性**，你无需将其声明为`mut`



内存顺序

- 内存顺序是指 CPU 在访问内存时的顺序，该顺序可能受以下因素的影响：
  - 代码中的先后顺序
  - 编译器优化导致在编译阶段发生改变(内存重排序 reordering)
  - 运行阶段因 CPU 的缓存机制导致顺序被打乱

- [限定内存顺序的 5 个规则](https://course.rs/advance/concurrency-with-threads/sync2.html#限定内存顺序的-5-个规则)



那么原子类型既然这么全能，它可以替代锁吗？答案是不行：

- 对于复杂的场景下，锁的使用简单粗暴，不容易有坑
- `std::sync::atomic`包中仅提供了数值类型的原子操作：`AtomicBool`, `AtomicIsize`, `AtomicUsize`, `AtomicI8`, `AtomicU16`等，而锁可以应用于各种类型
- 在有些情况下，必须使用锁来配合，例如上一章节中使用`Mutex`配合`Condvar`



`Send`和`Sync`是在线程间安全使用一个值的关键

- 实际上它们只是标记特征(marker trait，该特征未定义任何行为，因此非常适合用于标记)

- 实现`Send`的类型可以在线程间安全的传递其所有权
- 实现`Sync`的类型可以在线程间安全的共享(通过引用)
- 潜在的依赖：一个类型要在线程间安全的共享的前提是，指向它的引用必须能在线程间传递。因为如果引用都不能被传递，我们就无法在多个线程间使用引用去访问同一个数据了
- **若类型 T 的引用`&T`是`Send`，则`T`是`Sync`**



```rust
// Rc
impl<T: ?Sized> !marker::Send for Rc<T> {}
impl<T: ?Sized> !marker::Sync for Rc<T> {}

// Arc
unsafe impl<T: ?Sized + Sync + Send> Send for Arc<T> {}
unsafe impl<T: ?Sized + Sync + Send> Sync for Arc<T> {}

// RwLock
unsafe impl<T: ?Sized + Send + Sync> Sync for RwLock<T> {}

// Mutex
unsafe impl<T: ?Sized + Send> Sync for Mutex<T> {}
```



在 Rust 中，几乎所有类型都默认实现了`Send`和`Sync`，而且由于这两个特征都是可自动派生的特征(通过`derive`派生)，意味着一个复合类型(例如结构体), 只要它内部的所有成员都实现了`Send`或者`Sync`，那么它就自动实现了`Send`或`Sync`。

Rust 中绝大多数类型都实现了`Send`和`Sync`，除了以下几个(比较常见的):

- 裸指针两者都没实现，因为它本身就没有任何安全保证
- `UnsafeCell`不是`Sync`，因此`Cell`和`RefCell`也不是
- `Rc`两者都没实现(因为内部的引用计数器不是线程安全的)

如果是自定义的复合类型：只要复合类型中有一个成员不是`Send`或`Sync`，那么该复合类型也就不是`Send`或`Sync`。



1. 实现`Send`的类型可以在线程间安全的传递其所有权, 实现`Sync`的类型可以在线程间安全的共享(通过引用)
2. 绝大部分类型都实现了`Send`和`Sync`，常见的未实现的有：裸指针、`Cell`、`RefCell`、`Rc` 等
3. 可以为自定义类型实现`Send`和`Sync`，但是需要`unsafe`代码块
4. 可以为部分 Rust 中的类型实现`Send`、`Sync`，但是需要使用`newtype`



---

#### Macro宏

> ps：不懂，，，

宏和函数的区别

- 元编程
- 可变参数
- 宏展开



在 Rust 中宏分为两大类：**声明式宏( declarative macros )** `macro_rules!` 和三种**过程宏( procedural macros )**:

- `#[derive]`，在之前多次见到的派生宏，可以为目标结构体或枚举派生指定的代码，例如 `Debug` 特征
- 类属性宏(Attribute-like macro)，用于为目标添加自定义的属性
- 类函数宏(Function-like macro)，看上去就像是函数调用



声明式宏

- 示例宏( macros by example )、`macro_rules!` 或干脆直接称呼为**宏**
- 宏是通过 `macro_rules!` 宏来创建的

- **宏是将一个值跟对应的模式进行匹配，且该模式会与特定的代码相关联**。但是与 `match` 不同的是，**宏里的值是一段 Rust 源代码**(字面量)，模式用于跟这段源代码的结构相比较，一旦匹配，传入宏的那段源代码将被模式关联的代码所替换，最终实现宏展开。值得注意的是，**所有的这些都是在编译期发生，并没有运行期的性能损耗**
- 语法
  - 宏的参数使用一个美元符号 `$` 作为前缀，并使用一个**指示符**（designator）来注明类型
    - `block`、`expr` 用于表达式、`ident` 用于变量名或函数名、`item`、`pat` (**模式** *pattern*)、`path`、`stmt` (**语句** *statement*)、`tt` (**标记树** *token tree*)、`ty` (**类型** *type*)
  -  `stringify!` 宏把 `ident` 转换成字符串
  - 宏可以重载，从而接受不同的参数组合。在这方面，`macro_rules!` 的作用类似于匹配（match）代码块
  - 宏在参数列表中可以使用 `+` 来表示一个参数可能出现一次或多次，使用 `*` 来表示该 参数可能出现零次或多次。
    - 把模式这样： `$(...),+` 包围起来，就可以匹配一个或多个用逗号隔开的表达式
  - 每个分支都必须以分号结束；宏定义的最后一个分支可以不用分号作为结束



过程宏

- 过程宏是使用源代码作为输入参数，基于代码进行一系列操作后，再输出一段全新的代码

- 过程宏中的 derive 宏输出的代码并不会替换之前的代码，这一点与声明宏有很大的不同

- 目前只能在单独的包中定义过程宏



