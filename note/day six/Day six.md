# Day six

- 在一个 `impl` 块内，`Self` 指代被实现方法的结构体类型，`self` 指代此类型的实例

- Rust 并没有一个与 `->` 等效的运算符；相反，Rust 有一个叫 **自动引用和解引用**的功能。这种自动引用的行为之所以有效，是因为方法有一个明确的接收者—— `self` 的类型。在给出接收者和方法名的前提下，Rust 可以明确地计算出方法是仅仅读取（`&self`），做出修改（`&mut self`）或者是获取所有权（`self`）

- Rust 中有一个约定俗称的规则，使用 `new` 来作为构造器的名称，出于设计上的考虑，Rust 特地没有用 `new` 作为关键字，因为是函数，所以不能用 `.` 的方式来调用，需要用 `::` 来调用

-  const 泛型，定义的语法是 `const N: usize`，表示 const 泛型 `N` ，它基于的值类型是 `usize`。——使rust有利于矩阵运算

- Rust 通过在编译时进行泛型代码的 **单态化**(*monomorphization*)来保证效率。单态化是一个通过填充编译时使用的具体类型，将通用代码转换为特定代码的过程。我们可以使用泛型来编写不重复的代码，而 Rust 将会为每一个实例编译其特定类型的代码。这意味着在使用泛型时没有运行时开销；当代码运行，它的执行效率就跟好像手写每个具体定义的重复代码一样。这个单态化过程正是 Rust 泛型在运行时极其高效的原因。

- 如果你想要为类型 `A` 实现特征 `T`，那么 `A` 或者 `T` 至少有一个是在当前作用域中定义的！

- 默认实现允许调用相同特征中的其他方法，哪怕这些方法没有默认实现。如此，特征可以提供很多有用的功能而只需要实现指定的一小部分内容。

- 特征约束(trait bound) 与 `impl Trait` 

- 泛型类型 `T` 说明了 `item1` 和 `item2` 必须拥有同样的类型，同时 `T: Summary` 说明了 `T` 必须实现 `Summary` 特征。

  ```rust
  pub fn notify<T: Summary>(item1: &T, item2: &T) {}
  ```

- 一般情况`collection`为数组等

- | 使用方法                      | 等价使用方式                                      | 所有权     |
  | ----------------------------- | ------------------------------------------------- | ---------- |
  | `for item in collection`      | `for item in IntoIterator::into_iter(collection)` | 转移所有权 |
  | `for item in &collection`     | `for item in collection.iter()`                   | 不可变借用 |
  | `for item in &mut collection` | `for item in collection.iter_mut()`               | 可变借用   |

  

-   特殊情况——`collection`为引用切片（一般在函数调用过程中）

  ```rust
  // 堆分配耗时
  fn largest<T: PartialOrd + Clone>(list: &[T]) -> T {
      let mut largest = list[0].clone(); // largest is T
  
      for item in list.iter() {
          // item is &T
          if item > &largest {
              // &T > &T
              // largest = *item;  // move
              largest = item.clone(); //largest is mut; clone 因为没有写 copy
          }
      }
  
      largest
  }
  
  fn largest<T: PartialOrd + Copy>(list: &[T]) -> T {
      let mut largest = list[0]; // 需要实现Copy特征, 否则move; 看起来largest是T
  
      for item in list.iter() {
          if item > &largest {
              largest = *item; // Copy
          }
      }
  
      largest
  }
  
  fn largest<T: PartialOrd + Copy>(list: &[T]) -> T {
      let mut largest = list[0]; // 需要实现Copy特征, 否则move; 看起来largest是T
  
      for &item in list.iter() {
          // 模式匹配 &item item为T
          if item > largest {
              largest = item;
          }
      }
  
      largest
  }
  
  fn largest<T: PartialOrd>(list: &[T]) -> &T {
      // 区分 ref 与 &
  
      // let ref mut largest = list[0];
      // largest 是 &mut list[0]
      // largest 是list[0]的可变引用, 修改largest 相当于修改list[0]
  
      let mut largest = &list[0]; // largest is &T 可以去指向其他list[i]
  
      for item in list.iter() {
          // 模式匹配 &item item为T
          if item > largest {
              largest = item;
          }
      }
  
      largest
  }
  
  fn main() {
      let number_list = vec![34, 50, 25, 100, 65];
  
      let result = largest(&number_list);
      println!("The largest number is {}", result);
  
      let char_list = vec!['y', 'm', 'a', 'q'];
  
      let result = largest(&char_list);
      println!("The largest char is {}", result);
  }
  ```

  问题：切片引用每个元素类型是什么？比如这里的`list[0]` （是T）

  ```rust
  fn largest<T: PartialOrd + Copy>(list: &[T]) -> T {
      let mut largest = list[0];  // largest类型是通过list[0]的类型推断的吗？
      ...
  }
  ```

  