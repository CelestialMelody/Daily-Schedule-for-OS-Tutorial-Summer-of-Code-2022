# Day fourteen

>一篇文章：[深入理解操作系统之——分页式存储管理](https://zhuanlan.zhihu.com/p/37549063)



应用程序与基本执行环境

- [理解应用程序执行环境](https://learningos.github.io/rust-based-os-comp2022/chapter1/1app-ee-platform.html#id3)

- 目标三元组 (Target Triplet) 

  - 目标平台的 CPU 指令集、操作系统类型和标准运行时库

  - ```shell
    rustc --version --verbose
    ```

- rust 标准库 std 与 核心库 core

-  `#![no_std]`不使用 Rust 标准库 std 

- `start` 的语义项与`#![no_main]`

- `extern "C" fn _start() `

- RustSBI

- 链接脚本 (Linker Script) 

- 初始化栈空间

- >[bss段、data段、text段、堆(heap)和栈(stack)](https://www.cnblogs.com/yanghong-hnu/p/4705755.html) 

- 清空 .bss 段



---

**生命周期**

- 生命周期标注并不会改变任何引用的实际作用域
- 在通过函数签名指定生命周期参数时，我们并没有改变传入引用或者返回引用的真实生命周期，而是告诉编译器当不满足此约束条件时，就拒绝编译通过
- 函数的返回值如果是一个引用类型，那么它的生命周期只会来源于：
  - 函数参数的生命周期
  - 函数体中某个新建引用的生命周期（悬垂引用，改返回所有权）
- 结构体中使用引用：为结构体中的**每一个引用标注上生命周期**；结构体中的引用的生命周期要大于等于结构体生命周期
- 生命周期消除：返回值的引用是获取自参数，这就意味着参数和返回值的生命周期是一样的，可以不指明生命周期
  - 消除规则不是万能的，若编译器不能确定某件事是正确时，会直接判为不正确，那么你还是需要手动标注生命周期
  - 函数或者方法中，参数的生命周期被称为 `输入生命周期`，返回值的生命周期被称为 `输出生命周期`
- 三条消除规则——**函数**（闭包不行）
  - 每一个引用参数都会获得独自的生命周期
  - 若只有`一个输入生命周期`(函数参数中只有一个引用类型)，那么该生命周期会被赋给所有的`输出生命周期`，也就是所有返回值的生命周期都等于该`输入生命周期`
  - 若存在多个`输入生命周期`，且其中一个是 `&self` 或 `&mut self`，则 `&self` 的生命周期被赋给`所有的输出生命周期`
    - 拥有 `&self` 形式的参数，说明该函数是一个 `方法`，该规则让方法的使用便利度大幅提升
- `'a: 'b`，是生命周期约束语法，跟泛型约束非常相似，用于说明 `'a` 必须比 `'b` 活得久
- 可以把 `'a` 和 `'b` 都在同一个地方声明（如上），或者分开声明但通过 `where 'a: 'b` 约束生命周期关系
- `'static`生命周期的引用可以和整个程序活得一样久



生命周期导致编译失败案例

```rust
#[derive(Debug)]
struct Foo;

impl Foo {
    fn mutate_and_share(&mut se`lf) -> &Self { // 可变借用与返回值生命周期一致
       &*self //  再借用
    }

    fn share(&self) {}
}

fn main() {
    let mut foo = Foo;
    let loan = foo.mutate_and_share(); // 可变借用与返回值生命周期一致
    foo.share();
    println!("{:?}", loan); // 可变借用生命周期仍然存在
}
```



无界生命周期

生命周期约束 HRTB

- `'a: 'b`：`'a` >= `'b`，表示 `'a` 至少要活得跟 `'b` 一样久
- `T: 'a`：表示类型 `T` 必须比 `'a` 活得要久（目前编译器可以自动推导 `T: 'a` 类型的约束）



Reborrow 再借用

```rust
#[derive(Debug)]
struct Point {
    x: i32,
    y: i32,
}

impl Point {
    fn move_to(&mut self, x: i32, y: i32) {
        self.x = x;
        self.y = y;
    }
}

fn main() {
    let mut p = Point { x: 0, y: 0 };
    let r = &mut p;
    // reborrow! 此时对`r`的再借用不会导致跟上面的借用冲突
    let rr: &Point = &*r;

    // 再借用`rr`最后一次使用发生在这里，在它的生命周期中，我们并没有使用原来的借用`r`，因此不会报错
    println!("{:?}", rr);

    // 再借用结束后，才去使用原来的借用`r`
    r.move_to(10, 10);
    println!("{:?}", r);
}
```

