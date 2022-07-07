# Day seven

类型转换（**高级部分看不懂(┬┬﹏┬┬)**）

- 没看明白（好吧，这是生命周期章节的）

  ```rust
  impl<'a> Trait for &'a i32 {}
  ```

- 在直觉中，数组都可以通过索引访问，实际上只有数组切片才可以（(⊙_⊙)？

- 通用类型转换，点操作符

- 摘自评论：

  - `a.foo()` 假设`a: T`
  - 首先，foo方法必须有一个`self`参数，才可能被匹配到
  - 尝试使用**值方法调用**，即找self参数类型与a相同的函数foo
    1. 首先看如果a是**引用类型**(&或&mut，其他实现Deref的不算)，则先尝试调用`<*T>::foo(a)`
    2. 尝试调用`T::foo(a)`
  - 尝试使用**引用方法调用**：给a加**一个**引用，比如`&a:&T`，尝试调用：
    1. `<*&T>::foo(&a)`，即`T::foo(&a)`
    2. `<&T>::foo(&a)`，即`<&T>::foo(&a)`
  - 尝试**解引用方法调用**。得到解引用类型，然后回到第1步开始尝试。直到找到匹配解引用类型或无法再解引用。
  - 尝试将定长类型转换为不定长类型来查找方法，如将`[i32;3]`(数组)转为`[i32]`(切片)，查找`[i32]`上的方法。
  - 找不到，报错。

  关键的地方在于第2步的第(1)步，如果a的类型是&T，其实会先去尝试调用T的方法。 而如果a是其他实现了`Deref`的类型T，却不会先去调用`*T`的方法。



-  `unwrap` 成功则返回值，失败则 `panic`，总之不进行任何错误处理。

- panic 原理剖析



-  `?` 宏

  ```rust
  let mut f = match f {
      // 打开文件成功，将file句柄赋值给f
      Ok(file) => file,
      // 打开文件失败，将错误返回(向上传播)
      Err(e) => return Err(e),
  };
  ```

  - `?` 不仅仅可以用于 `Result` 的传播，还能用于 `Option` 的传播
  - `Result` 通过 `?` 返回错误，那么 `Option` 就通过 `?` 返回 `None`：
  - `?` 操作符需要一个变量来承载正确的值，这个函数只会返回 `Some(&i32)` 或者 `None`，只有错误值能直接返回，正确的值不行，`?` 只能用于以下形式：
    - `let v = xxx()?;`
    - `xxx()?.yyy()?;`

- `Box<dyn Error>` 特征对象； `std::error:Error` 是 Rust 中抽象层次最高的错误，其它标准库中的错误都实现了该特征，因此我们可以用该特征对象代表一切错误，就算 `main` 函数中调用任何标准库函数发生错误，都可以通过 `Box<dyn Error>` 这个特征对象进行返回



- 模块可见性不代表模块内部项的可见性，模块的可见性仅仅是允许其它模块去引用它，但是想要引用它内部的项，还得继续将对应的项标记为 `pub`

- 将结构体设置为 `pub`，但它的所有字段依然是私有的

- 将枚举设置为 `pub`，它的所有字段也将对外可见

- 让某一项可以在整个包中都可以被使用

  - 在包根中定义一个非 `pub` 类型的 `X`(父模块的项对子模块都是可见的，因此包根中的项对模块树上的所有模块都可见)
  - 在子模块中定义一个 `pub` 类型的 `Y`，同时通过 `use` 将其引入到包根

- 限制可见性语法

  `pub(crate)` 或 `pub(in crate::a)` 就是限制可见性语法，前者是限制在整个包内可见，后者是通过绝对路径，限制在包内的某个模块内可见，总结一下：

  - `pub` 意味着可见性无任何限制
  - `pub(crate)` 表示在当前包可见
  - `pub(self)` 在当前模块可见
  - `pub(super)` 在父模块可见
  - `pub(in <path>)` 表示在某个路径代表的模块中可见，其中 `path` 必须是父模块或者祖先模块

- `lib.rs`中通过 `#[path ="你的路径"]` 可以将`mod`放在任何目录



摘录：

