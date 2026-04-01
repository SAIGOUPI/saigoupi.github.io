---
layout: post
title: '游戏开发面试记录'
subtitle: ''
date: 2026-3-11
categories: 游戏开发
tags:  
- 游戏开发
---

## **网络同步**
### 帧同步
### TCP UDP

## **骨骼动画**
### IK

## **UI数据输入输出闭环**

## **Lua**

## **计算机动画**

## **寻路算法**
### A*
### 流场寻路

## **避障算法**
### RVO(相对速度障碍物算法)

## **Unreal**
### **TArray**
#### 为什么UE要自创TArray而不是直接用Vector

**1. 垃圾回收和反射系统**
* TArray：可以被标记为 UPROPERTY()。这意味着它能被 UE 的反射系统识别。如果 TArray 中存储的是 UObject*，引擎的垃圾回收器（GC）会自动追踪这些引用，防止对象在数组还在使用时被销毁。

* std::vector：无法被 UPROPERTY() 标记，引擎无法追踪其内部存储的指针。如果你在 std::vector 中存储了一个 UObject*，即便数组还在，该对象也可能被 GC 回收，导致悬空指针和崩溃。

**2. 内存分配和对齐**
* FMemory：TArray 使用 UE 的 FMemory 分配器，这允许引擎在不同平台（如 PS5、Xbox、Switch）上执行特定的内存优化，并方便开发者进行内存快照（Memory Profiling）分析。

* 对齐要求：在处理 SIMD（单指令多数据流）指令时，数据需要特定字节对齐。TArray 的分配器可以轻松支持这些硬件级的内存对齐要求，而 std::vector 通常使用通用分配器，控制力较弱。

**3. 序列化与网络复制**
* 序列化：通过 FArchive，TArray 可以实现“一键式”保存和加载，并能自动处理大小端转换等跨平台问题。

* 网络同步：UE 的网络架构（Replication）专门针对 TArray 做了同步优化。你可以标记一个 TArray 进行同步，引擎会处理数据的增量更新，而 std::vector 无法直接集成到这个网络协议栈中。

**4. 总结**

| 特性 | TArray | std::vector |
| :--- | :--- | :--- |
| **UE 反射系统支持**| 支持 (可标记 UPROPERTY) | 不支持 |
| **垃圾回收 (GC)** | 自动追踪内部 UObject | 无法追踪，易导致野指针 |
| **内存分配器** | FMemory (跨平台优化) | 标准 C++ 分配器 (STL) |
| **序列化/保存**| 内置支持 | 需手动实现遍历存储 |
| **网络同步** | 支持属性复制 | 不支持 |

UE开发中优先使用TArray，使数据和内存都在引擎的管理下，安全和性能都有保证。

**`Add`和`Emplace`的区别**
* Add()和Emplace()方法向数组末尾添加元素
* Emplace 可以使用参数直接构造元素，避免产生临时对象，Emplace()不创建临时变量，在容器内 “原地构造” 元素
* Add 底层调用的也是 Emplace，但是会先去构造一个对象，添加时创建临时变量，复制到数组中

#### TArray高级用法
**1. 预分配内存：使用 `Reserve`**
当数组元素超过当前容量时，引擎会申请更大的内存块并拷贝旧元素。
* **优化逻辑**：如果你预先知道数组的大致规模，调用 `Reserve(ExpectedSize)`。
* **收益**：将多次昂贵的内存分配和拷贝操作合并为一次。

**2. 避免冗余拷贝：`Emplace` 优于 `Add`**
* **`Add`**：通常会先创建一个临时对象，再将其拷贝或移动到数组中。
* **`Emplace`**：利用 C++ 完美转发，直接在数组预留的内存中构造对象。
* **收益**：减少了临时对象的构造与析构开销。

**3. 高效删除：`RemoveAtSwap`**
* **`RemoveAt`**：删除元素后，后续所有元素都会前移以保持顺序，复杂度为 $O(n)$。
* **`RemoveAtSwap`**：将数组最后一个元素覆盖到被删除位置，不保证顺序，复杂度为 $O(1)$。
* **收益**：在大规模数组删除操作中性能提升巨大。

**4. 内存池化：`Reset` 与 `Empty`**
* **`Empty()`**：删除所有元素并释放内存。
* **`Reset()`**：删除所有元素但**保留内存（Slack）**。
* **收益**：在循环反复填充同一个数组时，使用 `Reset()` 可避免反复触发堆分配。

**5. 栈空间优化：`TInlineAllocator`**
默认情况下 `TArray` 在堆（Heap）上分配内存。
* **用法**：`TArray<FVector, TInlineAllocator<16>> MyArray;`
* **收益**：前 16 个元素直接分配在栈上。对于生命周期短、元素少的小数组，完全消除了堆分配的开销。

**6. 参数传递：使用 `const TArray&`**
* **规则**：函数传参时严禁使用值传递（会触发整组拷贝）。
* **写法**：`void Process(const TArray<int32>& Data)`。

**7. 总结**

| 场景 | 推荐方式 | 复杂度/收益 |
| :--- | :--- | :--- |
| **已知元素数量** | `Reserve(N)` | 减少内存重新分配 |
| **添加复杂对象** | `Emplace(...)` | 避免临时对象拷贝 |
| **删除（不计顺序）** | `RemoveAtSwap(Index)` | 从 $O(n)$ 降至 $O(1)$ |
| **高频重用数组** | `Reset()` | 复用现有内存空间 |
| **局部临时小数组** | `TInlineAllocator<N>` | 消除堆分配开销 |

