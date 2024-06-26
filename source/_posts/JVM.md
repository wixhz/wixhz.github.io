---
title: JVM
date: 2021-12-10 21:21:48.12
updated: 2022-08-11 22:59:52.317
url: /archives/jvm
categories: 
- java基础
tags: 
---

# JVM概述

JVM：全称 Java Virtual Machine，即 Java 虚拟机，一种规范，本身是一个虚拟计算机，直接和操作系统进行交互，与硬件不直接交互，而操作系统可以帮我们完成和硬件进行交互的工作

![JVM-概述图](JVM/JVM-概述图-6b0f1b0647494d45892416333482c87a.png)

Java 代码执行流程：java程序 --（编译）--> 字节码文件 --（解释执行）--> 操作系统（Win，Linux）

## 架构模型

Java 编译器输入的指令流是一种基于栈的指令集架构。因为跨平台的设计，Java 的指令都是根据栈来设计的，不同平台 CPU 架构不同，所以不能设计为基于寄存器架构

* 基于栈式架构的特点：
  * 设计和实现简单，适用于资源受限的系统
  * 使用零地址指令方式分配，执行过程依赖操作栈，指令集更小，编译器容易实现
    * 零地址指令：机器指令的一种，是指令系统中的一种不设地址字段的指令，只有操作码而没有地址码。这种指令有两种情况：一是无需操作数，另一种是操作数为默认的（隐含的），默认为操作数在寄存器（ACC）中，指令可直接访问寄存器
    * 一地址指令：一个操作码对应一个地址码，通过地址码寻找操作数
  * 不需要硬件的支持，可移植性更好，更好实现跨平台
* 基于寄存器架构的特点：
  * 需要硬件的支持，可移植性差
  * 性能更好，执行更高效，寄存器比内存快
  * 以一地址指令、二地址指令、三地址指令为主

## 生命周期

JVM 的生命周期分为三个阶段，分别为：启动、运行、死亡。

- **启动**：当启动一个 Java 程序时，通过引导类加载器（bootstrap class loader）创建一个初始类（initial class），对于拥有 main 函数的类就是 JVM 实例运行的起点

- **运行**：

  - main() 方法是一个程序的初始起点，任何线程均可由在此处启动

  - 在 JVM 内部有两种线程类型，分别为：用户线程和守护线程，**JVM 使用的是守护线程，main() 和其他线程使用的是用户线程**，守护线程会随着用户线程的结束而结束

  - 执行一个 Java 程序时，真真正正在执行的是一个 Java 虚拟机的进程

  - JVM 有两种运行模式 Server 与 Client，两种模式的区别在于：Client 模式启动速度较快，Server 模式启动较慢；但是启动进入稳定期长期运行之后 Server 模式的程序运行速度比 Client 要快很多

    Server 模式启动的 JVM 采用的是重量级的虚拟机，对程序采用了更多的优化；Client 模式启动的 JVM 采用的是轻量级的虚拟机

- **死亡**：

  - 当程序中的用户线程都中止，JVM 才会退出
  - 程序正常执行结束、程序异常或错误而异常终止、操作系统错误导致终止
  - 线程调用 Runtime 类 halt 方法或 System 类 exit 方法，并且 java 安全管理器允许这次 exit 或 halt 操作

# 内存区域

Java 虚拟机在执行 Java 程序的过程中会把它管理的内存划分成若干个不同的数据区域

![Java运行时数据区域JDK1.8](JVM/Java运行时数据区域JDK1.8.png)

## 虚拟机栈

每个 Java 方法在执行的同时会创建一个栈帧（**一个方法一个栈帧**）用于存储局部变量表、操作数栈、动态连接、方法出口等信息。每个方法被调用直至执行完成的过程，对应着一个栈帧在虚拟机栈中入栈到出栈的过程

虚拟机栈特点：
- 虚拟机栈是**每个线程私有的**，对虚拟机来说只有在栈顶的方法才是运行的，位于栈顶的方法被称为"当前栈帧"
- 栈内存**不需要进行GC**，方法开始执行的时候会进栈，方法调用后自动弹栈，相当于清空了数据
- 栈内存分配越大越大，可用的线程数越少（内存越大，每个线程拥有的内存越大）
- 方法内的局部变量是否**线程安全**：
  - 如果方法内局部变量没有逃离方法的作用范围，它是线程安全的
  - 如果是局部变量引用了对象，并逃离方法的作用范围，需要考虑线程安全

异常：

- `StackOverFlowError` : 线程请求的栈深度超过最大值
- `OutOfMemoryError` : 栈帧过多导致栈内存溢出 （超过了栈的容量）

> 局部变量表存放了编译期可知的各种基本数据类型（boolean、byte、char、short、int、float、long、double）、对象引用和returnAddress类型，其中的数据没有初始化阶段，要想使用必须显示地赋初值。

## 本地方法栈

本地方法栈与虚拟机栈类似，它们之间的区别只是本地方法栈为本地方法服务

- 不需要进行GC，与虚拟机栈类似，也是线程私有的，有 StackOverFlowError 和 OutOfMemoryError 异常
- 虚拟机栈执行的是 Java 方法，在 HotSpot JVM 中，直接将本地方法栈和虚拟机栈合二为一
- 本地方法一般是由其他语言编写，并且被编译为基于本机硬件和操作系统的程序
- 当某个线程调用一个本地方法时，就进入了不再受虚拟机限制的世界，和虚拟机拥有同样的权限
  - 本地方法可以通过本地方法接口来**访问虚拟机内部的运行时数据区**
  - 直接从本地内存的堆中分配任意数量的内存
  - 可以直接使用本地处理器中的寄存器

## 程序计数器

程序计数器（寄存器）主要有两个作用：

1. 解释器通过改变程序计数器来依次读取指令，从而实现代码的流程控制
2. 在多线程的情况下，程序计数器用于记录当前线程执行的位置，从而保证线程切换后能恢复到正确的执行位置

特点：
- 是线程私有的
- 是 JVM 规范中唯一一个**不出现 OOM 的区域**，所以这个空间不会进行 GC

程序计数器的生命周期随着线程的创建而创建，随着线程的结束而死亡

## 堆

几乎所有对象及数组都在这里分配内存，是垃圾收集的主要区域（"GC 堆"），虚拟机启动时创建，由所有线程共享，堆中对象大部分都需要考虑线程安全的问题
存放哪些资源：
- 对象实例：类初始化生成的对象，**基本数据类型的数组也是对象实例**，new 创建对象都使用堆内存
- 字符串常量池：
  - 字符串常量池原本存放于方法区，jdk7 开始放置于堆中
  - 字符串常量池**存储的是 String 对象的直接引用或者对象**，是一张 string table
- 静态变量：静态变量是有 static 修饰的变量，jdk7 时从方法区迁移至堆中
- 线程分配缓冲区 Thread Local Allocation Buffer：线程私有但不影响堆的共性，可以提升对象分配的效率

在 Java7 中堆内会存在**年轻代、老年代和方法区（永久代）**：

