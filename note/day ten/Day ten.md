# Day ten

#### Rc 与 Arc

- 通过引用计数的方式，允许一个数据资源在同一时刻拥有多个所有者

- `Rc<T>` 是指向底层数据的不可变的引用
- 使用`drop`去释放，若引用计数尚未清零，数据仍然可以被使用，未被真正释放；最终引用计数归零，底层数据清理释放



**Rc**

- `Rc/Arc` 是不可变引用，你无法修改它指向的值，只能进行读取，如果要修改，需要配合后面章节的内部可变性 `RefCell` 或互斥锁 `Mutex`
- 一旦最后一个拥有者消失，则资源会自动被回收，这个生命周期是在编译期就确定下来的
- `Rc` 只能用于同一线程内部，想要用于线程之间的对象共享，你需要使用 `Arc`
  - `Rc<T>` 不能在线程间安全的传递，实际上是因为它没有实现 `Send` 特征，而该特征是恰恰是多线程间传递数据的关键
  - 更深层的原因：由于 `Rc<T>` 需要管理引用计数，但是该计数器并没有使用任何并发原语，因此无法实现原子化的计数操作，最终会导致计数错误
  - `Arc` 是 `Atomic Rc` 的缩写，原子化的 `Rc<T>` 智能指针。原子化是一种并发原语，能保证我们的数据能够安全的在线程间共享
- `Rc<T>` 是一个智能指针，实现了 `Deref` 特征，因此你无需先解开 `Rc` 指针，再使用里面的 `T`，而是可以直接使用 `T`



- 原子化或者其它锁虽然可以带来的线程安全，但是都会伴随着性能损耗
- 在 Rust 中，所有权机制保证了一个数据只会有一个所有者，但如果你想要在图数据结构、多线程等场景中共享数据，这种机制会成为极大的阻碍。好在 Rust 为我们提供了智能指针 `Rc` 和 `Arc`，使用它们就能实现多个所有者共享一个数据的功能
- `Rc` 和 `Arc` 的区别在于，后者是原子化实现的引用计数，因此是线程安全的，可以用于多线程中共享数据
- 这两者都是只读的，如果想要实现内部数据可修改，必须配合内部可变性 `RefCell` 或者互斥锁 `Mutex` 来一起使用

---

#### Cell 与 RefCell

-  Rust 提供了 `Cell` 和 `RefCell` 用于内部可变性，可以在拥有不可变引用的同时修改目标数据
- 内部可变性的实现是因为 Rust 使用了 `unsafe` 来做到这一点，但是对于使用者来说，这些都是透明的，因为这些不安全代码都被封装到了安全的 API 中
- 由于 `Cell` 类型针对的是实现了 `Copy` 特征的值类型，因此在实际开发中，`Cell` 使用的并不多，因为我们要解决的往往是可变、不可变引用共存导致的问题，此时就需要借助于 `RefCell` 来达成目的



| Rust 规则                            | 智能指针带来的额外规则                  |
| ------------------------------------ | --------------------------------------- |
| 一个数据只有一个所有者               | `Rc/Arc`让一个数据可以拥有多个所有者    |
| 要么多个不可变借用，要么一个可变借用 | `RefCell`实现编译期可变、不可变引用共存 |
| 违背规则导致**编译错误**             | 违背规则导致**运行时`panic`**           |



- `RefCell` 实际上并没有解决可变引用和引用可以共存的问题，只是将报错从编译期推迟到运行时，从编译器错误变成了 `panic` 异常
- `RefCell` **用于你确信代码是正确的，而编译器却发生了误判时**



**RefCell**

- 与 `Cell` 用于可 `Copy` 的值不同，`RefCell` 用于引用
- `RefCell` 只是将借用规则从编译期推迟到程序运行期，并不能帮你绕过这个规则
- `RefCell` 适用于编译期误报或者一个引用被在多处代码使用、修改以至于难于管理借用关系时
- 使用 `RefCell` 时，违背借用规则会导致运行期的 `panic`



