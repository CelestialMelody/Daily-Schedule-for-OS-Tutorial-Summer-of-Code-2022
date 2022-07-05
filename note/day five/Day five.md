# Day five

- _x 仍会将值绑定到变量，而 _ 则完全不会绑定；用 .. 忽略**剩余值**（最右），也不会绑定（回顾一下：没有使用`ref`的变量绑定——交付所有权）

  ```rust
  
  #![allow(unused)]
  fn main() {
  let s = Some(String::from("Hello!"));
  
  // if let Some(_) = s { // 下划线本身，则并不会绑定值，s所有权不变 最后可以打印
      
  if let Some(_s) = s { // _s 解构-匹配-绑定；所有权转移 value partially moved here
      println!("found a string");
  }// _s 作用域结束
  
  println!("{:?}", s); //   s不具有所有权，没法借用；value borrowed here after partial move
  }
  
  ```

  

- **匹配守卫**（*match guard*）是一个位于 `match` 分支模式之后的额外 `if` 条件，它能为分支模式提供更进一步的匹配条件。

  ```rust
  fn main() {
      let x = Some(String::from("hello"));
      let y = String::from("hello");
  
      match x {
          Some(ref n) if n == &y => println!("Matched, n = {}", n), // ref 引用 变量绑定时防止所有权转移；必须同种类型才能使用 `==` 故加上`&`
          _ => println!("Default case, x = {:?}", x),
      }
  
      println!("at the end: x = {:?}, y = {}", x, y);
  }
  ```

  