- Young 区被划分为三部分，Eden 区和两个大小严格相同的 Survivor 区。Survivor 区某一时刻只有其中一个是被使用的，另外一个留做垃圾回收时复制对象。在 Eden 区变满的时候， GC 就会将存活的对象移到空闲的 Survivor 区间中，根据 JVM 的策略，在经过几次垃圾回收后，仍然存活于 Survivor 的对象将被移动到 Tenured 区间
- Tenured 区主要保存生命周期长的对象，一般是一些老的对象，当一些对象在 Young 复制转移一定的次数以后，对象就会被转移到 Tenured 区
- Perm 代主要保存 Class、ClassLoader、静态变量、常量、编译后的代码，在 Java7 中堆内方法区会受到 GC 的管理

## 方法区

方法区：是各个线程共享的内存区域，用于存储已被虚拟机加载的类信息、常量、即时编译器编译后的代码等数据，虽然 Java 虚拟机规范把方法区描述为堆的一个逻辑部分，但是也叫 Non-Heap（非堆）

方法区是一个 JVM 规范，**永久代与本地内存的元空间都是其一种实现方式**

方法区的大小不必是固定的，可以动态扩展，加载的类太多，可能导致永久代内存溢出 (OutOfMemoryError)

方法区的 GC：针对常量池的回收及对类型的卸载，比较难实现

为了**避免方法区出现 OOM**，在 JDK8 中将堆内的方法区（永久代）移动到了本地内存上，重新开辟了一块空间，叫做元空间，元空间存储类的元信息，**静态变量和字符串常量池等放入堆中**

类元信息：在类编译期间放入方法区，存放了类的基本信息，包括类的方法、参数、接口以及常量池表

常量池表（Constant Pool Table）是 Class 文件的一部分，存储了**类在编译期间生成的字面量、符号引用**，JVM 为每个已加载的类维护一个常量池
- 字面量：基本数据类型、字符串类型常量、声明为 final 的常量值等
- 符号引用：类、字段、方法、接口等的符号引用

运行时常量池是方法区的一部分
- 常量池表（编译器生成的字面量和符号引用）中的数据会在类加载后放入运行时常量池
- 类在解析阶段将这些符号引用替换成直接引用
- 除了在编译期生成的常量，还允许动态生成，例如 String 类的 intern()

## 本地内存

本地内存：又叫做**堆外内存**，线程共享的区域，本地内存这块区域是不会受到 JVM 的控制的，不会发生 GC；因此对于整个 java 的执行效率是提升非常大，但是如果内存的占用超出物理内存的大小，同样也会报 OOM

本地内存概述图：

![JVM-内存图对比](JVM/JVM-内存图对比-43d46a97971a475ea72f08e65ff6a3d6.png)
***

### 元空间

永久代被元空间代替，永久代的**类信息、方法、常量池**等都移动到元空间区

元空间与永久代区别：元空间不在虚拟机中，使用的本地内存，默认情况下，元空间的大小仅受本地内存限制

方法区内存溢出：

* JDK1.8 以前会导致永久代内存溢出：java.lang.OutOfMemoryError: PerGen space

  ```sh
   -XX:MaxPermSize=8m		#参数设置
  ```

* JDK1.8 以后会导致元空间内存溢出：java.lang.OutOfMemoryError: Metaspace

  ```sh
  -XX:MaxMetaspaceSize=8m	#参数设置	
  ```


### 直接内存

直接内存是 Java 堆外、直接向系统申请的内存区间，不是虚拟机运行时数据区的一部分，在一些场景中显著提高性能，因为避免了在堆内存和堆外内存来回拷贝数据。不受jvm内存回收管理

# 内存分配

**堆空间的基本结构：**

![堆空间的基本结构.png](JVM/堆空间的基本结构-ea84ba5737524cec9020c64b4f35d79a.png)

## Minor GC 和 Full GC

1. Minor GC：回收新生代，因为新生代对象存活时间很短，因此 Minor GC 会频繁执行，执行的速度一般也会比较快
   **触发条件：**
   - 当 Eden 空间满时，就将触发一次 Minor GC
2. Full GC：回收老年代和新生代，老年代对象其存活时间长，因此 Full GC 很少执行，执行速度会比 Minor GC 慢很多
   **触发条件：**
   - 调用 System.gc()
     只是建议虚拟机执行 Full GC，但是虚拟机不一定真正去执行。不建议使用这种方式，而是让虚拟机管理内存
   - 老年代空间不足
     老年代空间不足的常见场景大对象直接进入老年代、长期存活的对象进入老年代等。为了避免以上原因引起的 Full GC，应当尽量不要创建过大的对象以及数组。
   - 空间分配担保失败
     使用复制算法的 Minor GC 需要老年代的内存空间作担保，如果担保失败会执行一次 Full GC

## 分配策略

### 1. 对象优先在 Eden 分配

大多数情况下，对象在新生代 Eden 上分配，当 Eden 空间不够时，发起 Minor GC。

### 2. 大对象直接进入老年代

大对象是指需要连续内存空间的对象，最典型的大对象是那种很长的字符串以及数组。

避免在 Eden 和 Survivor 之间的大量内存复制而降低效率。

### 3. 长期存活的对象进入老年代

为对象定义年龄计数器，对象在 Eden 出生并经过 Minor GC 依然存活，将移动到 Survivor 中，年龄就增加 1 岁，增加到一定年龄则移动到老年代中。

### 4. 动态对象年龄判定

虚拟机并不是永远要求对象的年龄必须达到 MaxTenuringThreshold 才能晋升老年代，如果在 Survivor 中相同年龄所有对象大小的总和大于 Survivor 空间的一半，则年龄大于或等于该年龄的对象可以直接进入老年代，无需等到 MaxTenuringThreshold 中要求的年龄。

### 5. 空间分配担保

空间分配担保是为了确保在 Minor GC 之前老年代本身还有容纳新生代所有对象的剩余空间。


# 垃圾回收

主要是针对对象内存的回收和对象内存的分配，垃圾收集最核心的功能是 **堆** 内存中对象的分配与回收。

## 判断一个对象是否可被回收

### ~~1. 引用计数算法~~

为对象添加一个引用计数器，当对象增加一个引用时计数器加 1，引用失效时计数器减 1。引用计数为 0 的对象可被回收。

**这个方法实现简单，效率高，但是目前主流的虚拟机中并没有选择这个算法来管理内存，其最主要的原因是它很难解决对象之间相互循环引用的问题。**

### 2. 可达性分析算法

以 GC Roots 作为起点，从这些节点开始向下搜索，节点所走过的路径称为引用链，当一个对象到 GC Roots 没有任何引用链相连的话，则**表示可以回收**

![可达性分析算法.png](JVM/可达性分析算法-3ae729d2de624472908fc65840ea65fb.png)

GC Roots（Set） 一般包含以下内容：

- 虚拟机栈（局部变量表）中引用的对象
- 本地方法栈中 JNI（Native方法） 引用的对象
- 方法区中类静态属性引用的对象
- 方法区中的常量引用的对象

## 引用类型

无论是通过引用计数算法判断对象的引用数量，还是通过可达性分析算法判断对象是否可达，判定对象是否可被回收都与引用有关。

Java 提供了四种强度不同的引用类型。

### 1. 强引用

当内存空间不足，Java 虚拟机宁愿抛出 OutOfMemoryError 错误，使程序异常终止，也不会靠随意回收具有强引用的对象来解决内存不足问题。