- `Cell` 只适用于 `Copy` 类型，用于提供值，而 `RefCell` 用于提供引用
- `Cell` 不会 `panic`，而 `RefCell` 会
- 与 `Cell` 的 `zero cost` 不同，`RefCell` 其实是有一点运行期开销的，原因是它包含了一个字大小的“借用状态”指示器，该指示器在每次运行时借用时都会被修改，进而产生一点开销
- 当非要使用内部可变性时，首选 `Cell`，只有你的类型没有实现 `Copy` 时，才去选择 `RefCell`
- `RefCell` 由于是**非线程安全的**，因此无需保证原子性，性能虽然有一点损耗，但是依然非常好，而 `Cell` 则完全不存在任何额外的性能损耗



简单**MQ**

```rust
use std::cell::RefCell;
pub trait Messenger {
    fn send(&self, msg: String); // 若为外部库 不可变借用
}

pub struct MsgQueue {
    msg_cache: RefCell<Vec<String>>,
    //  RefCell让 &self 中的 msg_cache 成为一个可变值
}

impl Messenger for MsgQueue {
    fn send(&self, msg: String) {
        self.msg_cache.borrow_mut().push(msg)
        // 使用RefCell 实现对其的修改
    }
}

fn main() {
    let mq = MsgQueue {
        msg_cache: RefCell::new(Vec::new()),
    };
    mq.send("hello, world".to_string());
}

```



[Rc + RefCell 组合使用——损耗分析](https://course.rs/advance/smart-pointer/cell-refcell.html#rc--refcell-%E7%BB%84%E5%90%88%E4%BD%BF%E7%94%A8)



- `Cell::from_mut`，该方法将 `&mut T` 转为 `&Cell<T>`
- `Cell::as_slice_of_cells`，该方法将 `&Cell<[T]>` 转为 `&[Cell<T>]`

---

- 函数式特性（闭包和迭代器等）可以让代码的可读性和易写性大幅提升



**闭包 Closure**

- 闭包是一种匿名函数，它可以赋值给变量也可以作为参数传递给其它函数，不同于函数的是，它允许捕获调用者作用域中的值



**缓存**

```rust
struct Cacher<T,E>
where
    T: Fn(E) -> E,
    E: Copy
{
    query: T,
    value: Option<E>,
}

impl<T,E> Cacher<T,E>
where
    T: Fn(E) -> E,
    E: Copy
{
    fn new(query: T) -> Cacher<T,E> {
        Cacher {
            query,
            value: None,
        }
    }

    fn value(&mut self, arg: E) -> E {
        match self.value {
            Some(v) => v,
            None => {
                let v = (self.query)(arg);
                self.value = Some(v);
                v
            }
        }
    }
}
fn main() {
  
}

#[test]
fn call_with_different_values() {
    let mut c = Cacher::new(|a| a);

    let v1 = c.value(1);
    let v2 = c.value(2);

    assert_eq!(v2, 1);
}

```



- 当闭包从环境中捕获一个值时，会分配内存去存储这些值。对于有些场景来说，这种额外的内存分配会成为一种负担。与之相比，函数就不会去捕获这些环境值，因此定义和使用函数不会拥有这种内存负担。
- 在竖线 `|` 之前使用 `move` 会强制闭包取得被捕获变量的所有权



**对比**

```rust
fn main() {
    use std::mem;

    // 不可复制类型（non-copy type）。
    let movable = Box::new(3);

    // `mem::drop` 要求 `T` 类型本身，所以闭包将会捕获变量的值。这种情况下，
    // 可复制类型将会复制给闭包，从而原始值不受影响。不可复制类型必须移动
    // （move）到闭包中，因而 `movable` 变量在这里立即移动到了闭包中。
    let consume = || {
        println!("`movable`: {:?}", movable);
        mem::drop(movable);
    };

    // `consume` 消耗了该变量，所以该闭包只能调用一次。
    consume();
    // consume();
    // ^ 试一试：将此行注释去掉。wrong
}

```



```rust
fn main() {
    use std::mem;

    let movable = Box::new(3);

    let consume = || {
        println!("`movable`: {:?}", movable);
        mem::drop(&movable); 
        // &movable 本质上是个 *const  Box<T>，而指针实现了 Copy 这个 trait（
    };

    consume();
    consume(); 
}
```