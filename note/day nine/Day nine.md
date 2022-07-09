# Day nine

- ```rust
  trait AppendBar {
      fn append_bar(self) -> Self;
  }
  
  impl AppendBar for Vec<String> {
      fn append_bar(mut self) -> Self {
          self.push("Bar".to_string());
          self
      }
  }    
  ```

  解答：为什么在traits中的签名为`self`，但实现是`mut self`也可以？

  无论是`self` 还是 `mut self` 所有权都发生了转移，`mut`用于修饰可变，并不影响（可以理解为已经拥有所有权，所有可以决定是否为`mut`）

  但是，`&mut self` 与 `&self`不一样；`&mut self`独占引用，签名与`&self`不同

- `self`是 `self: Self`的简写，`&mut self` 是 `self: &mut Self`

  ```rust
  fn display(&mut self) {}
  
  fn display(self: &mut Person) {} // 与上面相同
  ```

  

- 函数和方法中的隐式 Deref 转换

  - 若一个类型实现了 `Deref` 特征，那它的引用在传给函数或方法时，会根据参数签名来决定是否进行隐式的 `Deref` 转换
  - **仅引用类型的实参才会触发自动解引用**

  ```rust
  struct MyBox<T>(T);
  
  fn main() {
      let s = MyBox::new(String::from("hello, world"));
      let s1: &str = &s;
      let s2: String = s.to_string();
  }
  ```

  - 对于 `s1`，通过两次 `Deref` 将 `&str` 类型的值赋给了它（**赋值操作需要手动解引用**）；
  - 对于 `s2`，我们在其上直接调用方法 `to_string`，实际上 `MyBox` 根本没有没有实现该方法，能调用 `to_string`，完全是因为编译器对 `MyBox` 应用了 `Deref` 的结果（**方法调用会自动解引用**）。

- Rust 会在解引用时自动把智能指针和 `&&&&v` 做引用归一化操作，转换成 `&v` 形式，最终再对 `&v` 进行解引用：

  - 把智能指针（比如在库中定义的，Box、Rc、Arc、Cow 等）从结构体脱壳为内部的引用类型，也就是转成结构体内部的 `&v`
  - 把多重`&`，例如 `&&&&&&&v`，归一成`&v`



- 实现`Deref`可以从智能指针获取相应的类型；如从`Box<String>`中，使用`*`解引用获取`String`类型的数据；

- 对于智能指针：把智能指针（比如在库中定义的，Box、Rc、Arc、Cow 等）从结构体脱壳为内部的引用类型，也就是转成结构体内部的 `&v`

  - 效果上类似于从`Box<String>`的引用（防止所有权转移）中，拿到`&String`（`String`的引用)

  ```rust
  struct MyBox<T>(T);
  
  impl<T> MyBox<T> {
      fn new(x: T) -> MyBox<T> {
          MyBox(x)
      }
  }
  
  use std::ops::Deref;
  
  impl<T> Deref for MyBox<T> {
      type Target = T; // 关联类型，提高可读性，作用类似于泛型
  
      fn deref(&self) -> &Self::Target {
      	// &self -> self: &Self -> &Box<T> 取自身的引用
          &self.0
          // self.0 -> T
          // &self.0 -> &T -> &Self:: Target
      }
  }
  ```

- 对于多重引用：把多重`&`，例如 `&&&&&&&v`，归一成 `&v`

  ```rust
  impl<T: ?Sized> Deref for &T {
      type Target = T;
  
      fn deref(&self) -> &T {
          // &self -> self: &Self -> self: &&T
          *self
          // *self -> *(&&T) -> &T
      }
  }
  ```

- 实现 `Deref` trait 允许我们重载 **解引用运算符**（*dereference operator*）`*`

