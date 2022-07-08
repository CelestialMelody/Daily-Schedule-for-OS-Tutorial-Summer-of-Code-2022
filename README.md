# Daily Schedule for OS Training Camp 2022

OS Training Camp Daily Documents

每日笔记 借鉴[yunwei37](https://github.com/yunwei37)姐姐的日志安排

- [rust-begin](https://github.com/CelestialMelody/rust-begin)
  - [Rust By Practice( Rust 练习实践 )](https://zh.practice.rs/why-exercise.html)的练习
- [rust-begin-rustlings](https://github.com/CelestialMelody/rust-begin-rustlings)：forked from [rustlings](https://github.com/rust-lang/rustlings), 类似于 rust-by-practice

---

## TOC

**七月**

|         Mon          |         Tues         |         Wed          |         Thur         |          Fri          |         Sat         |         Sun         |
| :------------------: | :------------------: | :------------------: | :------------------: | :-------------------: | :-----------------: | :-----------------: |
|                      |                      |                      | 30<br>([D0](#day-0)) | 1 <br> ([D1](#day-1)) | 2<br>([D2](#day-2)) | 3<br>([D3](#day-3)) |
| 4<br/>([D4](#day-4)) | 5<br/>([D5](#day-5)) | 6<br/>([D6](#day-6)) | 7<br/>([D7](#day-7)) | 8<br/>([D8](#day-8))  |                     |                     |
|                      |                      |                      |                      |                       |                     |                     |
|                      |                      |                      |                      |                       |                     |                     |
|                      |                      |                      |                      |                       |                     |                     |

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







<h2 id="day-7">Day 7 2022/7/7</h2>

[今日笔记](https://github.com/CelestialMelody/daily_schedule_for_os_traning_camp_2022/blob/main/note/day%20eight/Day%20eight.md)

#### rust学习

**rustlings**

- 错误处理6，挺难的
- 做到智能指针了（standard_library_types）
