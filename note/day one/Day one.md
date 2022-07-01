# Day one

### rust

- 赋值时类型不一致

  ```rust
  let v:u16 = 38_u8 as u16;
  ```

- ```rust
  let v2 = i8::checked_add(119, 8).unwrap();
  ```

  第一个数必须在i8范围内，checked_add可以显示处理溢出的panic

- Rust 的 HashMap 数据结构，是一个 KV 类型的 Hash Map 实现，它对于 K 没有特定类型的限制，但是要求能用作 K 的类型必须实现了 std::cmp::Eq 特征，因此这意味着你无法使用浮点数作为 HashMap 的 Key，来存储键值对，但是作为对比，Rust 的整数类型、字符串类型、布尔类型都实现了该特征，因此可以作为 HashMap 的 Key。

- ```rust
  assert!((0.1_f64 + 0.2 - 0.3).abs() < 0.000_1);
  ```

  这里必须加上`_f64`说明是浮点数，才有`abs()`方法

- ```rust
  assert_eq!((1..5), Range { start: 1, end: 5 });
  assert_eq!((1..=5), RangeInclusive::new(1, 5));
  ```

- 目前版本`let`认为一定是语句，需要`;`结尾

  ```rust
  let v = {
      let x = 3;
      x
  };
  println!("{}", v);
  assert!(v == 3);
  //
  let v = {
      let x = 3;
  };
  assert!(v == ());
  ```

  只有表达式能返回值，而 `;` 结尾的是语句，在 Rust 中，一定要严格区分**表达式**和**语句**的区别。

- const 要指明类型

- 任何基本类型的组合可以 `Copy` ，不需要分配内存或某种形式资源的类型是可以 `Copy` 的。不可变引用 `&T` 是`Copy`类型

- String 本质是堆上面的 Vec，Vec 的切片引用就是 &[]
  所以 &str 和 String，等同于 &[u8] 和 Vec
  那么 as_bytes 就是拿到字符串的切片引用 &[u8]，s 本身的所有权没有发生变化
  而 into_bytes() 是拿到字符串的 Vec，然后发生了 move 动作，所有权转移了

- 可变性：当所有权转移时，可变性也可以随之改变。

- 可变引用同时只能存在一个——同一作用域，特定数据只能有一个可变引用，这种限制的好处就是使 Rust 在编译期就避免数据竞争，数据竞争可由以下行为造成：

  - 两个或更多的指针同时访问同一数据
  - 至少有一个指针被用来写入数据
  - 没有同步数据访问的机制

- 可变引用与不可变引用不能同时存在

  - 注意，**引用的作用域 `s` 从创建开始，一直持续到它最后一次使用的地方**，这个跟变量的作用域有所不同，变量的作用域从创建持续到某一个花括号 `}`

  - 对于这种编译器优化行为，Rust 专门起了一个名字 —— **Non-Lexical Lifetimes(NLL)**，专门用于找到某个引用在作用域(`}`)结束前就不再被使用的代码位置。

  - PS：这个让我想到了**数据库**的**两段封锁协议**，哈哈哈哈

  - ```rust
    fn main() {
       let mut s = String::from("hello");
    
        let r1 = &s;
        let r2 = &s;
        println!("{} and {}", r1, r2);
        // 新编译器中，r1,r2作用域在这里结束
    
        let r3 = &mut s; // 不报错
        println!("{}", r3); // 可运行
    } // 老编译器中，r1、r2、r3作用域在这里结束
      // 新编译器中，r3作用域在这里结束
    
    // 引用作用域的结束位置从花括号变成最后一次使用的位置
    
    ```

- 所有权转移：move；深拷贝：clone；浅拷贝：copy

- 悬垂引用

- Rust 中的字符是 Unicode 类型，因此每个字符占据 4 个字节内存空间，但是在字符串中不一样，字符串是 UTF-8 编码，也就是字符串中的字符所占的字节数是变化的(1 - 4)

- 字符串replace：`replace()`返回新的字符串；`replacen()`返回新的字符串；`replace_range()`仅适用于 `String` 类型，**不**返回新的字符串

- 与字符串删除相关的方法有 4 个，他们分别是 `pop()`，`remove()`，`truncate()`，`clear()`。这四个方法仅适用于 `String` 类型。

- 使用 `+` 或者 `+=` 连接字符串，要求右边的参数必须为字符串的切片引用（Slice)类型。其实当调用 `+` 的操作符时，相当于调用了 `std::string` 标准库中的 [`add()`](https://doc.rust-lang.org/std/string/struct.String.html#method.add) 方法，这里 `add()` 方法的第二个参数是一个引用的类型。因此我们在使用 `+`， 必须传递切片引用类型。不能直接传递 `String` 类型。**`+` 和 `+=` 都是返回一个新的字符串。所以变量声明可以不需要 `mut` 关键字修饰**。另外，`+`**左边的String所有权转移**

- Rust `drop()` 与 C++：*Resource Acquisition Is Initialization (RAII)*



### 其他	

#### algo

**二叉树**

- 快速排序：二叉树的前序遍历；归并排序：二叉树的后序遍历

  ```java
  //快速排序的代码框架如下
  void sort(int[] nums, int lo, int hi) {
      /****** 前序遍历位置 ******/
      // 通过交换元素构建分界点 p
      int p = partition(nums, lo, hi);
      /************************/
  
      sort(nums, lo, p - 1);
      sort(nums, p + 1, hi);
  }
  
  // 定义：排序 nums[lo..hi]
  void sort(int[] nums, int lo, int hi) {
      int mid = (lo + hi) / 2;
      // 排序 nums[lo..mid]
      sort(nums, lo, mid);
      // 排序 nums[mid+1..hi]
      sort(nums, mid + 1, hi);
  
      /****** 后序位置 ******/
      // 合并 nums[lo..mid] 和 nums[mid+1..hi]
      merge(nums, lo, mid, hi);
      /*********************/
  }
  ```

- 前序位置的代码执行是自顶向下的，而后序位置的代码执行是自底向上的 --> 语法制导翻译

  ![image-1](./pic/image-1.jpg)

  - 意味着前序位置的代码只能从函数参数中获取父节点传递来的数据，而后序位置的代码不仅可以获取参数数据，还可以获取到子树通过函数返回值传递回来的数据

  - 举具体的例子，现在给你一棵二叉树，两个简单的问题：

    1、如果把根节点看做第 1 层，如何打印出每一个节点所在的层数？（子层数 = 父层数+1，继承属性，使用L-属性定义，自顶向下分析，先序遍历）

    2、如何打印出每个节点的左右子树各有多少节点？（节点数=左子树节点数+右子树节点数，综合属性，使用S-属性定义，自底向上分析，后序遍历）

- 层次遍历

  - ````java
    // 输入一棵二叉树的根节点，层序遍历这棵二叉树
    void levelTraverse(TreeNode root) {
        if (root == null) return;
        Queue<TreeNode> q = new LinkedList<>();
        q.offer(root);
    
        // 从上到下遍历二叉树的每一层
        while (!q.isEmpty()) {
            int sz = q.size();
            // 从左到右遍历每一层的每个节点
            for (int i = 0; i < sz; i++) {
                TreeNode cur = q.poll();
                // 将下一层节点放入队列
                if (cur.left != null) {
                    q.offer(cur.left);
                }
                if (cur.right != null) {
                    q.offer(cur.right);
                }
            }
        }
    }
    
    ````

    ![image-2](./pic/image-2.png)

- 
