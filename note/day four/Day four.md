# Day four

- 查看变量类型

  ```rust
  use std::any::type_name;
  
  fn test_type<T>(_: T) {
    println!("{:?}", { type_name::<T>() });
  }
  
  // 改进 防止所有权转移
  fn type_of<T>(_: &T) {
      println!("{:?}", { type_name::<T>() });
  }
  ```

**枚举**

- 枚举成员没有具体说明类型时，整个枚举类型可以转化且只能转化为`isize`

  ```rust
  enum Number {
      Zero,
      One,
      Two,
  }
  
  test_type(Number::One);
  println!("{}", Number::One as i32); // 整个枚举类型可以被强转为isize
  
  enum Char {
      // c1 = 'a' as i8, // 只能是isize类型
      // c1 = 'a' as i64, //
      c1 = 'a' as isize,
      c2,
      c3,
  }
  let c1 = Char::c1;
  let c2 = Char::c2;
  let c3 = Char::c3;
  println!("{}", c1 as i8); // 这里可以转换为i8
  
  // println!("{}", c2); //wrong
  
  // test_type(c1);// c1 不是基本类型 类型为 Char（枚举） 所有权转移
  
  test_type(c2);
  ```

- 同一个枚举类型下的不同成员还能持有不同的数据类型，任何类型的数据都可以放入枚举成员中

  ```rust
  enum Message {
      Quit,
      Move { x: i32, y: i32 },
      Write(String),
      ChangeColor(i32, i32, i32),
  }
  // 枚举成员本身是枚举类型---Message
  test_type(Message::ChangeColor);
  // 实例化后是一种类型 -- Message::ChangeColor::{{constructor}}
  test_type(Message::ChangeColor(1, 2, 3)); 
  ```

- 前面发现直接使用`println!("{}"}`无法打印初始化后的枚举值；

  - 实际上与**结构体**类似，这种方式需要实现`trait`：` std::fmt::Display`；
  - 如果没有实现且需要打印，需要使用使用 `#[derive(Debug)]` 对结构体或者枚举类型进行标记，再使用 `println!("{:?}");` ，才能对其进行打印输出