使用 new 一个新对象的方式来创建强引用，此时它储与可达状态，不可能被垃圾回收，即使该对象以后永远不会被用到。因此强引用是造成内存泄漏的主要原因之一

### 2. 软引用

被软引用关联的对象只有在内存不够的情况下才会被回收。

软引用可以和一个引用队列（ReferenceQueue）联合使用，如果软引用所引用的对象被垃圾回收，JAVA 虚拟机就会把这个软引用加入到与之关联的引用队列中。软引用通常用在对内存敏感的程序中，比如高速缓存

```java
public static void main(String[] args) {
final int _4M = 4*1024*1024;
        //使用引用队列，用于移除引用为空的软引用对象
        ReferenceQueue<byte[]> queue = new ReferenceQueue<>();
        //使用软引用对象 list和SoftReference是强引用，而SoftReference和byte数组则是软引用
        List<SoftReference<byte[]>> list = new ArrayList<>();
        SoftReference<byte[]> ref= new SoftReference<>(new byte[_4M]);

        //遍历引用队列，如果有元素，则移除
        Reference<? extends byte[]> poll = queue.poll();
        while(poll != null) {
        //引用队列不为空，则从集合中移除该元素
        list.remove(poll);
        //移动到引用队列中的下一个元素
        poll = queue.poll();
        }
}
	
```

### 3. 弱引用

只具有弱引用的对象拥有更短暂的生命周期。被弱引用关联的对象一定会被回收，也就是说它只能存活到下一次垃圾回收发生之前。

弱引用可以和一个引用队列（ReferenceQueue）联合使用，如果弱引用所引用的对象被垃圾回收，Java 虚拟机就会把这个弱引用加入到与之关联的引用队列中。

### 4. 虚引用

虚引用并不会决定对象的生命周期，也无法通过虚引用得到一个对象。如果一个对象仅持有虚引用，那么它就和没有任何引用一样，在任何时候都可能被垃圾回收，虚引用必须和引用队列联合使用

虚引用的唯一目的是能在这个对象被回收时收到一个系统通知，用来跟踪对象被垃圾回收的活动。

### 5. 终结器引用

所有的类都继承自Object类，Object类有一个finalize方法。当某个对象不再被其他的对象所引用时，会先将终结器引用对象放入引用队列中，然后根据终结器引用对象找到它所引用的对象，然后调用该对象的finalize方法。调用以后，该对象就可以被垃圾回收了

## 垃圾收集算法

### 1. 标记 - 清除

![标记清除](JVM/标记清除-d583c73e1a3e41debb823b417bc1aae7.png)

该算法分为“标记”和“清除”阶段：首先标记出所有不需要回收的对象，在标记完成后统一回收掉所有没有被标记的对象。它是最基础的收集算法，后续的算法都是对其不足进行改进得到。这种垃圾收集算法会带来两个明显的问题：

1. 效率问题
2. 空间问题（标记清除后会产生大量不连续的碎片）

### 2. 标记 - 复制

![标记复制](JVM/标记复制-e9558462686544318ee168b6e0ad018c.png)

将内存分为大小相同的两块，每次使用其中的一块。当这一块的内存使用完后，就将还存活的对象复制到另一块去，然后再把使用的空间一次清理掉。这样就使每次的内存回收都是对内存区间的一半进行回收。

### 3. 标记 - 整理

![标记整理](JVM/标记整理-27f56b62f84f45a7a33e3cccffb94141.png)

让所有存活的对象都向一端移动，然后直接清理掉端边界以外的内存。
优点:

- 不会产生内存碎片


不足:

- 需要移动大量对象，处理效率比较低。

### 4. 分代收集

当前虚拟机的垃圾收集都采用分代收集算法，它根据对象存活周期将内存划分为几块，不同块采用适当的收集算法。
一般将堆分为新生代和老年代。

- 新生代使用：标记 - 复制算法，只需要付出少量对象的复制成本就可以完成每次垃圾收集。
- 老年代使用：标记 - 清除 或者 标记 - 整理 算法


## 垃圾收集器

![垃圾收集器](JVM/垃圾收集器-656711ef1db540019b36d53b734595c9.jpg)
以上是 HotSpot 虚拟机中的 7 个垃圾收集器

- 单线程与多线程：单线程指的是垃圾收集器只使用一个线程，而多线程使用多个线程；
- 串行与并行：串行指的是垃圾收集器与用户程序交替执行，这意味着在执行垃圾收集的时候需要停顿用户程序；并行指的是垃圾收集器和用户程序同时执行。除了 **CMS 和 G1**之外，其它垃圾收集器都是以串行的方式执行。

### 1. Serial 收集器(串行)

**新生代采用标记-复制算法，老年代采用标记-整理算法。**
![Serial 收集器](JVM/Serial 收集器-1f8f34505fa741009fd7b3f57812437b.png)

它以串行的方式执行。
它是单线程的收集器，只会使用一个线程进行垃圾收集工作，会暂停所有的用户线程，不适合服务器环境。
它的优点是简单高效，在单个 CPU 环境下，由于没有线程交互的开销，因此拥有最高的单线程收集效率。

### 2. ParNew 收集器(并行)

**新生代采用标记-复制算法，老年代采用标记-整理算法。**
![ParNew 收集器](JVM/ParNew 收集器-e8510337c392420aa5b32be7e28fbd7b.png)

它是 Serial 收集器的多线程版本。
它是 Server 场景下默认的**新生代收集器**，除了性能原因外，主要是因为除了 Serial 收集器，只有它能与 CMS 收集器配合使用。
它是激活CMS后的默认新生代收集器。

### 3. ++Parallel Scavenge 收集器(吞吐量优先)++

**新生代采用标记-复制算法，老年代采用标记-整理算法。**
![Parallel收集器](JVM/Parallel收集器-e3c36ed9dffa40cb90c7b4189b4a81b2.png)

也是使用多线程收集器，是 JDK1.8 **默认收集器**
Parallel Scavenge 收集器关注点是吞吐量（高效率的利用 CPU），CMS 等垃圾收集器的关注点更多的是用户线程的停顿时间（提高用户体验）。Parallel Scavenge 收集器提供自适应调节策略（自适应调节策略：虚拟机会根据当前系统的运行情况收集性能监控信息，动态调整这些参数以达到最大利用），垃圾收集停顿时间是以牺牲吞吐量和新生代空间换取的。

---

### 4. ~~Serial Old 收集器(串行)~~

Serial 收集器的**老年代版本**，它同样是一个单线程收集器。它主要有两大用途：一种用途是在 JDK1.5 以及以前的版本中与 Parallel Scavenge 收集器搭配使用，另一种用途是作为 CMS 收集器的后备方案。

### 5. Parallel Old 收集器(吞吐量优先)

Parallel Scavenge 收集器的老年代版本。使用多线程和“标记-整理”算法。在注重吞吐量或者注重CPU资源的场合，都可以优先考虑 Parallel Scavenge 收集器和 Parallel Old 收集器。

### 6. *++CMS 收集器(并发)++*

![CMS 收集器](JVM/CMS 收集器-ce625f3c94114306bff7859daf2e3f2c.jpg)

