# Daily Schedule for OS Training Camp 2022

OS Training Camp Daily Documents

每日笔记 借鉴[yunwei37](https://github.com/yunwei37)姐姐的日志安排

---

## TOC

**七月**

|                 Mon                 |                Tues                 |           Wed           |             Thur              |              Fri              |           Sat           |                 Sun                 |
| :---------------------------------: | :---------------------------------: | :---------------------: | :---------------------------: | :---------------------------: | :---------------------: | :---------------------------------: |
|                                     |                                     |                         |     30<br>([D0](#day-0))      |     1 <br> ([D1](#day-1))     |   2<br>([D2](#day-2))   |         3<br>([D3](#day-3))         |
|        4<br/>([D4](#day-4))         |        5<br/>([D5](#day-5))         |  6<br/>([D6](#day-6))   |     7<br/>([D7](#day-7))      |     8<br/>([D8](#day-8))      |  9<br/>([D9](#day-9))   | 10<br/>([D10-11-12](#day-10-11-12)) |
| 11<br/>([D10-11-12](#day-10-11-12)) | 12<br/>([D10-11-12](#day-10-11-12)) | 13<br/>([D13](#day-13)) | 14<br/>([D14-15](#day-14-15)) | 15<br/>([D14-15](#day-14-15)) | 16<br/>([D16](#day-16)) |    17<br/>([D17-18](#day-17-18))    |
|    18<br/>([D17-18](#day-17-18))    |                                     |                         |                               |                               |                         |                                     |
|                                     |                                     |                         |                               |                               |                         |                                     |

**八月**

| Mon | Tues | Wed | Thur | Fri | Sat | Sun |
| :-: | :--: | :-: | :--: | :-: | :-: | :-: |
|     |      |     |      |     |     |     |
|     |      |     |      |     |     |     |
|     |      |     |      |     |     |     |
|     |      |     |      |     |     |     |
|     |      |     |      |     |     |     |

---

<h2 id="day-0">Day 0 2022/6/30</h2>

#### 事件一：简单配置一下仓库，决定接下来 rust 的学习

- 学习 rust 圣经，写这里的时候才到[看这里](https://course.rs/basic/base-type/numbers.html)，菜菜

- [rust-begin](https://github.com/CelestialMelody/rust-begin)

  完成变量绑定与解构（Variables）的练习

- [rust-begin-rustlings](https://github.com/CelestialMelody/rust-begin-rustlings)

  体验一下，完成 intro 部分

#### 事件二：收到 OSTrainingCamp 入营通知

- 开心
- 看到[yunwei37](https://github.com/yunwei37)姐姐的日志安排，orz，厉害捏（顺便加了姐姐 QQ，开心！不过不知道聊啥，，，呆）

#### 问题

- 学习效率和进度好慢呀，不知道能不能完成第一阶段，呜呜呜

#### 规划

- 赶紧入门 rust 呀



<h2 id="day-1">Day 1 2022/7/1</h2>

[今日笔记](https://github.com/CelestialMelody/daily_schedule_for_os_traning_camp_2022/blob/main/note/day%20one/Day%20one.md)

#### Rust 学习

- 学习 rust 圣经+rust-by-prctice 提交到[rust-begin ](https://github.com/CelestialMelody/rust-begin)；完成练习如下：

  - char-bool-unit
  - functions
  - numbers
  - ownership
  - statement-exprissions

- [rust-begin-rustlings ](https://github.com/CelestialMelody/rust-begin-rustlings) 完成练习如下：
  - variables
  - functions

#### 其他日常

这两天中午都吃的肯德基，好饱呀。。。哈哈哈哈

收到了[春季清华 OS 课程](https://github.com/LearningOS/rust-based-os-comp2022/blob/main/relatedinfo.md)的链接，还没来得及看呢（现在点开看一眼

#### 规划

- 这周赶紧看完基础入门



<h2 id="day-2">Day 2 2022/7/2</h2>

[今日笔记](https://github.com/CelestialMelody/daily_schedule_for_os_traning_camp_2022/blob/main/note/day%20two/Day%20two.md)

#### Rust 学习

- 学习 rust 圣经+rust-by-prctice 提交到[rust-begin ](https://github.com/CelestialMelody/rust-begin)；完成练习如下：
  - String
  - slice
  - tuple
  - struct
- [rust-begin-rustlings ](https://github.com/CelestialMelody/rust-begin-rustlings) 还没来得及做呢，，，rust 圣经的变量看完了，明天开搞

#### Golang 学习

- 稍微看了一点点基础（虽然自己之前知道这些，不过巩固一下基础也好，要重新开始看了
- [go-begin](https://github.com/CelestialMelody/go-begin.git) 放仓库了 🐳

#### 其他

- 学习 rust 时没理解不可变引用与原变量为啥可以都在`println!()`中引用，，，万能群友解释是`println!()`宏展开是用了不可变引用（还没看到宏，，，
- rust 引用切片与 golang 切片类似，rust 引用底层是切片的第一个字是指向数据的指针，第二个字是切片的长度，golang 还多了一个 cap。所以 rust 引用切片大小为 2 个字，golang 切片估计是 3 个字。这些数据的类型大小均与 cpu 相关（联系到了 golang 到了 ╰(_°▽°_)╯
- 今天做一个简单的链表题（原谅我算法太差了，开始从基础做），顺便把 cpp `delete`和析构函数复习了一下 👍
- 讲道理，我很想把 go，rust，cpp，java，javascript，python 都好好看看，不过没有那么多时间呀，而且学习 rust 效率也没那么高，看完章节配合 rust-by-practice 练习，花时间挺长的(┬┬﹏┬┬)，，，希望这个暑假至少把 golang 和 rust，还有操作系统基础打好把，，，，太菜了 ಥ_ಥ



<h2 id="day-3">Day 3 2022/7/3</h2>

- 今日配置麻了，没有做什么事，呜呜呜
  实验0之前生成早了，与最新的不一样了，，，不过大概了解了怎么连接远程仓库

  ````
  git remote add upstream 远程仓库链接
  git fetch upstream
  git checkout -b foo
  git branch -D main
  git checkout -t origin/main
  git reset --hard  upstream/main
  git push -f
  ````

- 新的要求来了：rustlings用github codespace的形式了(｢・ω・)｢
  虽然之前我自己fork了一份然后在本地做了，不过反正我也没做多少，也还好
  
- 总感觉自己过不了第一阶段了，呜呜呜



<h2 id="day-4">Day 4 2022/7/4</h2>

[今日笔记](https://github.com/CelestialMelody/daily_schedule_for_os_traning_camp_2022/blob/main/note/day%20four/Day%20four.md)

#### Rust 学习

- 学习 rust 圣经，基础部分才看到一半，看到`method`了（而且后面的一半更难ಥ_ಥ
- 枚举和`match`花了点时间理解
  - ps: rust-course，rust圣经居然没讲`ref`只是在练习里面出现了，我在rust-by-prctice里面看的`ref`
  - rust枚举也太神奇了，在rust-by-prctice里面看见**使用枚举实现链表**，震惊（见识短浅，见谅见谅(●'◡'●)

- [rustlings](https://github.com/LearningOS/learn_rust_rustlings-CelestialMelody) 大概做到`mod`部分了，但是我还没看到`mod`呢

#### 其他

- `lab 0`的实验终于过了，原因就是git action+private；不太理解git action(只知道好像是有什么CI，云原生？？？

- 果然是立flag了，rust-course基础部分还是没看完，害😢😢



<h2 id="day-5">Day 5 2022/7/5</h2>

[今日笔记](https://github.com/CelestialMelody/daily_schedule_for_os_traning_camp_2022/blob/main/note/day%20five/Day%20five.md)

今天是摸鱼的一天呢(┬┬﹏┬┬)

晚上开营直播，老师说基本上一半同学会止步于第一阶段，我觉得我有可能止步于第一阶段，，不过没啥担心的，我还是挺喜欢rust的，如果这次没过，以后也会坚持学习哈哈哈哈，有时间继续参加哈（说的我真的过不了第一阶段。。。害，还是有希望的，加油加油(❁´◡`❁)🎏🎢

#### 问题

遇见一个不明白的问题，放笔记了（关于rust的`@`绑定）

#### 规划

可以看看前两届大佬留下的[博客笔记](https://rcore-os.github.io/blog/)，对之后的学习，做实验有帮助

另外，快点把基本语法看完把，得基本有个印象(o-ωｑ)).oO



<h2 id="day-6">Day 6 2022/7/6</h2>

[今日笔记](https://github.com/CelestialMelody/daily_schedule_for_os_traning_camp_2022/blob/main/note/day%20six/Day%20six.md)

#### rust学习

特征Trait基本看完了，估计明天就可以看完rust圣经基础部分。今天看完特征了，看地似懂非懂的，哈哈哈哈🎢🐳

#### 其他

随便看了看操作系统和计网的一点点知识，复习了一点点go知识。

#### 规划

明天开始搞剩下了rustlings，不过我看rustlngs除了基础部分还有多线程编程，明天看完Rust圣经基础部分后，把能做的rustlings都完成了，剩下的不明白的再看Rust圣经高级部分.



<h2 id="day-7">Day 7 2022/7/7</h2>

[今日笔记](https://github.com/CelestialMelody/daily_schedule_for_os_traning_camp_2022/blob/main/note/day%20seven/Day%20seven.md)

#### rust学习

基础部分基本看完了，但是rustlings今天还没怎么做（指今天摸鱼了(ノへ￣、)

类型转换，那部分属实有点看不懂(┬┬﹏┬┬)

#### 规划

明天开始搞剩下了rustlings。。。

可以把lab看起来了(每日1h?)

rv任务:阅读《计算机组成与设计（RISC-V版）》第一、二章;自学RISC-V手册：一本开源指令集的指南(重点是第10章)



<h2 id="day-8">Day 8 2022/7/8</h2>

[今日笔记](https://github.com/CelestialMelody/daily_schedule_for_os_traning_camp_2022/blob/main/note/day%20eight/Day%20eight.md)

#### rust学习

**rustlings**

- 错误处理6，挺难的
- 做到智能指针了（standard_library_types）



<h2 id="day-9">Day 9 2022/7/9</h2>

[今日笔记](https://github.com/CelestialMelody/daily_schedule_for_os_traning_camp_2022/blob/main/note/day%20nine/Day%20nine.md)

#### rust学习

学习智能指针，有点复杂。。。今天rustlings还未开动，，哎呀哎呀，感觉吃枣药丸哇哇哇哇இ௰இ



<h2 id="day-10-11-12">Day 10-12 2022/7/10-12</h2>

[day 10 笔记](https://github.com/CelestialMelody/daily_schedule_for_os_traning_camp_2022/blob/main/note/day%20ten/Day%20ten.md)

[day 11-12 笔记](https://github.com/CelestialMelody/daily_schedule_for_os_traning_camp_2022/blob/main/note/day%20eleven_twelve/Day%20eleven_twelve.md)

前两天基本上在摸鱼，没做什么关于本项目的事情，现在开始好好学习（x

实验部分还没看，rustlings还没做完（rust进阶部分理解有点耗时。。

ricv-32是我们计组课指令系统与控制器学习的指令集，虽然不是面面俱到，但是把基本的指令都介绍，复习也可以看我们计组课件



#### 目标

- 明天把rustlings弄完
- 开始渐渐看实验吧
- [RISC-V手册：一本开源指令集的指南](http://crva.io/documents/RISC-V-Reader-Chinese-v2p1.pdf)
  - 简单看看



#### 完成情况

**事件1**

[RISC-V手册：一本开源指令集的指南](http://crva.io/documents/RISC-V-Reader-Chinese-v2p1.pdf)阅览

- 只是简单看了看，其中第二、三章与我们计组课相关，但内容上各有不同：
  - 第二章简单介绍了rv32，并没有我们上课的课件详细，另外还将rv32与其他指令集进行了对比，不过我没仔细看，感觉rv吸收了之前指令集的经验教训，与rust类似，比较优秀



<h2 id="day-13">Day 13 2022/7/13</h2>

[今日笔记](https://github.com/CelestialMelody/daily_schedule_for_os_traning_camp_2022/blob/main/note/day%20thirteen/Day%20thirteen.md)

#### 完成情况

**事件1**

rustlings终于完成啦😂😁

#### 其他

rust进阶其实还有部分没看完；

感觉自己写一个稍微规范一点点的rust程序还是有点困难的，之前学的也还有一定的印象，遇到具体情况还是要复习。。。



<h2 id="day-14-15">Day 14-15 2022/7/14-15</h2>

[day 14 笔记](https://github.com/CelestialMelody/daily_schedule_for_os_traning_camp_2022/tree/main/note/day%20fifteen)

[day 15 笔记](https://github.com/CelestialMelody/daily_schedule_for_os_traning_camp_2022/tree/main/note/day%20fifteen)

#### 目标

- 文档：
  - 应用程序与基本执行环境
- rust：生命周期

#### 完成情况

**事件1**：文档阅读--应用程序与基本执行环境

之前只是配置了实验环境（lab0-0）但实际上对此实验细节还不是很了解；这篇文章讲解的就是该实验的具体细节部分；我很难记录一些笔记，感觉涉及的内容对于我来说比较晦涩，只能大致了解实验的流程。

**事件2**：rust生命周期学习

感觉还是不太明白，特别是生命周期的消除规则，以及提到的例子

**事件3**：阅读《行者无疆》、《网络是怎样连接的》

其实很久没有看文学书了，，，最近有点郁闷烦躁，看看文学书静静心，看书的时候，感觉余秋雨老师写的散文很有逻辑，而且通过序章明白了写这本书的目的——寻找文化对比坐标，更加清晰地理解中华文化；另外是关于网络的书籍，感觉讲的很有趣，第一章就给我讲明白了浏览器收发信息的过程，特别是DNS服务器部分，之前一直不理解，看了第一章恍然大悟，受益匪浅呀~

#### 其他

day 15可以说在摸鱼哈哈哈，所以这合并了

接下来要好好看实验了



<h2 id="day-16">Day 16 2022/7/16</h2>

[今日笔记](https://github.com/CelestialMelody/daily_schedule_for_os_traning_camp_2022/blob/main/note/day%20sixteen/Day%20sixteen.md)

#### 完成情况

**事件** 通过rcore-tutorial文档复习第一章

#### 其他

阅读rcore-tutorial，感觉对编译原理和计组有了更深入的理解



<h2 id="day-17-18">Day 17-18 2022/7/17-18</h2>

今日笔记

#### 完成情况

**事件1** 阅读rcore-tutorial文档——批处理系统

感觉阅读难度好高，只能大概看懂在做什么；看完后，回到引言部分：

- 首先改进应用程序，让它能够在用户态执行，并能发出系统调用
  - 调整程序的内存布局，让操作系统能够把应用加载到指定内存地址，然后顺利启动并运行应用程序
- 在应用程序的运行过程中，操作系统要支持应用程序的输出功能，并还能支持应用程序退出
  - 这需要实现跨特权级的系统调用接口，以及 `sys_write` 和 `sys_exit` 等具体的系统调用功能
  - 应用与操作系统内核之间系统调用的参数传递的约定
- 写完应用程序后，还需实现支持多个应用程序轮流启动运行的操作系统
  - 首先能把本来相对松散的应用程序执行代码和操作系统执行代码连接在一起，加载至内存，让操作系统能够找到应用程序的位置
  - 应用程序执行文件格式转换，便于操作系统能够找到应用的位置
- 让Binary应用能够启动和运行，需要分配好对应执行环境所需一系列的资源
  - 设置好用户栈和内核栈（在用户态的应用程序与在内核态的操作系统内核需要有各自的栈，避免应用程序破坏内核的执行）
  - 实现Trap上下文的保存与恢复（让应用能够在发出系统调用到内核态后，还能回到用户态继续执行）
  - 完成Trap分发与处理等工作

这里记录也并非完整，只是对[引言](http://rcore-os.cn/rCore-Tutorial-Book-v3/chapter2/0intro.html)的提炼；本章看完只能大概了解实验过程

**事件2** 跟着[os-comp2022第二章文档](https://learningos.github.io/rust-based-os-comp2022/chapter2/index.html)做实验



#### 其他