- [同一化类型](https://course.rs/basic/compound-type/enum.html#同一化类型)使用枚举对类型进行同一化

  ```rust
  #[derive(Debug)]
  enum Message {
      Quit,
      Move { x: i32, y: i32 },
      Write(String),
      ChangeColor(i32, i32, i32),
  }
  
  let msgs: [Message; 3] = [
      Message::Quit,
      Message::Move { x: 1, y: 3 },
      Message::ChangeColor(255, 255, 0),
  ];
  
  for msg in msgs {
      show_message(msg)
  }
  
  fn show_message(msg: Message) {
      // println!("{}", msg);
      println!("{:?}", msg);
  }
  ```

- [ref 模式](https://rustwiki.org/zh-CN/rust-by-example/scope/borrow/ref.html) 

  **在通过 let 绑定来进行模式匹配或解构时，ref 关键字可用来创建结构体/元组的字段的引用。**

  ```rust
  #[derive(Clone, Copy)] // 基本类型 使用这句帮助实现拷贝 克隆 （都要写）
  struct Point { x: i32, y: i32 }
  
  fn main() {
      let c = 'Q';
  
      // 赋值语句中左边的 `ref` 关键字等价于右边的 `&` 符号。
      let ref ref_c1 = c;
      let ref_c2 = &c;
  
      println!("ref_c1 equals ref_c2: {}", *ref_c1 == *ref_c2);
  
      let point = Point { x: 0, y: 0 };
  
      // 在解构一个结构体时 `ref` 同样有效。
      let _copy_of_x = {
          // `ref_to_x` 是一个指向 `point` 的 `x` 字段的引用。
          let Point { x: ref ref_to_x, y: _ } = point;
  
          // 返回一个 `point` 的 `x` 字段的拷贝。
          *ref_to_x
      };
  
      // `point` 的可变拷贝
      let mut mutable_point = point; //已实现拷贝
  
      {
          // `ref` 可以与 `mut` 结合以创建可变引用。
          let Point { x: _, y: ref mut mut_ref_to_y } = mutable_point;
  
          // 通过可变引用来改变 `mutable_point` 的字段 `y`。
          *mut_ref_to_y = 1;
      }
  
      println!("point is ({}, {})", point.x, point.y);
      println!("mutable_point is ({}, {})", mutable_point.x, mutable_point.y);
  
      // 包含一个指针的可变元组
      let mut mutable_tuple = (Box::new(5u32), 3u32);
      
      {
          // 解构 `mutable_tuple` 来改变 `last` 的值。
          let (_, ref mut last) = mutable_tuple;
          *last = 2u32;
      }
      
      println!("tuple is {:?}", mutable_tuple);
  }
  
  ```

- **match** 中使用`ref`

  ```rust
  enum Favour<'a> {
      Nor(u32),
      NorRef(u32),
      Ref(&'a u32),
      RefRef(&'a u32),
  }
  
  fn log(fav: Favour) {
      match fav {
          Favour::Nor(data) => {
              config(&data);
              print_type_name_of(data);
          },
          Favour::NorRef(ref data) => {
              config(data);
              print_type_name_of(data);
          },
          Favour::Ref(data) => {
              config(data);
              print_type_name_of(data);
          },
          Favour::RefRef(ref data) => {
              config(data);
              print_type_name_of(data);
          }
      }
  }
  
  fn print_type_name_of<T>(_: T) {
      println!("{}", unsafe { std::intrinsics::type_name::<T>() })
  }
  
  fn config(data: &u32) {
      println!("log data: {}", data);
  }
  ```

- **ref 与 &**

  ```rust
  let x = String::from("hello");
  let ref y = x;
  let ref z = &x;
  let yy = &x;
  let zz = &z;
  
  println!("{}, {}, {} , {}, {}", x, y, z, yy, zz); // 都是hello
  
  // test_type(x); // x已经借用；test_type参数非不可变引用
  test_type(y); // "&alloc::string::String"
  test_type(z); // "&&alloc::string::String"
  test_type(yy); // 与 y 同 "&alloc::string::String"
  test_type(zz); // "&&&alloc::string::String"
  ```

- 使用枚举实现list `ref` 与 `*` 与 `&`

  ```rust
  // 填空，让代码运行
  use crate::List::*;
  
  enum List {
      // Cons: 链表中包含有值的节点，节点是元组类型，第一个元素是节点的值，第二个元素是指向下一个节点的指针
      Cons(u32, Box<List>),
      // Nil: 链表中的最后一个节点，用于说明链表的结束
      Nil,
  }
  
  // 为枚举实现一些方法
  impl List {
      // 创建空的链表
      fn new() -> List {
          // 因为没有节点，所以直接返回 Nil 节点
          // 枚举成员 Nil 的类型是 List
          Nil
      }
  
      // 在老的链表前面新增一个节点，并返回新的链表
      fn prepend(self, elem: u32) -> List {
          // 不是List::Cons
          Cons(elem, Box::new(self))
      }
  
      // 返回链表的长度
      fn len(&self) -> u32 {
          match *self {
              // 这里我们不能拿走 tail 的所有权，因此需要获取它的引用
              Cons(_, ref tail) => 1 + tail.len(), // 模式匹配 解构tail （ref 声明为引用）
              // 空链表的长度为 0
              Nil => 0,
          }
      }
  
      // 返回链表的字符串表现形式，用于打印输出
      fn stringify(&self) -> String {
          match *self {
              Cons(head, ref tail) => {
                  // 递归生成字符串
                  format!("{}, {}", head, tail.stringify())
              }
              Nil => {
                  format!("Nil")
              }
          }
      }
  }
  
  fn main() {
      // 创建一个新的链表(也是空的)
      let mut list = List::new();
  
      // 添加一些元素
      list = list.prepend(1);
      list = list.prepend(2);
      list = list.prepend(3);
  
      // 打印列表的当前状态
      println!("链表的长度是: {}", list.len());
      println!("{}", list.stringify());
  }
  
  ```

  - 在`len`方法和`stringify方法`中`match`：`self`加`*`进行解引用，类型就是`List`；这时`tail`加`ref`，类型就是`&Box<List>`，防止所有权转移。

  另一种

  ```rust
  // 返回链表的长度
  fn len(&self) -> u32 {
      match self {
          // 这里我们不能拿走 tail 的所有权，因此需要获取它的引用
          Cons(_, tail) => 1 + tail.len(),
          // 空链表的长度为 0
          Nil => 0,
      }
  }
  
  // 返回链表的字符串表现形式，用于打印输出
  fn stringify(&self) -> String {
      match self {
          Cons(head, ref tail) => {
              // 递归生成字符串
              format!("{}, {}", head, tail.stringify())
          }
          Nil => {
              format!("Nil")
          }
      }
  }
  ```

  - 在`len`方法和`stringify方法`中`match`：`self`不加`*`进行解引用的话，类型就是`&List`；这时`tail`不加`ref`，类型就是`&Box<List>`。
  - 如果加了`*`对`self`解引用，而又不加`ref`，那么`tial`的类型就是`Box<List>`，所有权就发生了转移。

- **匹配守卫**

  ```rust
  matches!(bar, Some(x) if x > 2)
  ```

- `if let`

  ```rust
  fn main() {
      let age = Some(30);
      println!("在匹配前，age是{:?}", age);
      if let Some(x) = age { // 若这里x写age会发生覆盖
          println!("匹配出来的x是{}", x); // 解构age ，x去匹配
          println!("age: {:?}", age);
      }
  
      println!("在匹配后，age是{:?}", age);
  }
  ```

- match 模式绑定

  ```rust
  fn main() {
      let x = Some(5);
      let y = 10;
  
      match x {
          Some(50) => println!("Got 50"),
          Some(y) => println!("Matched, y = {:?}", y),
          _ => println!("Default case, x = {:?}", x),
      }
  
      println!("at the end: x = {:?}, y = {:?}", x, y);
  }
  
  ```

  第二个匹配分支中的模式引入了一个新变量 `y`，它会匹配任何 `Some` 中的值。因为这里的 `y` 在 `match` 表达式的作用域中，而不是之前 `main` 作用域中，所以这是一个新变量，不是开头声明为值 10 的那个 `y`。这个新的 `y` 绑定会匹配任何 `Some` 中的值，在这里是 `x` 中的值。因此这个 `y` 绑定了 `x` 中 `Some` 内部的值。这个值是 5，所以这个分支的表达式将会执行并打印出 `Matched，y = 5`。

- 解构并分解值

  ```rust
  struct Point {
      x: i32,
      y: i32,
      string: String,
  }
  
  fn main() {
      let p = Point {
          x: 0,
          y: 7,
          string: "hello".to_string(),
      };
  
      let Point {
          x: a,
          y: b,
          string: s, // 所有权转移
  
                     // 防止所有权转移
                     // string: ref s,
      } = p;
      println!("{}", p.x); // x不会转移所有权？-> 应该是基本类型 实现了Copy
      println!("{}", p.string); // p.string 所有权转移， println! 使用借用 故报错为“value borrowed here after move”
  
      // let mut mutable_p = p;
      // println!("point is ({}, {})", p.x, p.y); // 所有权已经
  }
  ```

  