CMS（Concurrent Mark Sweep），Mark Sweep 指的是标记 - 清除算法，一种以获取最短回收停顿时间为目标的老年代收集器。整个过程分为四个步骤：
- 初始标记：仅仅只是标记一下 GC Roots 能直接关联到的对象，速度很快，需要停顿。
- 并发标记：从 GC Roots 的直接关联对象开始遍历整个对象图的过程，在整个回收过程中耗时最长，不需要停顿。
- 重新标记：为了修正并发标记期间因用户程序继续运作而导致标记产生变动的那一部分对象的标记记录，需要停顿。
- 并发清除：清理删除掉判断死亡的对象，不需要停顿。

应用场景：适用于注重服务的响应速度，希望系统停顿时间最短，给用户带来更好的体验等场景下。

主要优点：并发收集、低停顿。但是它有下面三个明显的缺点：
- 对 CPU 资源敏感
- 无法处理浮动垃圾，有可能出现失败进而导致另一次完全stop the world的full gc产生；必须预留部分空间来保证并发时程序运行使用，若预留内存不够使用，则冻结用户线程并启用Serial Old 收集器
- 它使用的回收算法-“标记-清除”算法会导致收集结束时会有大量空间碎片产生。

由于并发进行，CMS在收集与应用线程会同时增加对堆内存的占用。也就是说，CMS必须在老年代堆内存用尽之前完成垃圾回收，否则会造成回收失败，此时将触发担保机制，Serial Old作为备用将会进行GC，从而造成较大停顿时间。

---

### 7. G1 收集器

是一款面向**服务端**的垃圾收集器，以极高概率满足 GC 停顿时间要求的同时,还具备高吞吐量性能特征，JDK 9以后默认使用，而且替代了CMS 收集器。**整体上是标记-整理算法，两个区域之间是标记-复制算法**，不会产生很多内存碎片

G1把连续的堆内存划分为多个大小相等的独立区域(Region)，每个Region都可以根据需要，扮演新生代的Eden空间、Survvivor空间、老年代空间。收集器对扮演不同角色的Region采用不同的策略处理。Region中还有一类特殊的Humongous区域，专门用来存储大对象。

- 并行与并发：G1 能充分利用 CPU、多核环境下的硬件优势，使用多个 CPU（CPU 或者 CPU 核心）来缩短 Stop-The-World 停顿时间。部分其他收集器原本需要停顿 Java 线程执行的 GC 动作，G1 收集器仍然可以通过并发的方式让 java 程序继续执行。
- 分代收集：虽然 G1 可以不需要其他收集器配合就能独立管理整个 GC 堆，但是还是保留了分代的概念。
- 空间整合：与 CMS 的“标记-清理”算法不同，G1 从整体来看是基于“标记-整理”算法实现的收集器；从局部上来看是基于“标记-复制”算法实现的。
- 可预测的停顿：这是 G1 相对于 CMS 的另一个大优势，降低停顿时间是 G1 和 CMS 共同的关注点，但 G1 除了追求低停顿外，还能建立可预测的停顿时间模型，能让使用者明确指定在一个长度为 M 毫秒的时间片段内。

G1 收集器的运作步骤：
- 初始标记：只标记GC Roots能直接关联到的对象，并修改TAMS指针的值，让下一阶段用户线程并发运行时能正确地在可用的Region中分配对象。此阶段需要停顿线程
- 并发标记：进行GC Roots Tracing的过程，耗时较长，但可与用户程序并发执行
- 最终标记：对用户线程做一个短暂停顿，用于处理并发标记期间因程序运行导致标记发生变化的那一部分对象
- 筛选回收：根据时间来进行价值最大化的回收，必须暂停用户线程以移动Region

**G1 收集器在后台维护了一个优先列表，每次根据允许的收集时间，优先选择回收价值最大的 Region** 。这种使用 Region 划分内存空间以及有优先级的区域回收方式，保证了 G1 收集器在有限时间内可以尽可能高的收集效率（把内存化整为零）。
![G1收集器运行示意图](JVM/G1收集器运行示意图-28b4d960958844c0a4ab2d1a971c4604.png)

可以由用户指定期望的停顿时间是G1收集器很强大的功能，设置不同的期望时间，可使得G1在不同场景下取得关注吞吐量和关注延迟之间的最佳平衡

与CMS相比有点：
1. G1不会产生内存碎片
2. 可以精确控制停顿。G1收集器把整个堆分成多个固定大小的区域，每次根据允许停顿额时间去收集垃圾最多的区域

与CMS相比缺点：
1. 内存占用高
2. 程序运行时的额外执行负载高

# 类的生命周期

![类的生命周期](JVM/类的生命周期-3f1c7c15ad654fdaa46aca7816fe76c5.png)

## 类加载过程

Java虚拟机把描述类的数据从Class文件加载到内存，并对数据进行校验、转换解析和初始化，最终形成可以被虚拟机直接使用的Java类型，这个过程被称作虚拟机的类加载机制。包含了加载、++验证、准备、解析++(统称为连接)和初始化这 5 个阶段。
### 1. 加载

加载过程完成以下三件事：

- 通过类的全限定名称获取定义该类的二进制字节流。
- 将该字节流表示的静态存储结构转换为方法区的运行时数据结构。
- **在内存中生成一个代表该类的`Class`对象**，作为方法区中该类各种数据的访问入口。

加载阶段完成后，二进制字节流就存储在方法区中了。
### 2. 链接
#### 2.1. 验证

确保 Class 文件的字节流中包含的信息符合当前虚拟机的要求，并且不会危害虚拟机自身的安全。

主要包括四种验证：文件格式验证、元数据验证、字节码验证、符号引用验证

#### 2.2. 准备

为类变量分配内存并设置类变量**初始值**的阶段，初始值一般为 0 值。不包括实例变量，类变量会分配在方法区（JDK8之后类变量都在堆中）中，而实例变量会在对象实例化时随着对象一起被分配在堆中。
`public static int value = 123;`
那么变量value在准备阶段后的初始值为0而不是123，因为这时候尚未开始执行任何Java方法，给value赋值123的动作存放于类构造器`<clinit>()`中，要到类的初始化阶段才会被执行。

此阶段不包含用final修饰的static，因为常量在编译阶段就分配了，准备阶段会显示初始化。
> static+final修饰的其他引用类型常量和final修饰的实例属性，在实例创建的时候才会赋值
#### 2.3. 解析

将常量池的符号引用转换为直接引用的过程。

### 3. 初始化

直到初始化阶段，Java虚拟机才真正开始执行类中编写的Java代码，将主导权移交给应用程序，此前的几个阶段除了在加载阶段用户应用程序可以通过自定义类加载器的方式局部参与外，其余动作都完全由Java虚拟机主导控制。

严格规定有且只有六种情况必须立即对类进行初始化：
1. 遇到new、getstatic、putstatic、invokestatic四条字节码指令
2. 对类型进行反射调用的时候
3. 当初始化类时，发现其父类还没有进行过初始化，则需要先初始化其父类
4. 当虚拟机启动时，用户需指定一个要执行的主类（包含main方法的那个类），虚拟机会先初始化这个类
5. JDK7中的动态语言支持
6. 被default修饰的接口方法的实现类发生初始化，那么该接口要在其之前初始化
> 一个接口在初始化时，并不要求其父接口全部完成初始化，只有在真正使用到父接口的时候（如引用接口中定义的常量）才会初始化

