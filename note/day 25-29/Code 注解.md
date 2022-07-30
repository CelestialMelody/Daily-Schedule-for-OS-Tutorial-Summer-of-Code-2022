# Lab 2

---

[地址格式与组成](http://rcore-os.cn/rCore-Tutorial-Book-v3/chapter4/3sv39-implementation-1.html#id3)

<img src="./pic/sv39-va-pa.png" alt="v39-va-pa.png" style="zoom:80%;" />



> 地址转换是以页为单位进行的，在地址转换的前后地址的 **页内偏移部分不变**。可以认为 MMU 只是从虚拟地址中取出 27 位虚拟页号，在页表中查到其对应的物理页号（如果存在的话），最后将得到的44位的物理页号与虚拟地址的12位页内偏移依序拼接到一起就变成了56位的物理地址

地址和页号的类型定义；各自抽象出不同的类型而不是都使用与RISC-V 64硬件直接对应的 usize 基本类型，目的是通过多种方便且安全的类型转换 (Type Conversion) 来构建页表

```rust
// os/src/mm/address.rs

#[derive(Copy, Clone, Ord, PartialOrd, Eq, PartialEq)]
pub struct PhysAddr(pub usize);

#[derive(Copy, Clone, Ord, PartialOrd, Eq, PartialEq)]
pub struct VirtAddr(pub usize);

#[derive(Copy, Clone, Ord, PartialOrd, Eq, PartialEq)]
pub struct PhysPageNum(pub usize);

#[derive(Copy, Clone, Ord, PartialOrd, Eq, PartialEq)]
pub struct VirtPageNum(pub usize);
```

这些类型本身可以和 usize 之间互相转换，以物理地址 `PhysAddr` 为例

```rust
const PA_WIDTH_SV39: usize = 56;
const PPN_WIDTH_SV39: usize = PA_WIDTH_SV39 - PAGE_SIZE_BITS; 
// `PAGE_SIZE_BITS` 为 12，表示页内偏移的位宽

impl From<usize> for PhysAddr {
    fn from(v: usize) -> Self { Self(v & ( (1 << PA_WIDTH_SV39) - 1 )) }
}
impl From<usize> for PhysPageNum {
    fn from(v: usize) -> Self { Self(v & ( (1 << PPN_WIDTH_SV39) - 1 )) }
}

impl From<PhysAddr> for usize {
    fn from(v: PhysAddr) -> Self { v.0 }
}
impl From<PhysPageNum> for usize {
    fn from(v: PhysPageNum) -> Self { v.0 }
}
```

地址和页号之间可以相互转换；以物理地址和物理页号之间的转换为例

```rust
// os/src/mm/address.rs

impl PhysAddr {
    pub fn page_offset(&self) -> usize { self.0 & (PAGE_SIZE - 1) } 
    // `PAGE_SIZE` 为 4096 -> 12位，表示每个页面的大小
}

// from PhysAddr to PhysPageNum
impl From<PhysAddr> for PhysPageNum {
    fn from(v: PhysAddr) -> Self {
        assert_eq!(v.page_offset(), 0);
        v.floor()
        // 不对齐的情况
    }
}

impl From<PhysPageNum> for PhysAddr {
    fn from(v: PhysPageNum) -> Self { Self(v.0 << PAGE_SIZE_BITS) } 
    // `PAGE_SIZE_BITS` 为 12，表示页内偏移的位宽
}
```

> 从物理页号到物理地址的转换只需左移 12 位即可，但是物理地址需要保证它与页面大小对齐才能通过右移转换为物理页号；对于不对齐的情况，物理地址不能通过 `From/Into` 转换为物理页号，而是需要通过它自己的 `floor` 或 `ceil` 方法来进行下取整或上取整的转换

```rust
// os/src/mm/address.rs

impl PhysAddr {
    pub fn floor(&self) -> PhysPageNum { PhysPageNum(self.0 / PAGE_SIZE) }
    pub fn ceil(&self) -> PhysPageNum { PhysPageNum((self.0 + PAGE_SIZE - 1) / PAGE_SIZE) 
}
```



---

[页表项的数据结构抽象与类型定义](http://rcore-os.cn/rCore-Tutorial-Book-v3/chapter4/3sv39-implementation-1.html#id5)

```rust
// os/src/main.rs

#[macro_use]
extern crate bitflags;

// os/src/mm/page_table.rs

use bitflags::*;

// 标志位 PTEFlags
bitflags! {
    pub struct PTEFlags: u8 {
        const V = 1 << 0;
        const R = 1 << 1;
        const W = 1 << 2;
        const X = 1 << 3;
        const U = 1 << 4;
        const G = 1 << 5;
        const A = 1 << 6;
        const D = 1 << 7;
    }
}
```

页表项

```rust
// os/src/mm/page_table.rs

#[derive(Copy, Clone)] 
// 实现 Copy/Clone Trait，来让这个类型以值语义赋值/传参的时候不会发生所有权转移，而是拷贝一份新的副本
#[repr(C)]
pub struct PageTableEntry {
    pub bits: usize,
}

impl PageTableEntry {
    // 从一个物理页号 PhysPageNum 和一个页表项标志位 PTEFlags 生成一个页表项 PageTableEntry 实例
    pub fn new(ppn: PhysPageNum, flags: PTEFlags) -> Self {
        PageTableEntry {
            bits: ppn.0 << 10 | flags.bits as usize,
        }
    }
    pub fn empty() -> Self {
        PageTableEntry {
            bits: 0,
        }
    }
    // 取出ppn
    pub fn ppn(&self) -> PhysPageNum {
        (self.bits >> 10 & ((1usize << 44) - 1)).into()
    }
    // 取出flags
    pub fn flags(&self) -> PTEFlags {
        PTEFlags::from_bits(self.bits as u8).unwrap()
    }
}

// 判断一个页表项的 V/R/W/X 标志位是否为 1; 以 V 标志位的判断为例
// os/src/mm/page_table.rs
impl PageTableEntry {
    pub fn is_valid(&self) -> bool {
        (self.flags() & PTEFlags::V) != PTEFlags::empty()
    }
}
```



---

[可用物理页的分配与回收](http://rcore-os.cn/rCore-Tutorial-Book-v3/chapter4/4sv39-implementation-2.html#id3)

需要知道物理内存的哪一部分是可用的；在 `os/src/linker.ld` 中，我们用符号 `ekernel` 指明了内核数据的终止物理地址，在它之后的物理内存都是可用的

```rust
// os/src/config.rs

pub const MEMORY_END: usize = 0x80800000;
// 硬编码整块物理内存的终止物理地址为 `0x80800000` 
```

[之前](http://rcore-os.cn/rCore-Tutorial-Book-v3/chapter1/3first-instruction-in-kernel1.html#term-physical-memory) 提到过物理内存的起始物理地址为 `0x80000000` ，这意味着我们将可用内存大小设置为 8MiB 

用一个左闭右开的物理页号区间来表示可用的物理内存，则：

- 区间的左端点应该是 `ekernel` 的物理地址以上取整方式转化成的物理页号；
- 区间的右端点应该是 `MEMORY_END` 以下取整方式转化成的物理页号

通过区间，物理页帧管理器用于初始化

```rust
// FrameAllocator Trait 来描述一个物理页帧管理器需要提供哪些功能

// os/src/mm/frame_allocator.rs
trait FrameAllocator {
    fn new() -> Self; // 创建一个物理页帧管理器的实例
    // 以物理页号为单位进行物理页帧的分配和回收
    fn alloc(&mut self) -> Option<PhysPageNum>; 
    fn dealloc(&mut self, ppn: PhysPageNum);
}
```

栈式物理页帧管理策略 `StackFrameAllocator`

```rust
// os/src/mm/frame_allocator.rs

pub struct StackFrameAllocator {
    current: usize,  //空闲内存的起始物理页号
    end: usize,      //空闲内存的结束物理页号
    recycled: Vec<usize>,
}
```

物理页号区间 [ `current` , `end` ) 此前均 *从未* 被分配出去过，而向量 `recycled` 以后入先出的方式保存了被回收的物理页号

初始化

- 在通过 `FrameAllocator` 的 `new` 方法创建实例的时候，只需将区间两端均设为 0 ，然后创建一个新的向量；

- 而在它真正被使用起来之前，需要调用 `init` 方法将自身的 [current,end) 初始化为可用物理页号区间

```rust
// os/src/mm/frame_allocator.rs

impl FrameAllocator for StackFrameAllocator {
    fn new() -> Self {
        Self {
            current: 0,
            end: 0,
            recycled: Vec::new(),
        }
    }
}

impl StackFrameAllocator {
    pub fn init(&mut self, l: PhysPageNum, r: PhysPageNum) {
        self.current = l.0;
        self.end = r.0;
    }
}
```

物理页帧分配和回收实现

```rust
// os/src/mm/frame_allocator.rs

impl FrameAllocator for StackFrameAllocator {
    fn alloc(&mut self) -> Option<PhysPageNum> {
        if let Some(ppn) = self.recycled.pop() {
            Some(ppn.into())
        } else {
            if self.current == self.end {
                None
            } else {
                self.current += 1;
                Some((self.current - 1).into())
            }
        }
    }
    fn dealloc(&mut self, ppn: PhysPageNum) {
        let ppn = ppn.0;
        // validity check
        if ppn >= self.current || self.recycled
            .iter()
            .find(|&v| {*v == ppn})
            .is_some() {
            panic!("Frame ppn={:#x} has not been allocated!", ppn);
        }
        // recycle
        self.recycled.push(ppn);
    }
}
```

创建 `StackFrameAllocator` 的全局实例 `FRAME_ALLOCATOR` 

```rust
// os/src/mm/frame_allocator.rs

use crate::sync::UPSafeCell;
type FrameAllocatorImpl = StackFrameAllocator;
lazy_static! {
    pub static ref FRAME_ALLOCATOR: UPSafeCell<FrameAllocatorImpl> = unsafe {
        UPSafeCell::new(FrameAllocatorImpl::new())
    };
}
```

这里我们使用 `UPSafeCell<T>` 来包裹栈式物理页帧分配器。每次对该分配器进行操作之前，我们都需要先通过 `FRAME_ALLOCATOR.exclusive_access()` 拿到分配器的可变借用

物理页帧全局管理器 `FRAME_ALLOCATOR` 初始化

```rust
// os/src/mm/frame_allocator.rs

pub fn init_frame_allocator() {
    extern "C" {
        fn ekernel();
    }
    FRAME_ALLOCATOR
        .exclusive_access()
        .init(PhysAddr::from(ekernel as usize).ceil(), PhysAddr::from(MEMORY_END).floor());
}
```



---

[分配/回收物理页帧的接口](http://rcore-os.cn/rCore-Tutorial-Book-v3/chapter4/4sv39-implementation-2.html#id4)

公开给其他内核模块调用的分配/回收物理页帧的接口

```rust
// os/src/mm/frame_allocator.rs

pub fn frame_alloc() -> Option<FrameTracker> {
    FRAME_ALLOCATOR
        .exclusive_access()
        .alloc()
        .map(|ppn| FrameTracker::new(ppn))
}

fn frame_dealloc(ppn: PhysPageNum) {
    FRAME_ALLOCATOR
        .exclusive_access()
        .dealloc(ppn);
}
```

借用了 [RAII](https://zh.m.wikipedia.org/zh-hans/RAII) 的思想，将一个物理页帧的生命周期绑定到一个 `FrameTracker` 变量上，当一个 `FrameTracker` 被创建的时候，我们需要从 `FRAME_ALLOCATOR` 中分配一个物理页帧

```rust
// os/src/mm/frame_allocator.rs

pub struct FrameTracker {
    pub ppn: PhysPageNum,
}

impl FrameTracker {
    pub fn new(ppn: PhysPageNum) -> Self {
        // page cleaning
        let bytes_array = ppn.get_bytes_array(); // 获取 PageAddr 对应的 mut [u8] 切片
        for i in bytes_array {
            *i = 0;
        }
        Self { ppn }
    }
}
```

将分配来的物理页帧的物理页号作为参数传给 `FrameTracker` 的 `new` 方法来创建一个 `FrameTracker` 实例
由于这个物理页帧之前可能被分配过并用做其他用途，我们在这里直接将这个物理页帧上的所有字节清零

> PS: 这里的物理页号 转换为 物理地址 返回 PageAddr 对应的 mut [u8] 切片

当一个 `FrameTracker` 生命周期结束被编译器回收的时候，我们需要将它控制的物理页帧回收到 `FRAME_ALLOCATOR` 中

```rust
// os/src/mm/frame_allocator.rs

impl Drop for FrameTracker {
    fn drop(&mut self) {
        frame_dealloc(self.ppn);
    }
}
```



---

[页表基本数据结构与访问接口](http://rcore-os.cn/rCore-Tutorial-Book-v3/chapter4/4sv39-implementation-2.html#id6)

> ps: 这里的实现与页表机制并不一样（就是页表机制 图中的页表 与这里定义的页表并不一致 所以暂时还不是很明白意义？仅仅是为了保存根页表？另外保存的是 物理页号 的封装 framtracker，并非 物理页号 也并非 页表项）； 只保存了三级页表的root_ppn 和 所有节点的物理页帧

```rust
// os/src/mm/page_table.rs

pub struct PageTable {
    root_ppn: PhysPageNum, // 是三级页表的位置 satp 的ppn
    frames: Vec<FrameTracker>,
}

impl PageTable {
    pub fn new() -> Self {
        let frame = frame_alloc().unwrap();
        PageTable {
            root_ppn: frame.ppn,
            frames: vec![frame],
        }
    }
}
```

`PageTable` 要保存它根节点的物理页号 `root_ppn` 作为页表唯一的区分标志

此外，向量 `frames` 以 `FrameTracker` 的形式保存了页表所有的节点（包括根节点）所在的物理页帧。
将这些 `FrameTracker` 的生命周期进一步绑定到 `PageTable` 下面

当 `PageTable` 生命周期结束后，向量 `frames` 里面的那些 `FrameTracker` 也会被回收，也就意味着存放多级页表节点的那些物理页帧被回收了

当我们通过 `new` 方法新建一个 `PageTable` 的时候，它只需有一个根节点。为此我们需要分配一个物理页帧 `FrameTracker` 并挂在向量 `frames` 下，然后更新根节点的物理页号 `root_ppn` 



操作系统需要动态维护一个虚拟页号到页表项的映射，支持插入/删除键值对 （后文实现）

```rust
impl PageTable {
    pub fn map(&mut self, vpn: VirtPageNum, ppn: PhysPageNum, flags: PTEFlags);
    pub fn unmap(&mut self, vpn: VirtPageNum);
}
```

- 通过 `map` 方法来在多级页表中插入一个键值对，注意这里将物理页号 `ppn` 和页表项标志位 `flags` 作为不同的参数传入；
- 通过 `unmap` 方法来删除一个键值对，在调用时仅需给出作为索引的虚拟页号即可



---

[内核中访问物理页帧的方法](http://rcore-os.cn/rCore-Tutorial-Book-v3/chapter4/4sv39-implementation-2.html#id7)

构造可变引用来直接访问一个物理页号 `PhysPageNum` 对应的物理页帧，不同的引用类型对应于物理页帧上的一种不同的内存布局

```rust
// os/src/mm/address.rs

impl PhysPageNum {
    
    // get_pte_array 返回的是一个 页表项定长数组 的可变引用，代表 多级页表中的一个节点
    pub fn get_pte_array(&self) -> &'static mut [PageTableEntry] {
        let pa: PhysAddr = self.clone().into(); // 从 PhysPageNum to  PageAddr
        unsafe {
            core::slice::from_raw_parts_mut(pa.0 as *mut PageTableEntry, 512)
        }
    }
    
    // get_bytes_array 返回的是一个字节数组的可变引用，可以以字节为粒度对物理页帧上的数据进行访问，前面进行数据清零就用到了这个方法
    pub fn get_bytes_array(&self) -> &'static mut [u8] {
        let pa: PhysAddr = self.clone().into(); // 从 PhysPageNum to  PageAddr
        unsafe {
            core::slice::from_raw_parts_mut(pa.0 as *mut u8, 4096)
        }
    }
    
   	//  get_mut 是个泛型函数，可以获取一个恰好放在一个物理页帧开头的类型为 T 的数据的可变引用
    pub fn get_mut<T>(&self) -> &'static mut T {
        let pa: PhysAddr = self.clone().into(); // 从 PhysPageNum to  PageAddr
        unsafe {
            (pa.0 as *mut T).as_mut().unwrap()
        }
    }
}
```

实现方面，都是先把物理页号转为物理地址 `PhysAddr` ，然后再转成 usize 形式的物理地址。接着，我们直接将它转为裸指针用来访问物理地址指向的物理内存

注意，我们在返回值类型上附加了静态生命周期泛型 `'static` ，这是为了绕过 Rust 编译器的借用检查，实质上可以将返回的类型也看成一个裸指针，因为它也只是标识数据存放的位置以及类型。但与裸指针不同的是，无需通过 `unsafe` 的解引用访问它指向的数据，而是可以像一个正常的可变引用一样直接访问



---

[建立和拆除虚实地址映射关系](http://rcore-os.cn/rCore-Tutorial-Book-v3/chapter4/4sv39-implementation-2.html#id8)

建立和拆除虚实地址映射关系的 `map` 和 `unmap` 方法是如何实现 —— 多级页表中找到一个虚拟地址对应的页表项

```rust
// os/src/mm/address.rs

impl VirtPageNum {
    // 取出虚拟页号的三级页索引，并按照从高到低的顺序返回
    pub fn indexes(&self) -> [usize; 3] {
        let mut vpn = self.0;
        let mut idx = [0usize; 3];
        for i in (0..3).rev() { // 反转
            idx[i] = vpn & 511;
            vpn >>= 9;
        }
        idx
    }
}

// os/src/mm/page_table.rs

impl PageTable {
    
    // 在多级页表找到一个虚拟页号对应的页表项的可变引用
    fn find_pte_create(&mut self, vpn: VirtPageNum) -> Option<&mut PageTableEntry> {
        let idxs = vpn.indexes();
        let mut ppn = self.root_ppn; // 获取三级页表的位置
        let mut result: Option<&mut PageTableEntry> = None;
        for i in 0..3 {
            let pte = &mut ppn.get_pte_array()[idxs[i]];
            if i == 2 {
                // 即使找到的页表项不合法，还是会将其返回回去而不是返回 None -- map / unmap 中进行判断
                result = Some(pte);
                break;
            }
            // 如果在遍历的过程中发现有节点尚未创建则会新建一个节点
            if !pte.is_valid() {
                let frame = frame_alloc().unwrap();
                *pte = PageTableEntry::new(frame.ppn, PTEFlags::V);
                self.frames.push(frame);
            }
            ppn = pte.ppn();
        }
        result
    }
    fn find_pte(&self, vpn: VirtPageNum) -> Option<&mut PageTableEntry> {
        let idxs = vpn.indexes();
        let mut ppn = self.root_ppn;
        let mut result: Option<&mut PageTableEntry> = None;
        for i in 0..3 {
            let pte = &mut ppn.get_pte_array()[idxs[i]];
            if i == 2 {
                // 即使找到的页表项不合法，还是会将其返回回去而不是返回 None -- map / unmap 中进行判断
                result = Some(pte);
                break;
            }
            if !pte.is_valid() {
                return None;
            }
            ppn = pte.ppn();
        }
        // 当找不到合法叶子节点的时候不会新建叶子节点而是直接返回 `None` 即查找失败
        result 
    }
}
```

- `VirtPageNum` 的 `indexes`，注意它里面包裹的 usize 可能有 27 位，也有可能有 64−12=52 位，但这里我们是用来在多级页表上进行遍历，因此只取出低 27 位

- <img src="./pic/sv39-full.png" alt="sv39-full.png" style="zoom: 80%;" />

  

- `find_pte_create`：变量 `ppn` 表示当前节点的物理页号，最开始指向多级页表的根节点。随后每次循环通过 `get_pte_array` 将取出当前节点的页表项数组，并根据当前级页索引找到对应的页表项。如果当前节点是一个叶节点，那么直接返回这个页表项的可变引用；否则尝试向下走。走不下去的话就新建一个节点，更新作为下级节点指针的页表项，并将新分配的物理页帧移动到向量 `frames` 中方便后续的自动回收。注意在更新页表项的时候，不仅要更新物理页号，还要将标志位 V 置 1，不然硬件在查多级页表的时候，会认为这个页表项不合法，从而触发 Page Fault 而不能向下走
- `PageTable::find_pte` 与 `find_pte_create` 的不同在于当找不到合法叶子节点的时候不会新建叶子节点而是直接返回 `None` 即查找失败。因此，它不会尝试对页表本身进行修改，但是注意它返回的参数类型是页表项的可变引用，也即它允许我们修改页表项
- 即使找到的页表项不合法，还是会将其返回回去而不是返回 `None` 。这说明在目前的实现中，页表和页表项是相对解耦合的



操作系统需要动态维护一个虚拟页号到页表项的映射，支持插入/删除键值对

```rust
impl PageTable {
    pub fn map(&mut self, vpn: VirtPageNum, ppn: PhysPageNum, flags: PTEFlags);
    pub fn unmap(&mut self, vpn: VirtPageNum);
}
```

- 通过 `map` 方法来在多级页表中插入一个键值对，注意这里将物理页号 `ppn` 和页表项标志位 `flags` 作为不同的参数传入；
- 通过 `unmap` 方法来删除一个键值对，在调用时仅需给出作为索引的虚拟页号即可

map / unmap 实现

```rust
// os/src/mm/page_table.rs

impl PageTable {
    pub fn map(&mut self, vpn: VirtPageNum, ppn: PhysPageNum, flags: PTEFlags) {
        let pte = self.find_pte_create(vpn).unwrap();
        assert!(!pte.is_valid(), "vpn {:?} is mapped before mapping", vpn);
        *pte = PageTableEntry::new(ppn, flags | PTEFlags::V);
    }
    pub fn unmap(&mut self, vpn: VirtPageNum) {
        let pte = self.find_pte(vpn).unwrap();
        assert!(pte.is_valid(), "vpn {:?} is invalid before unmapping", vpn);
        *pte = PageTableEntry::empty();
    }
}
```

为 PageTable 提供一种类似 MMU 操作的手动查页表的方法

```rust
// os/src/mm/page_table.rs

impl PageTable {
    
    /// Temporarily used to get arguments from user space.
    // from_token 临时创建一个专用来手动查页表的 PageTable ，它仅有一个从传入的 satp token 中得到的多级页表根节点的物理页号，它的 frames 字段为空，也即不实际控制任何资源
    pub fn from_token(satp: usize) -> Self {
        Self {
            root_ppn: PhysPageNum::from(satp & ((1usize << 44) - 1)),
            frames: Vec::new(),
        }
    }
    
    // translate 调用 find_pte 来实现，如果能够找到页表项，那么它会将页表项拷贝一份并返回，否则就返回一个 None
    pub fn translate(&self, vpn: VirtPageNum) -> Option<PageTableEntry> {
        self.find_pte(vpn)
            .map(|pte| {pte.clone()})
    }
}
```

当遇到需要查一个特定页表（非当前正处在的地址空间的页表时），可先通过 `PageTable::from_token` 新建一个页表，再调用它的 `translate` 方法查页表



---

[逻辑段：一段连续地址的虚拟内存](http://rcore-os.cn/rCore-Tutorial-Book-v3/chapter4/5kernel-app-spaces.html#id4)

```rust
// os/src/mm/memory_set.rs

// 以逻辑段 MapArea 为单位描述一段连续地址的虚拟内存
pub struct MapArea {
    vpn_range: VPNRange, // VPNRange 描述一段虚拟页号的连续区间，表示该逻辑段在地址区间中的位置和长度 [os/src/mm/address.rs]
    data_frames: BTreeMap<VirtPageNum, FrameTracker>,
    map_type: MapType,
    map_perm: MapPermission,
}
```

`MapType` 描述该逻辑段内的所有虚拟页面映射到物理页帧的同一种方式，它是一个枚举类型，在内核当前的实现中支持两种方式

```rust
// os/src/mm/memory_set.rs

#[derive(Copy, Clone, PartialEq, Debug)]
pub enum MapType {
    Identical, //  Identical 恒等映射方式
    Framed, // Framed 表示对于每个虚拟页面都有一个新分配的物理页帧与之对应，虚地址与物理地址的映射关系是相对随机的
}
```

恒等映射方式主要是用在启用多级页表之后，内核仍能够在虚存地址空间中访问一个特定的物理地址指向的物理内存

当逻辑段采用 `MapType::Framed` 方式映射到物理内存的时候， `data_frames` 是一个保存了该逻辑段内的每个虚拟页面和它被映射到的物理页帧 `FrameTracker` 的一个键值对容器 `BTreeMap` 中，这些物理页帧被用来存放实际内存数据而不是作为多级页表中的中间节点。

和之前的 `PageTable` 一样，这也用到了 RAII 的思想，将这些物理页帧的生命周期绑定到它所在的逻辑段 `MapArea` 下，当逻辑段被回收之后这些之前分配的物理页帧也会自动地同时被回收

`MapPermission` 表示控制该逻辑段的访问方式，它是页表项标志位 `PTEFlags` 的一个子集，仅保留 U/R/W/X 四个标志位，因为其他的标志位仅与硬件的地址转换机制细节相关，这样的设计能避免引入错误的标志位。

- R(Read)/W(Write)/X(eXecute)：分别控制索引到这个页表项的对应虚拟页面是否允许读/写/执行；
- U(User)：控制索引到这个页表项的对应虚拟页面是否在 CPU 处于 U 特权级的情况下是否被允许访问；

```rust
// os/src/mm/memory_set.rs

bitflags! {
    pub struct MapPermission: u8 {
        const R = 1 << 1;
        const W = 1 << 2;
        const X = 1 << 3;
        const U = 1 << 4;
    }
}
```



---

[地址空间：一系列有关联的逻辑段](http://rcore-os.cn/rCore-Tutorial-Book-v3/chapter4/5kernel-app-spaces.html#id5)

地址空间

```rust
// os/src/mm/memory_set.rs

pub struct MemorySet {
    page_table: PageTable, // 地址空间的多级页表 page_table
    areas: Vec<MapArea>, // 一个逻辑段 MapArea 的向量 areas
}
```

`PageTable` 下挂着所有多级页表的节点所在的物理页帧，而每个 `MapArea` 下则挂着对应逻辑段中的数据所在的物理页帧，这两部分合在一起构成了一个地址空间所需的所有物理页帧。这同样是一种 RAII 风格，当一个地址空间 `MemorySet` 生命周期结束后，这些物理页帧都会被回收。

地址空间 `MemorySet` 的方法

```rust
// os/src/mm/memory_set.rs

impl MemorySet {
    
    // new_bare 方法可以新建一个空的地址空间
    pub fn new_bare() -> Self {
        Self {
            page_table: PageTable::new(),
            areas: Vec::new(),
        }
    }
    
    // push 方法可以在当前地址空间插入一个新的逻辑段 map_area 
    // 如果它是以 Framed 方式映射到物理内存，还可以可选地在那些被映射到的物理页帧上写入一些初始化数据 data 
    fn push(&mut self, mut map_area: MapArea, data: Option<&[u8]>) {
        map_area.map(&mut self.page_table);
        if let Some(data) = data {
            map_area.copy_data(&mut self.page_table, data); // copy_data
        }
        self.areas.push(map_area);
    }
    
    // insert_framed_area 方法调用 push ，可以在当前地址空间插入一个 Framed 方式映射到物理内存的逻辑段
    // 注意该方法的调用者要保证同一地址空间内的任意两个逻辑段不能存在交集
    pub fn insert_framed_area(
        &mut self,
        start_va: VirtAddr, end_va: VirtAddr, permission: MapPermission
    ) {
        self.push(MapArea::new(
            start_va,
            end_va,
            MapType::Framed,
            permission,
        ), None);
    }
    
    // new_kernel 可以生成内核的地址空间
    pub fn new_kernel() -> Self;

    // from_elf 分析应用的 ELF 文件格式的内容，解析出各数据段并生成对应的地址空间
    pub fn from_elf(elf_data: &[u8]) -> (Self, usize, usize);
}
```

在实现 `push` 方法在地址空间中插入一个逻辑段 `MapArea` 的时候，需要同时维护地址空间的多级页表 `page_table` 记录的虚拟页号到页表项的映射关系，也需要用到这个映射关系来找到向哪些物理页帧上拷贝初始数据

`MapArea` 提供的另外几个方法

```rust
// os/src/mm/memory_set.rs

impl MapArea {
    
    // new 方法可以新建一个逻辑段结构体，注意传入的起始/终止虚拟地址会分别被下取整/上取整为虚拟页号并传入迭代器 vpn_range 中
    pub fn new(
        start_va: VirtAddr,
        end_va: VirtAddr,
        map_type: MapType,
        map_perm: MapPermission
    ) -> Self {
        let start_vpn: VirtPageNum = start_va.floor();
        let end_vpn: VirtPageNum = end_va.ceil();
        Self {
            vpn_range: VPNRange::new(start_vpn, end_vpn),
            data_frames: BTreeMap::new(),
            map_type,
            map_perm,
        }
    }
    
    // 将 当前逻辑段 到 物理内存的映射 从 传入的该逻辑段所属的地址空间的多级页表 中加入
    pub fn map(&mut self, page_table: &mut PageTable) {
        for vpn in self.vpn_range {
            self.map_one(page_table, vpn);
        }
    }
    
    // 将 当前逻辑段 到 物理内存的映射 从 传入的该逻辑段所属的地址空间的多级页表 中删除
    pub fn unmap(&mut self, page_table: &mut PageTable) {
        for vpn in self.vpn_range {
            self.unmap_one(page_table, vpn);
        }
    }

    // copy_data 方法 将切片 data 中的 数据 拷贝到当前逻辑段实际被内核放置在的 各物理页帧 上，从而在地址空间中通过该逻辑段就能访问这些数据
    pub fn copy_data(&mut self, page_table: &mut PageTable, data: &[u8]) {
        assert_eq!(self.map_type, MapType::Framed);
        let mut start: usize = 0;
        let mut current_vpn = self.vpn_range.get_start();
        let len = data.len();
        loop {
            let src = &data[start..len.min(start + PAGE_SIZE)];
            let dst = &mut page_table
                .translate(current_vpn)
                .unwrap()
                .ppn()
                .get_bytes_array()[..src.len()];
            dst.copy_from_slice(src);
            start += PAGE_SIZE;
            if start >= len {
                break;
            }
            current_vpn.step();
        }
    }
}
```

- `map` 与 `unmap` 的实现是遍历逻辑段中的所有虚拟页面，并以 每个虚拟页面 为单位依次在多级页表中进行键值对的插入或删除，分别对应 `MapArea` 的 `map_one` 和 `unmap_one` 方法
