# Day sixteen

### 第一章笔记——环境

- [应用程序执行环境](http://rcore-os.cn/rCore-Tutorial-Book-v3/chapter1/1app-ee-platform.html#id4)

- [端序或尾序（大小端)_内存对齐](http://rcore-os.cn/rCore-Tutorial-Book-v3/chapter1/3first-instruction-in-kernel1.html#id3) 补充：[C语言字节对齐、结构体对齐](https://blog.csdn.net/lanzhihui_10086/article/details/44353381)
- [qume启动流程_真实计算机的加电启动流程](http://rcore-os.cn/rCore-Tutorial-Book-v3/chapter1/3first-instruction-in-kernel1.html#qemu)
- [程序内存布局](http://rcore-os.cn/rCore-Tutorial-Book-v3/chapter1/3first-instruction-in-kernel1.html#id7)
- [编译流程](http://rcore-os.cn/rCore-Tutorial-Book-v3/chapter1/3first-instruction-in-kernel1.html#id8)
- [静态链接与动态链接](http://rcore-os.cn/rCore-Tutorial-Book-v3/chapter1/4first-instruction-in-kernel2.html#id4)

- [可执行与可链接格式 （Executable and Linkable Format， ELF）](https://zh.wikipedia.org/zh-cn/%E5%8F%AF%E5%9F%B7%E8%A1%8C%E8%88%87%E5%8F%AF%E9%8F%88%E6%8E%A5%E6%A0%BC%E5%BC%8F)-> .o目标文件、os.bin内核镜像等等

- 函数调用：举例

  ```assembly
  call: jal  x1, imm20
  ...
  ret:  jalr x0, 0(x1)
  ```

- [函数调用与栈](http://rcore-os.cn/rCore-Tutorial-Book-v3/chapter1/5support-func-call.html#id3)

  - 多层嵌套调用
  - 函数调用上下文
  - 调用者保存寄存器 被调用者保存寄存器

- [调用规范](http://rcore-os.cn/rCore-Tutorial-Book-v3/chapter1/5support-func-call.html#term-calling-convention)

  - [RISC-V 架构上的 C 语言调用规范](https://riscv.org/wp-content/uploads/2015/01/riscv-calling.pdf)

  - 栈 栈帧stackframe 栈指针sp 栈帧指针fp

  - Rust 外部符号引用`extern "C"`