- 初始化阶段是执行类构造器方法`<clinit> ()`的过程。
此方法并不是程序员直接在代码中编写的方法，它是javac编译器自动**收集类中的所有类变量的赋值动作和静态代码块中的语句**合并而来，编译器收集的顺序是由语句在源文件中出现的顺序决定的。静态语句块中只能访问到定义在静态语句块之前的变量，定义在它之后的变量，在前面的静态语句块中可以赋值，但是不能访问。
```java
public class Test {
	static {
	   i = 0 ;//给变量赋值可以正常编译通过
	   System.out.print(i) ;//编译器会提示"非法向前引用"
	}
	static int i = 1;
}
```
- `<clinit> ()`不同于类的构造器（`<init> ()`），它不需要显示地调用父类构造器，虚拟机会保证子类的`<clinit> ()`执行前，父类的`<clinit> ()`已经执行完毕
- 由于父类的`<clinit> ()`方法先执行，意味着父类中定义的静态语句块要优于子类的变量赋值操作
- `<clinit> ()`方法并不是必须的，若一个类没有静态语句块，也没有对变量的赋值操作，那么就不会生成
- 执行接口的`<clinit> ()`方法不会先执行父接口的`<clinit> ()`，只有父接口中定义的变量被使用时，父接口才会被初始化。此外接口的实现类在初始化时也不会执行接口的`<clinit> ()`方法
- 虚拟机保证一个类的`<clinit> ()`方法在多线程下被同步加锁

> 类的初始化顺序：（静态变量、静态代码块：取决于它们在类中的出现顺序）->（普通成员变量、初始化代码块：取决于它们在类中的出现顺序）->构造器

## 类加载器
### 类与类加载器
即使两个类来源于同一个class文件，被同一个虚拟机加载，只要加载他们的类加载器不同，那这两个类就必定不同。

JVM只有两种类型的类加载器，分别为启动类加载器（BootstrapClassLoader）和自定义类加载器，除了 BootstrapClassLoader 其他类加载器均由 Java 实现且全部继承自`java.lang.ClassLoader`，Java虚拟机规范将所有派生于抽象类ClassLoader的类加载器都划分为自定义类加载器。
1. BootstrapClassLoader(启动类加载器) ：最顶层的加载类，由 C++实现，负责加载`%JAVA_HOME%/lib`目录下的 jar 包和类或者被`-Xbootclasspath`参数指定的路径中的所有类，用于提供JVM自身所需要的类。启动类加载器无法被Java程序直接引用
2. ExtensionClassLoader(扩展类加载器) ：主要负责加载`%JRE_HOME%/lib/ext`目录下的 jar 包和类，或被`java.ext.dirs`系统变量所指定的路径下的 jar 包。
3. AppClassLoader(应用程序类加载器) ：面向用户的加载器，负责加载当前应用`classpath`下的所有 jar 包和类。

### 双亲委派模型
双亲委派模型要求除了顶层的启动类加载器外，其余的类加载器都应有自己的父类加载器（一般不是以继承实现的）

Java虛拟机对class文件采用的是按需加载的方式，也就是说当需要使用该类时才会将它的class文件加载到内存生成class对象。每一个类都有一个对应它的类加载器。系统中的 ClassLoader 在协同工作的时候会默认使用 双亲委派模型 。即在类加载的时候，系统会首先判断当前类是否被加载过。已经被加载的类会直接返回，否则才会尝试加载。如果一个类加载器收到了类加载的请求，它首先不会自己区尝试加载这个类，而是把这个请求委派给父类加载器去完成，因此所有的请求最终都应该传送到顶层的启动类加载器`BootstrapClassLoader`中。只有当父类加载器无法处理时，才由自己来处理。当父类加载器为 null 时，会使用启动类加载器`BootstrapClassLoader`作为父类加载器。

![双亲委派模型](JVM/双亲委派模型-f36b2b1145bf47fc82502d6243dcbed4.png)

优势：
1. 保证了 Java 程序的稳定运行，可以避免类的重复加载（JVM 区分不同类的方式不仅仅根据类名，相同的类文件被不同的类加载器加载产生的是两个不同的类）
2. 保证了 Java 的核心 API 不被篡改。如果没有使用双亲委派模型，而是每个类加载器加载自己的话就会出现一些问题，如编写一个称为`java.lang.Object`类的话，那么程序运行的时候，系统就会出现多个不同的 Object 类。

# 内存模型

## JMM

JAVA内存模型（JMM）本身是一种抽象概念并不真实存在，它描述的是一组规范，通过这组规范定义了程序中各个变量的访问方式。

JMM关于同步的规定：

1. 线程解锁前，必须把共享变量的值刷新回主内存
2. 线程加锁前，必须读取主内存的最新值到自己的工作内存
3. 加锁解锁是同一把锁

JVM运行程序的实体是线程，而每个线程创建时JVM都会为其创建一个工作内存，工作内存是每个线程私有的数据区域，而JMM规定所有变量都存储在**主内存**，主内存是共享内存区域，所有线程都能访问，但<u>线程对变量的操作（读取赋值等）必须在工作内存中进行，首先要将变量从主内存拷贝到自己的工作内存，然后对变量进行操作，操作完成后再将变量写回主内存</u>，不能直接操作主内存中的变量，各个线程中的工作内存中存储着主内存中的*变量拷贝副本*，因此不同的线程间无法访问对方的工作内存，线程间的通信必须通过主内存来完成。
> 此时所说的主内存、工作内存于Java内存区域中的堆、栈、方法区并不是同一个层次的对内存的划分，两者几乎没有任何关系

JMM定义了一套在多线程读写共享数据时(成员变量,数组)时，对数据的可见性、有序性和原子性的规则和保障

- 原子性 - 保证指令不会受到线程上下文切换的影响
- 可见性 - 保证指令不会受 cpu 缓存的影响
- 有序性 - 保证指令不会受 cpu 指令并行优化的影响

### 原子性

两个线程对初始值为 0 的静态变量一个做自增，一个做自减，各做 5000 次，以上的结果可能是正数、负数、零。

对于 i++ 而言（i 为静态变量），实际会产生如下的 JVM 字节码指令：

```java
//自增
getstatic i
// 获取静态变量i的值
iconst_1
// 准备常量1
iadd
// 加法
putstatic i
// 将修改后的值存入静态变量i
```

多线程下自增自减可能交错运行，导致读取到错误的值

**解决方法**

- 使用**synchronized**关键字
- 使用`AtomicInteger`（原子数据类型，基于CAS实现）

### 可见性

main 线程对 run 变量的修改对于 t 线程不可见，导致了 t 线程无法停止

```java
static boolean run = true;
public static void main (String[]args) throws InterruptedException {
        Thread t = new Thread(() -> {
        while (run) {
        // ....
        }
        });
        t.start();
        Thread.sleep(1000);
        run = false;
        // 线程t不会如预想的停下来
        }
```

工作内存与主内存同步延迟现象造成了可见性问题

**解决方法**

- 使用**volatile**关键字
- 它可以用来修饰**成员变量**和**静态成员变量**（放在主存中的变量），可以避免线程从自己的工作缓存中查找变量的值，必须到主存中获取它的值，线程操作 volatile 变量都是**直接操作主存**