```rust
// 一个名为 `my_mod` 的模块
mod my_mod {
    // 模块中的项默认具有私有的可见性
    fn private_function() {
        println!("called `my_mod::private_function()`");
    }

    // 使用 `pub` 修饰语来改变默认可见性。
    pub fn function() {
        println!("called `my_mod::function()`");
    }

    // 在同一模块中，项可以访问其它项，即使它是私有的。
    pub fn indirect_access() {
        print!("called `my_mod::indirect_access()`, that\n> ");
        private_function();
    }

    // 模块也可以嵌套
    pub mod nested {
        pub fn function() {
            println!("called `my_mod::nested::function()`");
        }

        #[allow(dead_code)]
        fn private_function() {
            println!("called `my_mod::nested::private_function()`");
        }

        // 使用 `pub(in path)` 语法定义的函数只在给定的路径中可见。
        // `path` 必须是父模块（parent module）或祖先模块（ancestor module）
        pub(in crate::my_mod) fn public_function_in_my_mod() {
            print!("called `my_mod::nested::public_function_in_my_mod()`, that\n > ");
            public_function_in_nested()
        }

        // 使用 `pub(self)` 语法定义的函数则只在当前模块中可见。
        pub(self) fn public_function_in_nested() {
            println!("called `my_mod::nested::public_function_in_nested");
        }

        // 使用 `pub(super)` 语法定义的函数只在父模块中可见。
        pub(super) fn public_function_in_super_mod() {
            println!("called my_mod::nested::public_function_in_super_mod");
        }
    }

    pub fn call_public_function_in_my_mod() {
        print!("called `my_mod::call_public_funcion_in_my_mod()`, that\n> ");
        nested::public_function_in_my_mod();
        print!("> ");
        nested::public_function_in_super_mod();
    }

    // `pub(crate)` 使得函数只在当前包中可见
    pub(crate) fn public_function_in_crate() {
        println!("called `my_mod::public_function_in_crate()");
    }

    // 嵌套模块的可见性遵循相同的规则
    mod private_nested {
        #[allow(dead_code)]
        pub fn function() {
            println!("called `my_mod::private_nested::function()`");
        }
    }
}

fn function() {
    println!("called `function()`");
}

fn main() {
    // 模块机制消除了相同名字的项之间的歧义。
    function();
    my_mod::function();

    // 公有项，包括嵌套模块内的，都可以在父模块外部访问。
    my_mod::indirect_access();
    my_mod::nested::function();
    my_mod::call_public_function_in_my_mod();

    // pub(crate) 项可以在同一个 crate 中的任何地方访问
    my_mod::public_function_in_crate();

    // pub(in path) 项只能在指定的模块中访问
    // 报错！函数 `public_function_in_my_mod` 是私有的
    //my_mod::nested::public_function_in_my_mod();
    // 试一试 ^ 取消该行的注释

    // 模块的私有项不能直接访问，即便它是嵌套在公有模块内部的

    // 报错！`private_function` 是私有的
    //my_mod::private_function();
    // 试一试 ^ 取消此行注释

    // 报错！`private_function` 是私有的
    //my_mod::nested::private_function();
    // 试一试 ^ 取消此行的注释

    // 报错！ `private_nested` 是私有的
    //my_mod::private_nested::function();
    // 试一试 ^ 取消此行的注释
}

```





```bash
cargo doc --open
```



- 格式化输出-具名参数：带名称的参数必须放在不带名称参数的后面

- ```rust
  // 使用x作为占位符输出内容，同时使用5作为宽度
  println!("Hello {1:0$}!", 5, "x");
  
  // println!("Hello {1:0$}!", "x", 5); // wrong 1 可能是指定的宽度 usize --> 其实是位置参数
  
  // {:.*}接收两个参数，第一个是精度，第二个是被格式化的值 => Hello abc!
  println!("Hello {:.*}!", 3, "abcdefg");
  
  // 这里的2好像没有什么意义
  println!("{:2e}", 1000000000); 
  println!("{:2E}", 1000000000); 
  
  // {使用{转义，}使用} => Hello {}
  println!("Hello {{}}");
  println!("Hello {\{\}}"); // wrong
  ```

- 
