# Day six

- 在一个 `impl` 块内，`Self` 指代被实现方法的结构体类型，`self` 指代此类型的实例

- Rust 并没有一个与 `->` 等效的运算符；相反，Rust 有一个叫 **自动引用和解引用**的功能。这种自动引用的行为之所以有效，是因为方法有一个明确的接收者—— `self` 的类型。在给出接收者和方法名的前提下，Rust 可以明确地计算出方法是仅仅读取（`&self`），做出修改（`&mut self`）或者是获取所有权（`self`）

- Rust 中有一个约定俗称的规则，使用 `new` 来作为构造器的名称，出于设计上的考虑，Rust 特地没有用 `new` 作为关键字，因为是函数，所以不能用 `.` 的方式来调用，需要用 `::` 来调用

- const 泛型，定义的语法是 `const N: usize`，表示 const 泛型 `N` ，它基于的值类型是 `usize`。——使rust有利于矩阵运算

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

  

- 特殊情况——`collection`为引用切片（一般在函数调用过程中）

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

- ```rust
  struct Point<T: Add<T, Output = T>> { //限制类型T必须实现了Add特征，否则无法进行+操作。
      x: T,
      y: T,
  }
  
  
  // 不知道这里也没有解释对：
  
  impl<T: Add<T, Output = T>> Add for Point<T> { // 特征约束Add <>+ 默认泛型类型（T）参数（Output)
      type Output = Point<T>; // 关联类型
  
      fn add(self, p: Point<T>) -> Point<T> {
          Point{
              x: self.x + p.x,
              y: self.y + p.y,
          }
      }
  }
  ```
  
- 通过 `&` 引用或者 `Box<T>` 智能指针的方式来创建特征对象；`dyn` 关键字只用在特征对象的类型声明上，在创建时无需使用 `dyn`

-  `dyn` 不能单独作为特征对象的定义，原因是特征对象可以是任意实现了某个特征的类型，编译器在编译期不知道该类型的大小，不同的类型大小是不同的。而 `&dyn` 和 `Box<dyn>` 在编译期都是已知大小，所以可以用作特征对象的定义

- 特征对象（多种）与（泛型）特征约束（多种取一）

  ```rust
  // 通过特征对象实现
  pub struct Screen {
      // 存储了一个动态数组，里面元素的类型是 Draw 特征对象：
      // Box<dyn Draw>，任何实现了 Draw 特征的类型，都可以存放其中。
      pub components: Vec<Box<dyn Draw>>,
  }
  
  impl Screen {
      pub fn run(&self) {
          for component in self.components.iter() {
              component.draw();
          }
      }
  }
  
  // 通过泛型实现
  pub struct Screen<T: Draw> {
      // 存储了类型为 T 的元素
      pub components: Vec<T>,
  }
  
  // 在 Screen 中使用特征约束让 T 实现了 Draw 特征，进而可以调用 draw 方法
  // 这种写法限制了 Screen 实例的 Vec<T> 中的每个元素必须是实现了Draw 特征的类型中的 `一种`
  impl<T> Screen<T>
      where T: Draw { // where 特征约束
      pub fn run(&self) {
          for component in self.components.iter() {
              component.draw();
          }
      }
  }
  ```

  - 如果只需要同质（相同类型）集合，更倾向于使用泛型和特征约束，因为实现更清晰，且性能更好
  - **特征对象**指向实现了 `Draw` 特征的类型的实例，这种映射关系是存储在一张表中，可以在运行时通过特征对象找到具体调用的类型方法
  - 特征对象，需要在运行时从 `vtable` 动态查找需要调用的方法，性能不如泛型+特征约束
  - ~~特征对象更加类似于go的interface~~(乱说的，更类似于鸭子类型操作)

- 静态分发与动态分发

  泛型是在编译期完成处理的：编译器会为每一个泛型参数对应的具体类型生成一份代码，这种方式是**静态分发(static dispatch)**，因为是在编译期完成的，对于运行期性能完全没有任何影响。

  与静态分发相对应的是**动态分发(dynamic dispatch)**，在这种情况下，直到运行时，才能确定需要调用什么方法。之前代码中的关键字 `dyn` 正是在强调这一“动态”的特点。

  当使用特征对象时，Rust 必须使用动态分发。编译器无法知晓所有可能用于特征对象代码的类型，所以它也不知道应该调用哪个类型的哪个方法实现。为此，Rust 在运行时使用特征对象中的指针来知晓需要调用哪个方法。动态分发也阻止编译器有选择的内联方法代码，这会相应的禁用一些优化。