- 问题：解引用，再赋值，所有权会转移吗？

  - 这里实现的`Deref`返回的都是引用，实际上不会转移所有权；不过需要注意都是引用

    ```rust
    #[derive(Debug)]
    struct MyBox<T>(T);
    
    impl<T> MyBox<T> {
        fn new(x: T) -> MyBox<T> {
            MyBox(x)
        }
    }
    
    use std::ops::Deref;
    
    impl<T> Deref for MyBox<T> {
        type Target = T;
    
        fn deref(&self) -> &Self::Target {
            &self.0
        }
    }
    
    fn main() {
        let mut s = MyBox::new(String::from("hello, world"));
        let s1: &str = &&&&&s; // &self
        let s2: String = s.to_string(); //deref to &str, &str use to_string()
    
        println!("{:?}", s);
        println!("{}", s1);
        println!("{}", s2);
    
        s.0.push_str(", Rust!"); // &mut self
    
        // println!("{:?}", s); // &self
    
        println!("{}", s1);
        println!("{}", s2);
    }
    
    ```

    

- 要实现 `DerefMut` 必须要先实现 `Deref` 特征：`pub trait DerefMut: Deref`
- `T: DerefMut<Target=U>` 解读：将 `&mut T` 类型通过 `DerefMut` 特征的方法转换为 `&mut U` 类型，对应上例中，就是将 `&mut MyBox<String>` 转换为 `&mut String`
- `T: Deref<Target=U>` 解读：将 `&T` 类型通过 `Deref` 特征的方法转换为 `&U` 类型



Drop 顺序

- **变量级别，按照逆序的方式**，`_x` 在 `_foo` 之前创建，因此 `_x` 在 `_foo` 之后被 `drop`
- **结构体内部，按照顺序的方式**，结构体 `_x` 中的字段按照定义中的顺序依次 `drop`



- 对于 Rust 而言，不允许显式地调用析构函数`x.drop()`，需要显示使用`drop(x)`

  

- 问题：

  ```rust
  #[derive(Debug)]
  struct Foo;
  
  impl Drop for Foo {
      fn drop(&mut self) {
          println!("Dropping Foo!")
      }
  }
  
  fn main() {
      let foo = Foo;
      // foo.drop();// 不允许显式地调用析构函数`x.drop()`
      // 使用 drop 函数 -> pub fn drop<T>(_x: T)
      drop(foo); // 输出：Dropping Foo!
  }
  
  ```

  - 我们实现的是 `fn drop(&mut self)`，与 `pub fn drop<T>(_x: T)`不同，但是函数内部与我们实现的`drop函数相同`（即输出的为我们实现的`drop`函数的代码），这是为什么？
  - rust中释放资源指什么？所有权转移且作用域结束?
    - 不用管，资源释放时编译器做的， `std::mem::drop`其实什么都干，仅仅做了所有权转移；资源释放是编译器确定作用域结束后做的事情
    - [对象析构](https://doc.rust-lang.org/reference/destructors.html)有两部份，一是实现了Drop的，先调用其drop函数；二是按定义顺序递归地依次析构其成员 
  - 通过自己实现的`drop`（可变借用）如何释放资源？
    - 不用管，资源释放时编译器做的， `std::mem::drop`其实什么都干，仅仅做了所有权转移；
    - 自己实现的`Drop::drop`实际上就是可以在`mem::drop`前被隐式的调用

  

问题：rust是如何做到不会在手动`drop`后（利用可变引用），自动`drop`（所有权转移）时，不会再次`drop`已经手动`drop`的资源；即如何避免二次析构的？



- 无法为一个类型同时实现 `Copy` 和 `Drop` 特征。
  - 实现了 `Copy` 的特征会被编译器隐式的复制，因此非常难以预测析构函数执行的时间和频率。因此这些实现了 `Copy` 的类型无法拥有析构函数。



- Rc::clone：这里的 `clone` **仅仅复制了智能指针并增加了引用计数，并没有克隆底层数据**，因此共享了底层的数据，这种**复制效率是非常高**的。