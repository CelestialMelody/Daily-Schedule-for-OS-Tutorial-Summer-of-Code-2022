# Day seventeen_eighteen

### **批处理系统**

- **批处理系统** (Batch System)： 将多个程序打包到一起输入计算机；当一个程序运行结束后，计算机会自动执行下一个程序
- 保护操作系统不受出错程序破坏的机制被称为 **特权级** (Privilege) 机制， 它实现了用户态和内核态的隔离
- [特权级的软硬件协同设计](http://rcore-os.cn/rCore-Tutorial-Book-v3/chapter2/1rv-privilege.html#id3)
  - `ecall` 	`eret`
- [RISC-V 特权级架构](http://rcore-os.cn/rCore-Tutorial-Book-v3/chapter2/1rv-privilege.html#risc-v)
  - 4 种特权级 U S H M
  - [执行环境栈](http://rcore-os.cn/rCore-Tutorial-Book-v3/chapter1/1app-ee-platform.html#app-software-stack)
  - 执行环境的功能
    - 在执行它支持的上层软件之前进行一些初始化工作
    - 对上层软件的执行进行监控管理
  - 监督模式执行环境 (SEE, Supervisor Execution Environment)
  - 异常控制流 (ECF, Exception Control Flow) 被称为 异常（Exception）
  - 用户态应用直接触发从用户态到内核态的异常的原因
    - 用户态软件为获得内核态操作系统的服务功能而执行特殊指令
    - 在执行某条指令期间产生了错误（如执行了用户态不允许执行的指令或者其他错误）并被 CPU 检测到
  - RISC-V 异常
  - 陷入(trap)类的指令——陷入异常：断点 (Breakpoint) 和 执行环境调用 (Environment call) 
  - 其他的异常
    - 在执行某一条指令的时候发生了某种错误（如除零、无效地址访问、无效指令等）
    - 处理器认为处于当前特权级下执行的当前指令是高特权级指令或会访问不应该访问的高特权级的资源（可能危害系统）
  - 监督模式二进制接口 (Supervisor Binary Interface, SBI)
  - 应用程序二进制接口 (Application Binary Interface, ABI)——系统调用 (syscall, System Call) 
  - 陷入异常控制流 —— 切换 CPU 特权级
  - 将接口下降到机器/汇编指令级才能够满足其跨高级语言的通用性和灵活性
  - 每层特权级的软件都只能做高特权级软件允许它做的、且不会产生什么撼动高特权级软件的事情，一旦低特权级软件的要求超出了其能力范围，就必须寻求高特权级软件的帮助，否则就是一种异常行为
- [RISC-V的特权指令](http://rcore-os.cn/rCore-Tutorial-Book-v3/chapter2/1rv-privilege.html#term-csr-instr)
  - 每个特权级都对应的特殊指令 与 控制状态寄存器 (CSR, Control and Status Register) 
- 静态绑定 与 动态加载
- [找到并加载应用程序二进制码](http://rcore-os.cn/rCore-Tutorial-Book-v3/chapter2/3batch-system.html#id4)
  - Rust 所有权模型和借用检查
  - 内部可变性
  - `static mut` 变量的访问控制都是 unsafe 的
  - 当声明一个全局变量的时候，Rust编译器会默认会在多线程上使用它
  - 为`RefCell`实现`Sync`——允许在单核上安全使用可变全局变量
  - 数据缓存 (d-cache) 和 指令缓存 (i-cache)
- [特权级切换的起因](http://rcore-os.cn/rCore-Tutorial-Book-v3/chapter2/4trap-handling.html#id3)
- [特权级切换相关的控制状态寄存器](http://rcore-os.cn/rCore-Tutorial-Book-v3/chapter2/4trap-handling.html#id4)
  - [进入S特权级Trap的相关CSR](http://rcore-os.cn/rCore-Tutorial-Book-v3/chapter2/4trap-handling.html#id4:~:text=%E8%BF%9B%E5%85%A5%20S%20%E7%89%B9%E6%9D%83%E7%BA%A7%20Trap%20%E7%9A%84%E7%9B%B8%E5%85%B3%20CSR)
- [特权级切换](http://rcore-os.cn/rCore-Tutorial-Book-v3/chapter2/4trap-handling.html#id5)
- [特权级切换的硬件控制机制](http://rcore-os.cn/rCore-Tutorial-Book-v3/chapter2/4trap-handling.html#trap-hw-mechanism)
- [用户栈与内核栈](http://rcore-os.cn/rCore-Tutorial-Book-v3/chapter2/4trap-handling.html#id7)
- [Trap 上下文的保存与恢复](http://rcore-os.cn/rCore-Tutorial-Book-v3/chapter2/4trap-handling.html#id8)
- [Trap 分发与处理](http://rcore-os.cn/rCore-Tutorial-Book-v3/chapter2/4trap-handling.html#id9)
- [实现系统调用功能](http://rcore-os.cn/rCore-Tutorial-Book-v3/chapter2/4trap-handling.html#id10)
- CSR 相关原子指令
- sscratch CSR 的用途：中转寄存器，从用户栈到内核栈的切换
- [执行应用程序](http://rcore-os.cn/rCore-Tutorial-Book-v3/chapter2/4trap-handling.html#ch2-app-execution)
  - 从操作系统内核返回到运行应用程序之前要完成的工作

---

### lab0-1

- [inline assembly](https://doc.rust-lang.org/nightly/reference/inline-assembly.html)
- 代码目录：`os2` `user`
- 本实验不需要做任何操作，只用完成
  1. 接收 [第二个实验练习的github classroom在线邀请](https://classroom.github.com/a/UEOvz4qO)
  2. `make codespaces_setenv`
  3. `make setupclassroom_test2`

​	即可，关键应该是了解（最好做到理解做了什么）

---

### Rust

**复习**

>1. **发散函数**（diverging function）没有返回值，它使用感叹号 `!` 作为返回类型。发散函数一般都以 `panic!` 宏调用或调用其他发散函数结束，所以，调用发散函数会导致当前线程崩溃