可见性保证的是在多个线程之间，一个线程对 volatile 变量的修改对另一 个线程可见， **不能保证原子性**，仅用在一个写线程，多个读线程的情况

> 注意 
> final可以保证可见性
> synchronized 语句块既可以保证代码块的原子性，也同时保证代码块内变量的可见性。但缺点是 synchronized是属于重量级操作，性能相对更低。

### 有序性

**指令重排**：JVM 会在**不影响正确性**的前提下，可以**调整**语句的执行**顺序**，**多线程下『指令重排』会影响正确性**。

**解决办法**：**volatile** 修饰的变量，可以**禁用**指令重排
- 禁止的是加volatile关键字变量之前的代码被重排序

## 先行发生原则
先行发生（happens-before）是Java内存模型中定义的**两项操作之间的偏序关系**，Java 内存模型具备一些先天的“有序性”，即不需要通过任何同步手段（volatile、synchronized 等）就能够得到保证的安全，这个通常也称为 happens-before 原则。它是判断数据是否存在竞争，线程是否安全的有效手段，如果两个操作之间缺乏happens-before关系，那么JVM可以对他们任意地重排序。

Java内存模型具备以下天然的先行发生关系：
- 程序次序规则：在一个线程内，按照**控制流**顺序，书写在前面的操作先行发生于书写在后面的操作
- 管程锁定规则：一个unlock操作先行发生于后面对同一个锁的lock操作
- volatile变量规则：对一个volatile变量的写操作先行发生于后面对这个变量的读操作
- 线程启动规则：Thread对象的start()方法先行发生于此线程的任何操作
- 线程终止规则：线程中的所有操作都先行发生于此线程的终止检测
- 线程中断规则：当一个线程在另一个线程上调用interrupt时，必须在被中断线程检测到interrupt调用之前执行
- 对象终结规则：一个对象的初始化完成先行发生于它的finalize()方法的开始
- 传递性：如果操作A在操作B之前执行，并且操作B在操作C之前执行，那么操作A必须在操作C之前执行


## CAS与volatile
### CAS
CAS 即 Compare and Swap ，它体现的一种**乐观锁**的思想，乐观并发需要硬件保证某些语义上看需要多次操作的行为可以通过一条处理器指令就能完成。它的功能是判断内存某个位置的值是否为预期值，如果是则更新，这个过程是原子的。Java中调用Unsafe类中的CAS方法，JVM会实现出原子操作，达到数据一致。

比如多个线程要对一个共享的整型变量执行 +1 操作：
```java
// 需要不断尝试
while(true) {
	int 旧值 = 共享变量 ; // 比如拿到了当前值 0
	int 结果 = 旧值 + 1; // 在旧值 0 的基础上增加 1 ，正确结果是 1
    /*
    这时候如果别的线程把共享变量改成了 5，本线程的正确结果 1 就作废了，这时候
    compareAndSwap 返回 false，重新尝试，直到：
    compareAndSwap 返回 true，表示我本线程做修改的同时，别的线程没有干扰
    */
	if( compareAndSwap ( 旧值, 结果 )) {
	// 成功，退出循环
	}
}
```

结合 CAS 和 volatile 可以实现**无锁并发**，适用于**线程数少、多核 CPU** 的场景下。

- synchronized 是基于**悲观锁**的思想
- CAS 是基于**乐观锁**的思想，体现的是无锁并发、无阻塞并发，缺点：
  - 因为没有使用 synchronized，所以线程不会陷入阻塞，这是效率提升的因素之一，但如果竞争激烈，重试必然频繁发生，可能给cpu带来很大的开销
  - 只能保证一个共享变量的原子操作
  - 引出ABA问题

### volatile原理
> “观察加入volatile关键字和没有加入volatile关键字时所生成的汇编代码发现，加入volatile关键字时，会多出一个lock前缀指令”

lock前缀指令实际上相当于一个内存屏障，内存屏障会提供3个功能：

- 它确保指令重排序时不会把其后面的指令排到内存屏障之前的位置，也不会把前面的指令排到内存屏障的后面
- 它会强制将对缓存的修改操作立即写入主存；
- 如果是写操作，它会导致其他CPU中对应的缓存行无效。

### Unsafe类

1. unsafe是CAS的核心类，由于Java方法无法直接访问底层系统，需要通过本地方法来访问，基于该类可以直接操作特定内存的数据。

```java
public class AtomicInteger extends Number implements java.io.Serializable {
    private static final long serialVersionUID = 6214790243416807050L;

    // setup to use Unsafe.compareAndSwapInt for updates
    private static final Unsafe unsafe = Unsafe.getUnsafe();
    private static final long valueOffset;

    static {
        try {
            valueOffset = unsafe.objectFieldOffset
                (AtomicInteger.class.getDeclaredField("value"));
        } catch (Exception ex) { throw new Error(ex); }
    }
    private volatile int value;
    
    public final int getAndIncrement() {
        //this指的是对象本身
        return unsafe.getAndAddInt(this, valueOffset, 1);
    }
}
```

2. 变量valueOffset表示该变量值在内存中的偏移地址，因为unsafe就是根据内存偏移地址获取数据的。
3. 变量value用volatile修饰，保证了多线程间的内存可见性。

### ABA问题

CAS会导致ABA问题。CAS算法实现的一个前提是需要取出内存中某时刻的数据并在当下时刻比较并替换，那么在这个时间差会发生数据的变化。

> 如线程1从内存位置V取出数据A，此时线程2也从内存取出数据A，由于线程2运行速度快，它将A变成B又变成A，这时候线程1进行CAS操作发现内存中仍然是A，然后线程1操作成功。虽然线程1CAS操作成功，但是这个过程是存在问题的。

ABA解决：`AtomicStampedReference`（原子引用+时间戳）

> 目前来说这个类处于相当鸡肋的位置，大部分情况下ABA问题不会影响程序并发的正确性，如果需要解决ABA问题，改用传统的互斥同步可能比原子类更加高效
-- 深入理解Java虚拟机

```java
class user {
    String name;
    int age;

    public user(String name, int age) {
        this.name = name;
        this.age = age;
    }
}
//普通原子引用类
public class AtomicDemo {
    public static void main(String[] args) {
        user user1 = new user("小米", 11);
        user user2 = new user("华为", 22);
        AtomicReference<user> userAtomicReference = new AtomicReference<>();
        userAtomicReference.set(user1);
        System.out.println(userAtomicReference.compareAndSet(user1, user2));
    }
}
```

`AtomicStampedReference`使用详解