- 静态分发 `Box<T>` 和动态分发 `Box<dyn Trait>` 的区别：

<img src="./pic/1.jpg" height="400" style="margin: 0 auto;" />

- **特征对象大小不固定**：这是因为，对于特征 `Draw`，类型 `Button` 可以实现特征 `Draw`，类型 `SelectBox` 也可以实现特征 `Draw`，因此特征没有固定大小
- 几乎总是使用特征对象的引用方式，如`&dyn Draw`，`Box<dyn Draw>`
  - 虽然特征对象没有固定大小，但它的引用类型的大小是固定的，它由两个指针组成（`ptr` 和 `vptr`），因此占用两个指针大小
  - 一个指针 `ptr` 指向实现了特征 `Draw` 的具体类型的实例，也就是当作特征 `Draw` 来用的类型的实例，比如类型 `Button` 的实例、类型 `SelectBox` 的实例
  - 另一个指针 `vptr` 指向一个虚表 `vtable`，`vtable` 中保存了类型 `Button` 或类型 `SelectBox` 的实例对于可以调用的实现于特征 `Draw` 的方法。当调用方法时，直接从 `vtable` 中找到方法并调用。之所以要使用一个 `vtable` 来保存各实例的方法，是因为实现了特征 `Draw` 的类型有多种，这些类型拥有的方法各不相同，当将这些类型的实例都当作特征 `Draw` 来使用时(此时，它们全都看作是特征 `Draw` 类型的实例)，有必要区分这些实例各自有哪些方法可调用

- 当类型 `Button` 实现了特征 `Draw` 时，类型 `Button` 的实例对象 `btn` 可以当作特征 `Draw` 的特征对象类型来使用，`btn` 中保存了作为特征对象的数据指针（指向类型 `Button` 的实例数据）和行为指针（指向 `vtable`）。

- 一定要注意，此时的 `btn` 是 `Draw` 的特征对象的实例，而**不再是具体类型 `Button` 的实例**
  -  `btn` 的 `vtable` 只包含了实现自特征 `Draw` 的那些方法（比如 `draw`）
  - 因此 `btn` 只能调用实现于特征 `Draw` 的 `draw` 方法，而**不能调用类型 `Button` 本身实现的方法**和类型 `Button` 实现于其他特征的方法。
  - **也就是说，`btn` 是哪个特征对象的实例，它的 `vtable` 中就包含了该特征的方法。**
  - 补充：调用函数时传入的`Box<T>` （类型实例）可以被隐式转换成函数参数签名中的 `Box<dyn Trait>`，这个被转换的出的即为**特征对象**

- 对象安全：
  - 方法的返回类型不能是 `Self`：一旦有了特征对象，就不再需要知道实现该特征的具体类型是什么了——特征对象忘记了其真正的类型，`Self`无意义
  - 方法没有任何泛型参数：当使用特征对象时其具体类型被抹去了，故而无从得知放入泛型参数类型到底是什么

- 关联类型
- 默认类型参数与类型约束
  - 默认类型参数（当使用泛型类型参数时，可以为其指定一个默认的具体类型）
  - 类型约束（使用特征约束有条件地实现方法或特征：指定类型 + 指定特征的条件下去实现方法）
  - 区分（方法名后的泛型`<T>` 与 方法名前）
- 调用同名的方法（不同特征拥有同名的方法；结构体方法与特征方法同名）
  - 优先调用类型上的方法
  - 显式调用特征方法
  - 完全限定语法`<Type as Trait>`
- 特征定义中的特征约束`supertrait` ：需要让某个特征 A 能使用另一个特征 B 的功能(另一种形式的特征约束)。不仅仅要为类型实现特征 A，还要为类型实现特征 B 才行
- 在外部类型上实现外部特征(newtype)
  - 孤儿规则，特征或者类型必需至少有一个是本地的，才能在此类型上定义特征。
  - 使用newtype 模式，绕过孤儿规则：就是为一个元组结构体创建新类型。该元组结构体封装有一个字段，该字段就是希望实现特征的具体类型。该封装类型是本地的，因此我们可以为此类型实现外部的特征。（ps: 类似go组合？）
- `Deref`特征