**参考文章：**
* [为性能表现最优化TArray的使用](https://www.unrealengine.com/zh-CN/blog/optimizing-tarray-usage-for-performance)

### **GAS网络同步**
### **移动组件网络同步**
### **DS优化**

### **UObject生命周期**
UObject的生命周期可分为以下几个阶段：

**1. 创建阶段：内存分配和初始化**
* UObject通过NewObject<T>()创建
* ClassConstructor()调用类的构造函数，C++成员变量，组件默认值(CreateDefaultSubobject<>())真正被初始化
* PostInitProperties() 子类自定义函数

NewObject<T>()
│
│  ① 打包参数
├─→ FStaticConstructObjectParameters
│
│  ② 进入引擎内部
├─→ StaticConstructObject_Internal()
│   │
│   │  ▼▼▼ 阶段一：分配 + 注册 ▼▼▼
│   ├─→ StaticAllocateObject()
│   │   ├── AllocateUObject()         // 分配原始内存
│   │   ├── Memzero()                 // 清零（安全保障）
│   │   ├── placement new UObjectBase // 填充元数据
│   │   └── AddObject()
│   │       ├── AllocateUObjectIndex() // 注册到全局数组（GC用）
│   │       └── HashObject()           // 注册到全局哈希表（查找用）
│   │
│   │  ▼▼▼ 阶段二：构造 + 初始化 ▼▼▼
│   └─→ ClassConstructor(FObjectInitializer)
│       ├── C++ 构造函数              // 你的代码在这里跑
│       └── ~FObjectInitializer()     // RAII 自动触发
│           ├── InitProperties()      // 从 CDO/Template 拷贝属性
│           ├── PostInitProperties()  // 虚函数回调（子类自定义）
│           └── ClearFlags(RF_NeedInitialization)  // 标记：完成！
│
▼
返回完全可用的 UObject*

**2. 激活阶段：BeginPlay和Tick**
* AActor或者UActorComponent会触发BeginPlay()和Tick

**3. 标记阶段：标记准备GC**
* 当没有任何被UPROPERTY()宏标记的指针指向该对象时，下一次GC扫描中会被标记销毁
* 对于 AActor，调用 Destroy()
* 对于普通 UObject，可以调用 MarkAsGarbage() 或 ConditionalBeginDestroy()。
* 对象被标记销毁后，并不会立即从内存消失，而是进入一个“待回收”的队列。

**4. 回收阶段：销毁和内存回收**
* BeginDestroy()：销毁前最后一次机会处理非托管资源
* IsReadyForFinishDestroy()：检查异步操作（如物理线程计算）是否完成
* FinishDestroy()：UObject释放内部资源
* 调用C++析构函数

**参考文章：**

* [UE UObject 系统底层机制解析：反射、GC 与内存管理](https://blog.jcmz.art/archives/ue-uobject-system-reflection-gc-memory)
* [UE UObject 创建流程详解](https://zhuanlan.zhihu.com/p/2020970695018979988)

### 行为树


## **数据结构和算法**
### 求二维平面上一个点P是否在一个三角形ABC内
* 用向量叉乘 P和三角形的各个顶点，检查向量是否都在在三角形边的同侧
* 面积法 底*高/2

### 洗牌算法
* 100个数内，取n个数字，要求不能重复。
解法：先对数组洗牌，然后取头n个树

## **C++**

### 引用
参考文章：
* [一文读懂C++右值引用和std::move](https://zhuanlan.zhihu.com/p/335994370])

### 内存对齐
**1. 什么是内存对齐**

现代计算机系统对于基本类型数据在内存中存放的位置有限制，要求这些数据的首地址的值是4或者8的倍数。因为处理器在读取内存时，一般按照双字节,四字节,8字节,16字节甚至32字节为单位来存取内存。按照内存对齐的规则存放数据后，处理器读取数据时能更快，且避免内存浪费

比如一个数据地址以2开头，占4个字节，那处理器按照4字节读取内存时，就要先从0地址开始读，然后截取地址2开头后的数据，再从4地址读取，截取地址5前面的数据，两次数据拼凑才能获得原始数据；如果数据从地址0或者地址4开始存储，那么处理器一次就能读取完整的数据

**2. 内存对齐规则**

有的数据就只有1个字节，那么要满足上面的内存对齐原则，方便处理器读取，就要对源数据进行内存对齐的操作，这样的操作要遵守一定的规则：

* 基本类型的对齐值就是sizeof值
* 结构体的对齐值是结构体的sizeof值
* 编译器可以设置一个最大对齐值(#pragma pack(n))，实际对齐值是此类型与最大对齐值中的最小值

```C++
//下面两个结构体, 在#pragmapack(4)和#pragmapack(8)的情况下，结构体的大小分别
#pragma pack(4) //默认对齐数修改成4
struct One {
	double d;  //0-7偏移
	char c;    //8偏移
	           //9-11偏移浪费
	int i;     //12-15偏移
};             //结构体one占16字节
struct Two {
	char c;    //0偏移
	           //1-3偏移浪费
	double d;  //4-11偏移
	int i;     //12-15偏移
};             //结构体two占16字节

#pragma pack(8) //默认对齐数修改成8
struct One {
	double d;  //0-7偏移
	char c;    //8偏移
			   //9-11偏移浪费
	int i;     //12-15偏移
};             //结构体one占16字节
struct Two {
	char c;    //0偏移
			   //1-3偏移浪费
	double d;  //8-15偏移
	int i;     //16-19偏移
	           //20-23偏移浪费
};             //结构体two占24字节

```