```java
public class AtomicDemo {
    static AtomicReference<String> atomicReference = new AtomicReference<>("A");
    static AtomicStampedReference<String> atomicStampedReference = new AtomicStampedReference("A", 1);

    public static void main(String[] args) {
//        System.out.println("===============ABA问题产生============");
//        new Thread(() -> {
//            atomicReference.compareAndSet("A", "B");
//            atomicReference.compareAndSet("B", "A");
//        }, "线程1").start();
//
//        new Thread(() -> {
//            try {
//                TimeUnit.SECONDS.sleep(1);
//            } catch (InterruptedException e) {
//                e.printStackTrace();
//            }
//            System.out.println(atomicReference.compareAndSet("A", "B") + "\t" + atomicReference.get());
//        }, "线程2").start();

        System.out.println("===============ABA问题解决============");
        new Thread(() -> {
            System.out.println(Thread.currentThread().getName()+"\t第1次版本号"+atomicStampedReference.getStamp());
            atomicStampedReference.compareAndSet("A", "B",atomicStampedReference.getStamp(),atomicStampedReference.getStamp()+1);
            try {
                TimeUnit.SECONDS.sleep(2);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName()+"\t第2次版本号"+atomicStampedReference.getStamp());
            atomicStampedReference.compareAndSet("B", "A",atomicStampedReference.getStamp(),atomicStampedReference.getStamp()+1);
            System.out.println(Thread.currentThread().getName()+"\t第3次版本号"+atomicStampedReference.getStamp());
        }, "线程3").start();

        new Thread(() -> {
            int stamp = atomicStampedReference.getStamp();
            try {
                TimeUnit.SECONDS.sleep(4);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName()+"\t第1次版本号"+stamp);
            System.out.println(atomicStampedReference.compareAndSet("A", "B", stamp, stamp + 1));
            System.out.println(Thread.currentThread().getName()+"\t第2次版本号"+atomicStampedReference.getStamp());
        }, "线程4").start();
    }
}
```

## 原子操作类

juc（java.util.concurrent）中提供了原子操作类，可以提供线程安全的操作，例如：AtomicInteger、 AtomicBoolean等，它们底层就是采用 CAS 技术 + volatile 来实现的。

```java
 AtomicInteger i = new AtomicInteger(0);
 
// 获取并自增（i = 0, 结果 i = 1, 返回 0），类似于 i++ 
System.out.println(i.getAndIncrement());
 
// 自增并获取（i = 1, 结果 i = 2, 返回 2），类似于 ++i 
System.out.println(i.incrementAndGet());
 
// 自减并获取（i = 2, 结果 i = 1, 返回 1），类似于 --i 
System.out.println(i.decrementAndGet());
 
// 获取并自减（i = 1, 结果 i = 0, 返回 1），类似于 i--
System.out.println(i.getAndDecrement());
 
// 获取并加值（i = 0, 结果 i = 5, 返回 0） 
System.out.println(i.getAndAdd(5));
 
// 加值并获取（i = 5, 结果 i = 0, 返回 0） 
System.out.println(i.addAndGet(-5));
 
// 获取并更新（i = 0, p 为 i 的当前值, 结果 i = -2, 返回 0） 
// 其中函数中的操作能保证原子，但函数需要无副作用 
System.out.println(i.getAndUpdate(p -> p - 2));
 
// 更新并获取（i = -2, p 为 i 的当前值, 结果 i = 0, 返回 0）
// 其中函数中的操作能保证原子，但函数需要无副作用 
System.out.println(i.updateAndGet(p -> p + 2));
 
// 获取并计算（i = 0, p 为 i 的当前值, x 为参数1, 结果 i = 10, 返回 0） 
// 其中函数中的操作能保证原子，但函数需要无副作用 // getAndUpdate 如果在 lambda 中引用了外部的局部变量，要保证该局部变量是 final 的 
// getAndAccumulate 可以通过 参数1 来引用外部的局部变量，但因为其不在 lambda 中因此不必是 
final System.out.println(i.getAndAccumulate(10, (p, x) -> p + x));
 
// 计算并获取（i = 10, p 为 i 的当前值, x 为参数1, 结果 i = 0, 返回 0） 
// 其中函数中的操作能保证原子，但函数需要无副作用
System.out.println(i.accumulateAndGet(-10, (p, x) -> p + x));
```
# 锁优化
## synchronized优化
synchronized 代码块是由一对儿 monitorenter/monitorexit 指令实现的，Monitor 对象是同步的基本实现单元。

在 Java 6 之前，Monitor 的实现完全是依靠操作系统内部的互斥锁，因为需要进行用户态到内核态的切换，所以同步操作是一个无差别的重量级操作。

现代的（Oracle）JDK 中，JVM 提供了三种不同的 Monitor 实现，也就是常说的三种不同的锁：偏斜锁（Biased Locking）、轻量级锁和重量级锁，大大改进了其性能。

构造方法不能使用 synchronized 关键字修饰，因为**构造方法就是线程安全的**

锁可以升级, 但不能降级. 即: 无锁 -> 偏向锁 -> 轻量级锁 -> 重量级锁
![锁升级](JVM/锁升级.png)

### 自旋锁与自适应自旋
重量级锁竞争时，尝试获取锁的线程不会立即阻塞，可以使用自旋（默认 10 次）来进行优化，让后面请求锁的那个线程执行一个忙循环，不放弃处理器的执行时间，看看持有锁的线程是否很快会释放锁，这就是自旋锁。自旋锁不能代替阻塞，自旋需要占用处理器时间，因此自旋等待的时间必须有由一定限度。

优点：不会进入阻塞状态，减少线程上下文切换的消耗

缺点：当自旋的线程越来越多时，会不断的消耗 CPU 资源

在 Java 6 之后自旋锁是自适应的，比如在同一个对象上，刚刚的一次自旋操作成功过并且持有锁的线程正在运行中，那么认为这次自旋成功的可能性会高，就多自旋几次；反之，就少自旋甚至不自旋。
```java
   //手写自旋锁
   public class SpinLockDemo {
       AtomicReference<Thread> atomicReference = new AtomicReference<>();
   
       public void myLock() {
           Thread thread = Thread.currentThread();
           System.out.println(thread.getName() + "myLock");
           while (!atomicReference.compareAndSet(null, thread)) {
   
           }
       }
   
       public void myUnLock() {
           Thread thread = Thread.currentThread();
           System.out.println(thread.getName() + "myUnLock");
           while (!atomicReference.compareAndSet(thread, null)) {
   
           }
       }
   
       public static void main(String[] args) throws Exception {
           SpinLockDemo spinLockDemo = new SpinLockDemo();
           new Thread(() -> {
               spinLockDemo.myLock();
               try {
                   TimeUnit.SECONDS.sleep(3);
               } catch (InterruptedException e) {
                   e.printStackTrace();
               }
               spinLockDemo.myUnLock();
           }, "AA").start();
   
           TimeUnit.SECONDS.sleep(1);
           new Thread(() -> {
               spinLockDemo.myLock();
               try {
                   TimeUnit.SECONDS.sleep(3);
               } catch (InterruptedException e) {
                   e.printStackTrace();
               }
               spinLockDemo.myUnLock();
           }, "BB").start();
       }
   }
```

### 偏向锁
当没有竞争出现时，默认会使用偏斜锁。JVM 会利用 CAS 操作（compare and swap），在对象头上的 Mark Word 部分设置线程 ID，以表示这个对象偏向于当前线程，所以并不涉及真正的互斥锁。偏向锁的思想是偏向于让第一个获取锁对象的线程，这个线程之后重新获取该锁不再需要同步操作：
1. 当锁对象第一次被线程获取时，虚拟机把对象头中的标志位设置为"01"、把偏向模式设置为"1"，表示进入偏向模式，同时使用 CAS 把获取到这个锁的线程ID记录在对象的 MarkWord 中，如果成功，则获取偏向锁成功；如果失败，则进行锁升级。
2. 偏向锁标志是已偏向状态，MarkWord 中的线程 ID 是自己的线程 ID，则成功获取锁；MarkWord 中的线程 ID 不是自己的线程 ID，需要进行锁升级

> 大多数时候是不存在锁竞争的，常常是一个线程多次获得同一个锁，因此如果每次都要竞争锁会增大很多没有必要付出的代价，为了降低获取锁的代价，才引入的偏向锁。如果在重入到一定阈值之后仍然没有任何线程抢占行为发生，JVM就会省略CAS这个操作，以后只要不发生竞争，这个对象就归该线程所有。

当对象进入偏向状态时，MarkWord大部分的空间都用于存储持有锁的线程ID，占用了原有存储对象哈希码的位置。因此，当一个对象已经计算过一致性哈希码后，他就再也无法进入偏向锁状态了；而当一个对象正处于偏向锁状态，又收到一条计算哈希的指令，它的偏向状态就会立即撤销，并且膨胀为重量级锁。

以下几种情况会使对象的偏向锁失效:
- 调用对象的hashCode方法
- 多个线程使用该对象
- **调用了wait/notify方法**（调用wait方法会导致锁膨胀而使用**重量级锁**）

如果程序中大多数的锁都是被多个不同的线程访问，那么偏向模式就是多余的，此时禁用偏向锁反而可能会提升性能

### 轻量级锁
**轻量级锁使用场景**：一个对象有多个线程要加锁，但加锁的时间是错开的（没有竞争），可以使用轻量级锁来优化，轻量级锁对使用者是透明的（不可见）。轻量级锁是在无竞争的情况下使用CAS操作去消除同步使用的互斥量

如果有另外的线程试图锁定某个已经被偏斜过的对象，JVM 就需要撤销（revoke）偏斜锁，并切换到轻量级锁实现。轻量级锁依赖 CAS 操作 Mark Word 来试图获取锁，如果重试成功，就使用普通的轻量级锁；否则，进一步升级为重量级锁。
**加锁**：
- 首先在当前线程的栈帧中建立一个**锁记录**（Lock Record）对象，用来存储锁定对象的Mark Word拷贝（不再一开始就使用Monitor）
  ![轻量级锁1](JVM/轻量级锁1-374c1fcc15a54e76aea62433c10d3fda.png)
- 然后虚拟机使用CAS操作尝试把对象的Mark Word更新为指向Lock Record的指针
  ![轻量级锁2](JVM/轻量级锁2-30e70af2014649a9b688f1b9e38610e9.png)
- 如果cas替换成功，则将对象Mark Word的锁标志位转变为**00（轻量级锁状态）**，并由该线程给对象加锁
  ![轻量级锁3](JVM/轻量级锁3-6df823326fcb4416a759b0fe639423ce.png)
- 如果更换失败，那就意味着至少存在一条线程与当前线程竞争获取该对象的锁

有线程A和线程B来竞争对象c的锁(如: synchronized(c){}), 这时线程A和线程B同时将对象c的MarkWord复制到自己的锁记录中, 两者竞争去获取锁, 假设线程A成功获取锁, 并将对象c的对象头中的线程ID(MarkWord中)修改为指向自己的锁记录的指针, 这时线程B仍旧通过CAS去获取对象c的锁, 因为对象c的MarkWord中的内容已经被线程A改了, 所以获取失败. 此时为了提高获取锁的效率, 线程B便尝试使用自旋来获取锁, 这个循环是有次数限制的, 如果在循环结束之前CAS操作成功, 那么线程B就获取到锁, 如果循环结束依然获取不到锁, 则获取锁失败, 对象c的MarkWord中的记录会被修改为重量级锁, 然后线程B就会被挂起, 之后有线程C来获取锁时, 看到对象c的MarkWord中的是重量级锁的指针, 说明竞争激烈, 直接挂起

**解锁**：
它的解锁过程同样是通过CAS操作的，如果对象的Mark Word仍然指向线程的所记录，那就要CAS操作把对象当前的Mark Word和线程中复制的Mark Word替换回来。假如能够替换成功，那整个同步过程就顺利完成了；如果替换失败，则说明有其他线程尝试过获取锁，就要在释放锁的同时，唤醒被挂起的线程。

### 重量级锁
重量级锁是使用操作系统互斥量（mutex）来实现的传统锁。当所有对锁的优化都失效时，将退回到重量级锁。它与轻量级锁不同，竞争的线程不再通过自旋来竞争线程，而是直接进入堵塞状态，此时不消耗CPU，然后等拥有锁的线程释放锁后，唤醒堵塞的线程，然后线程再次竞争锁。

在重量级锁的实现中，对象头指向了重量级锁的位置，代表重量级锁的ObjectMonitor类里有字段可以记录非加锁状态下的MarkWord，其中自然可以存储原来的哈希码。

| 锁       | 优点    | 缺点      | 适用场景    |
| ------- | ------- | --------- | ---------- |
| 偏向锁   | 加锁和解锁不需要额外的消耗, 和执行非同步代码方法的性能相差无几 | 如果线程间存在锁竞争, 会带来额外的锁撤销的消耗. | 适用于只有一个线程访问的同步场景   |
| 轻量级锁 | 竞争的线程不会阻塞, 提高了程序的响应速度 | 如果始终得不到锁竞争的线程, 使用自旋会消耗CPU   | 追求响应时间, 同步快执行速度非常快 |
| 重量级锁 | 线程竞争不适用自旋, 不会消耗CPU  | 线程堵塞, 响应时间缓慢 | 追求吞吐量, 同步快执行时间速度较长 |

### 其他优化

1.  减少上锁时间
    同步代码块中尽量短

2. 减少锁的粒度
   将一个锁拆分为多个锁提高并发度，例如：ConcurrentHashMap

3. 锁粗化
对相同对象多次加锁，导致线程发生多次重入，频繁的加锁操作就会导致性能损耗，可以使用锁粗化方式优化

如果虚拟机探测到一串的操作都对同一个对象加锁，将会把加锁的范围扩展（粗化）到整个操作序列的外部

```java
   new StringBuffer().append("a").append("b").append("c");
```

4. 锁消除
   JVM 会进行代码的逃逸分析，例如某个加锁对象是方法内局部变量，不会被其它线程所访问到，这时候就会被即时编译器忽略掉所有同步操作。

5.  读写分离
    1. CopyOnWriteArrayList 
    2. ConyOnWriteSet

## 读写锁

独占锁：指该锁一次只能被一个线程所持有。`synchronized`和`ReentrantLock`都是独占锁

共享锁：值该锁可被多个线程锁持有

`ReentrantReadWriteLock`其读锁是共享锁，写锁是独占锁

读锁的共享锁保证并发读是高效的，读写、写读、写写的过程是互斥的

## synchronized和volatile的区别

1. `volatile`关键字是线程同步的轻量级实现，所以`volatile`性能肯定比`synchronized`关键字要好。但是`volatile`关键字只能用于变量而 `synchronized`关键字可以修饰方法以及代码块。
2. `volatile`关键字能保证数据的可见性，但不能保证数据的原子性。`synchronized`关键字两者都能保证。
3. `volatile`关键字主要用于解决变量在多个线程之间的可见性，而 `synchronized`关键字解决的是多个线程之间访问资源的同步性。