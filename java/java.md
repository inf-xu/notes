## 一、Java 内存区域

JVM 内存区域主要分为线程私有区域【程序计数器、虚拟机栈、本地方法栈】、线程共享区域【JAVA 堆、方法区】、直接内存。

1. 程序计数器（Program Counter Register）：是一块较小的内存区域，**记录当前线程所执行的字节码地址**。
2. Java 虚拟机栈（Java Virtual Machine Stacks）： 每个 Java 方法在执行时会创建一个栈帧（Stack Frame），**用于存储局部变量表、操作数栈、动态链接、方法出口等信息**。每个线程都有自己的 Java 虚拟机栈，它们的生命周期与线程相同。
3. 本地方法栈（Native Method Stack）：与 Java 虚拟机栈类似，用于存储本地方法的信息。
4. Java堆（Java Heap）：是 Java 虚拟机管理的最大的一块内存空间，用于**存储对象实例和数组等数据**。Java 堆是所有线程共享的，也是**垃圾回收的主要区域**。
5. 方法区（Method Area）：用于存储已被虚拟机加载的**类信息、常量池、静态变量**等数据。



### 1. 运行时数据区域

JDK 1.8 和之前的版本略有不同，我们这里以 JDK 1.7 和 JDK 1.8 这两个版本为例介绍。

<img src="D:\Data\笔记\java\java.assets\java-runtime-data-areas-jdk1.7.png" alt="Java 运行时数据区域（JDK1.7）" style="zoom:67%;" />

<img src="https://oss.javaguide.cn/github/javaguide/java/jvm/java-runtime-data-areas-jdk1.8.png" alt="Java 运行时数据区域（JDK1.8 ）" style="zoom: 67%;" />



#### （1）程序计数器

程序计数器是一块较小的内存空间，可以看作是当前线程所执行的字节码的行号指示器。**字节码解释器工作时通过改变这个计数器的值来选取下一条需要执行的字节码指令**，分支、循环、跳转、异常处理、线程恢复等功能都需要依赖这个计数器来完成。

另外，为了线程切换后能恢复到正确的执行位置，每条线程都需要有一个独立的程序计数器，各线程之间计数器互不影响，独立存储，我们称这类内存区域为“线程私有”的内存。

程序计数器主要有两个作用：

- 字节码解释器通过改变程序计数器来依次读取指令，从而实现代码的流程控制，如：顺序执行、选择、循环、异常处理。

- 在多线程的情况下，程序计数器用于记录当前线程执行的位置，从而当线程被切换回来的时候能够知道该线程上次运行到哪儿了。

>⚠️注意：
>
>程序计数器是唯一一个不会出现 `OutOfMemoryError` 的内存区域，**它的生命周期随着线程的创建而创建，随着线程的结束而死亡。**

#### （2）Java 虚拟机栈

与程序计数器一样，Java 虚拟机栈（后文简称栈）也是线程私有的，它的生命周期和线程相同，随着线程的创建而创建，随着线程的死亡而死亡。

栈绝对算的上是 JVM 运行时数据区域的一个核心，除了一些 Native 方法调用是通过本地方法栈实现的，其他所有的 Java 方法调用都是通过栈来实现的（也需要和其他运行时数据区域比如程序计数器配合）。

> 🎓补充知识：
>
> 在 Java 中，Native 方法是指在 Java 代码中声明但实现在本地代码中的方法。它们通常用于访问操作系统或硬件的功能，或者在性能和功能方面需要比纯 Java 代码更高级别的控制时使用。
>
> 要使用Native方法，需要编写本地代码并将其编译为与目标平台兼容的动态链接库（DLL）。然后，在 Java 代码中使用 System.loadLibrary() 方法加载该库，并使用 native 关键字声明 Native 方法。在运行时，Java 虚拟机将在需要时加载本地代码，使得 Java 应用程序可以调用 Native 方法。
>
> 通常情况下，使用 Native 方法有以下几种情况：
>
> 1. 访问操作系统或硬件的功能：如果需要访问底层的操作系统或硬件的功能，例如文件系统、网络接口或其他外设，则可以使用 Native 方法。
> 2. 性能要求高：Java 代码可以提供很好的跨平台兼容性和安全性，但是在某些情况下，Native 方法可能比  Java 代码更加高效。例如，对于大量的数学计算或图像处理等密集型任务，使用本地代码实现可能会提高性能。
> 3. 使用第三方库：如果希望在 Java 应用程序中使用 C 或 C++ 编写的第三方库，则可以使用 Native 方法来调用该库。
>
> 需要注意的是，使用Native方法存在一些安全风险，因为它们可以访问本地计算机的资源和内存。因此，必须小心使用Native方法，确保它们不会被恶意利用。

方法调用的数据需要通过栈进行传递，每一次方法调用都会有一个对应的栈帧被压入栈中，每一个方法调用结束后，都会有一个栈帧被弹出。

栈由一个个栈帧组成，而**每个栈帧中都拥有：局部变量表、操作数栈、动态链接、方法返回地址**。和数据结构上的栈类似，两者都是先进后出的数据结构，只支持出栈和入栈两种操作。

<img src="D:\Data\笔记\java\java.assets\image-20230426162224178.png" alt="image-20230426162224178" style="zoom:67%;" />

**局部变量表**

主要存放了编译期可知的各种数据类型（boolean、byte、char、short、int、float、long、double）、对象引用（reference 类型，它不同于对象本身，可能是一个指向对象起始地址的引用指针，也可能是指向一个代表对象的句柄或其他与此对象相关的位置）。

**操作数栈**

主要作为方法调用的中转站使用，用于存放方法执行过程中产生的中间计算结果。另外，计算过程中产生的临时变量也会放在操作数栈中。

**动态链接**

主要服务一个方法需要调用其他方法的场景。Class 文件的常量池里保存有大量的符号引用比如方法引用的符号引用。当一个方法要调用其他方法，需要将常量池中指向方法的符号引用转化为其在内存地址中的直接引用。动态链接的作用就是为了将符号引用转换为调用方法的直接引用，这个过程也被称为 **动态连接** 。

![img](D:\Data\笔记\java\java.assets\jvmimage-20220331175738692.png)

栈空间虽然不是无限的，但一般正常调用的情况下是不会出现问题的。不过，如果函数调用陷入无限循环的话，就会导致栈中被压入太多栈帧而占用太多空间，导致栈空间过深。那么当线程请求栈的深度超过当前 Java 虚拟机栈的最大深度的时候，就抛出 `StackOverFlowError` 错误。

Java 方法有两种返回方式，一种是 return 语句正常返回，一种是抛出异常。不管哪种返回方式，都会导致栈帧被弹出。也就是说， **栈帧随着方法调用而创建，随着方法结束而销毁。无论方法正常完成还是异常完成都算作方法结束。**

除了 `StackOverFlowError` 错误之外，栈还可能会出现`OutOfMemoryError`错误，这是因为如果栈的内存大小可以动态扩展， 如果虚拟机在动态扩展栈时无法申请到足够的内存空间，则抛出`OutOfMemoryError`异常。

程序运行中栈可能会出现两种错误：

- **`StackOverFlowError`：** 若栈的内存大小不允许动态扩展，那么当线程请求栈的深度超过当前 Java 虚拟机栈的最大深度的时候，就抛出 `StackOverFlowError` 错误。
- **`OutOfMemoryError`：** 如果栈的内存大小可以动态扩展， 如果虚拟机在动态扩展栈时无法申请到足够的内存空间，则抛出`OutOfMemoryError`异常。

#### （3）本地方法栈

和虚拟机栈所发挥的作用非常相似，区别是： **虚拟机栈为虚拟机执行 Java 方法 （也就是字节码）服务，而本地方法栈则为虚拟机使用到的 Native 方法服务。** 在 HotSpot 虚拟机中和 Java 虚拟机栈合二为一。

>🎓补充知识：
>
>HotSpot 虚拟机是一款由 Oracle 公司开发的 Java 虚拟机。它实现了 Java 虚拟机规范中定义的所有功能，并提供了高性能、可扩展性和可移植性。HotSpot 虚拟机采用即时编译技术，在运行时将 Java 字节码转换成本地机器代码，从而提高应用程序的执行速度。
>
>HotSpot 虚拟机还具有优秀的垃圾回收机制，可以自动管理内存，避免了程序员手动进行内存管理的问题。除此之外，HotSpot 虚拟机还支持 Java 平台调试体系结构（JPDA），包括 Java 调试器接口（JDI）和 Java 虚拟机调试接口（JVMTI），使得开发者可以方便地进行调试操作。
>
>总的来说，HotSpot 虚拟机的出现为 Java 语言的开发带来了很多好处，让 Java 程序能够在不同的硬件平台上高效运行，同时也为 Java 开发者提供了更好的开发和调试工具。

本地方法被执行的时候，在本地方法栈也会创建一个栈帧，用于存放该本地方法的局部变量表、操作数栈、动态链接、出口信息。

方法执行完毕后相应的栈帧也会出栈并释放内存空间，也会出现 `StackOverFlowError` 和 `OutOfMemoryError` 两种错误。

#### （4）堆

Java 虚拟机所管理的内存中最大的一块，Java 堆是所有线程共享的一块内存区域，在虚拟机启动时创建。**此内存区域的唯一目的就是存放对象实例，几乎所有的对象实例以及数组都在这里分配内存。**

Java 世界中“几乎”所有的对象都在堆中分配，但是，随着 JIT 编译器的发展与逃逸分析技术逐渐成熟，栈上分配、标量替换优化技术将会导致一些微妙的变化，所有的对象都分配到堆上也渐渐变得不那么“绝对”了。从 JDK 1.7 开始已经默认开启逃逸分析，如果某些方法中的对象引用没有被返回或者未被外面使用（也就是未逃逸出去），那么对象可以直接在栈上分配内存。

Java 堆是垃圾收集器管理的主要区域，因此也被称作 **GC 堆（Garbage Collected Heap）**。从垃圾回收的角度，由于现在收集器基本都采用分代垃圾收集算法，所以 Java 堆还可以细分为：新生代和老年代；再细致一点有：Eden、Survivor、Old 等空间。进一步划分的目的是更好地回收内存，或者更快地分配内存。

在 JDK 7 版本及之前，堆内存被通常分为下面三部分：

1. 新生代内存（Young Generation）
2. 老生代（Old Generation）
3. 永久代（Permanent Generation）

下图所示的 Eden 区、两个 Survivor 区 S0 和 S1 都属于新生代，中间一层属于老年代，最下面一层属于永久代。

![堆内存结构](D:\Data\笔记\java\java.assets\hotspot-heap-structure.png)

**JDK 8 版本之后 PermGen（永久代） 已被 Metaspace（元空间）取代，元空间使用的是本地内存。**。

大部分情况，对象都会首先在 Eden 区域分配，在一次新生代垃圾回收后，如果对象还存活，则会进入 S0 或者 S1，并且对象的年龄还会加 1（Eden 区->Survivor 区后对象的初始年龄变为 1），当它的年龄增加到一定程度（默认为 15 岁），就会被晋升到老年代中。对象晋升到老年代的年龄阈值，可以通过参数 `-XX:MaxTenuringThreshold` 来设置。

堆这里最容易出现的就是 `OutOfMemoryError` 错误，并且出现这种错误之后的表现形式还会有几种，比如：

1. **`java.lang.OutOfMemoryError: GC Overhead Limit Exceeded`** ： 当 JVM 花太多时间执行垃圾回收并且只能回收很少的堆空间时，就会发生此错误。
2. **`java.lang.OutOfMemoryError: Java heap space`** :假如在创建新的对象时, 堆内存中的空间不足以存放新创建的对象, 就会引发此错误。（和配置的最大堆内存有关，且受制于物理内存大小。最大堆内存可通过`-Xmx`参数配置，若没有特别配置，将会使用默认值，详见：[Default Java 8 max heap sizeopen in new window](https://stackoverflow.com/questions/28272923/default-xmxsize-in-java-8-max-heap-size)）
3. .....

#### （5）方法区

方法区属于是 JVM 运行时数据区域的一块逻辑区域，是各个线程共享的内存区域。

《Java 虚拟机规范》只是规定了有方法区这么个概念和它的作用，方法区到底要如何实现那就是虚拟机自己要考虑的事情了。也就是说，在不同的虚拟机实现上，方法区的实现是不同的。

当虚拟机要使用一个类时，它需要读取并解析 Class 文件获取相关信息，再将信息存入到方法区。方法区会存储已被虚拟机加载的 **类信息、字段信息、方法信息、常量、静态变量、即时编译器编译后的代码缓存等数据**。

**方法区和永久代以及元空间是什么关系呢？** 

方法区和永久代以及元空间的关系很像 Java 中接口和类的关系，类实现了接口，这里的类就可以看作是永久代和元空间，接口可以看作是方法区，也就是说永久代以及元空间是 HotSpot 虚拟机对虚拟机规范中方法区的两种实现方式。并且，永久代是 JDK 1.8 之前的方法区实现，JDK 1.8 及以后方法区的实现变成了元空间。

![HotSpot 虚拟机方法区的两种实现](D:\Data\笔记\java\java.assets\method-area-implementation.png)

**为什么要将永久代 (PermGen) 替换为元空间 (MetaSpace) 呢?**

1、整个永久代有一个 JVM 本身设置的固定大小上限，无法进行调整，而元空间使用的是本地内存，受本机可用内存的限制，虽然元空间仍旧可能溢出，但是比原来出现的几率会更小。

> 当元空间溢出时会得到如下错误： `java.lang.OutOfMemoryError: MetaSpace`

你可以使用 `-XX：MaxMetaspaceSize` 标志设置最大元空间大小，默认值为 unlimited，这意味着它只受系统内存的限制。`-XX：MetaspaceSize` 调整标志定义元空间的初始大小如果未指定此标志，则 Metaspace 将根据运行时的应用程序需求动态地重新调整大小。

2、元空间里面存放的是类的元数据，这样加载多少类的元数据就不由 `MaxPermSize` 控制了, 而由系统的实际可用空间来控制，这样能加载的类就更多了。

3、在 JDK8，合并 HotSpot 和 JRockit 的代码时, JRockit 从来没有一个叫永久代的东西, 合并之后就没有必要额外的设置这么一个永久代的地方了。

**方法区常用参数有哪些？**

JDK 1.8 之前永久代还没被彻底移除的时候通常通过下面这些参数来调节方法区大小。

```java
-XX:PermSize=N // 方法区 (永久代) 初始大小
-XX:MaxPermSize=N // 方法区 (永久代) 最大大小,超过这个值将会抛出 OutOfMemoryError 异常:java.lang.OutOfMemoryError: PermGen
```

相对而言，垃圾收集行为在这个区域是比较少出现的，但并非数据进入方法区后就“永久存在”了。

JDK 1.8 的时候，方法区（HotSpot 的永久代）被彻底移除了（JDK1.7 就已经开始了），取而代之是元空间，元空间使用的是本地内存。下面是一些常用参数：

```java
-XX:MetaspaceSize=N //设置 Metaspace 的初始（和最小大小）
-XX:MaxMetaspaceSize=N //设置 Metaspace 的最大大小
```

与永久代很大的不同就是，如果不指定大小的话，随着更多类的创建，虚拟机会耗尽所有可用的系统内存。

#### （6）运行时常量池

运行时常量池是在类加载后由虚拟机动态生成的，**用于存放类中的符号引用，以及字面量等常量**。

Class 文件中除了有类的版本、字段、方法、接口等描述信息外，还有用于存放编译期生成的各种字面量（Literal）和符号引用（Symbolic Reference）的**常量池表（Constant Pool Table）** 。

字面量是源代码中的固定值的表示法，即通过字面我们就能知道其值的含义。字面量包括整数、浮点数和字符串字面量。常见的符号引用包括类符号引用、字段符号引用、方法符号引用、接口方法符号。

![符号引用和直接引用](D:\Data\笔记\java\java.assets\symbol-reference-and-direct-reference.png)

常量池表会在类加载后存放到方法区的运行时常量池中。

运行时常量池的功能类似于传统编程语言的符号表，尽管它包含了比典型符号表更广泛的数据。

既然运行时常量池是方法区的一部分，自然受到方法区内存的限制，当常量池无法再申请到内存时会抛出 `OutOfMemoryError` 错误。

#### （7）字符串常量池

**字符串常量池** 是 JVM 为了提升性能和减少内存消耗针对字符串（String 类）专门开辟的一块区域，主要目的是为了避免字符串的重复创建。

```java
// 在堆中创建字符串对象”ab“
// 将字符串对象”ab“的引用保存在字符串常量池中
String aa = "ab";
// 直接返回字符串常量池中字符串对象”ab“的引用
String bb = "ab";
System.out.println(aa==bb);// true
```

HotSpot 虚拟机中字符串常量池的实现是 `src/hotspot/share/classfile/stringTable.cpp` ,`StringTable` 可以简单理解为一个固定大小的`HashTable` ，容量为 `StringTableSize`（可以通过 `-XX:StringTableSize` 参数来设置），保存的是字符串（key）和 字符串对象的引用（value）的映射关系，字符串对象的引用指向堆中的字符串对象。

JDK1.7 之前，字符串常量池存放在永久代。JDK1.7 字符串常量池和静态变量从永久代移动了 Java 堆中。

>🎓补充知识：
>
>在 JDK1.7 中，字符串常量池和静态变量不再位于永久代中，而是被移动到了 Java 堆中。这个改变是为了解决关于 PermGen 空间溢出的问题，因为在旧的 JVM 版本中，字符串常量池和静态变量是存放在 PermGen（永久代）中的，当应用程序动态加载大量类时，会导致 PermGen 空间溢出。
>
>在 JDK1.7 中，字符串常量池和静态变量被移到堆中，使得其生命周期可以更好地控制，同时也提高了垃圾回收器的效率。虽然这种改变增加了堆的压力，但可以通过调整堆大小来平衡内存使用，而不像以前那样需要调整 PermGen 大小。

![image-20230426165517167](D:\Data\笔记\java\java.assets\image-20230426165517167.png)

#### （8）直接内存

直接内存是一种特殊的内存缓冲区，并不在 Java 堆或方法区中分配的，而是通过 JNI 的方式在本地内存上分配的。

直接内存并不是虚拟机运行时数据区的一部分，也不是虚拟机规范中定义的内存区域，但是这部分内存也被频繁地使用。而且也可能导致 `OutOfMemoryError` 错误出现。

JDK1.4 中新加入的 **NIO（Non-Blocking I/O，也被称为New I/O）**，引入了一种基于**通道（Channel）与缓存区（Buffer）的 I/O 方式，它可以直接使用 Native 函数库直接分配堆外内存，然后通过一个存储在 Java 堆中的 DirectByteBuffer 对象作为这块内存的引用进行操作。这样就能在一些场景中显著提高性能，因为避免了在 Java 堆和 Native 堆之间来回复制数据**。

直接内存的分配不会受到 Java 堆的限制，但是，既然是内存就会受到本机总内存大小以及处理器寻址空间的限制。

类似的概念还有**堆外内存** 。在一些文章中将直接内存等价于堆外内，个人觉得不是特别准确。

堆外内存就是把内存对象分配在堆（新生代+老年代+永久代）以外的内存，这些内存直接受操作系统管理（而不是虚拟机），这样做的结果就是能够在一定程度上减少垃圾回收对应用程序造成的影响。



### 2. HotSpot 虚拟机对象探秘

#### （1）对象的创建

**Step1：类加载检查**

虚拟机遇到一条 new 指令时，首先将去检查这个指令的参数是否能在常量池中定位到这个类的符号引用，并且检查这个符号引用代表的类是否已被加载过、解析和初始化过。如果没有，那必须先执行相应的类加载过程。

**Step2：分配内存**

在**类加载检查**通过后，接下来虚拟机将为新生对象**分配内存**。对象所需的内存大小在类加载完成后便可确定，为对象分配空间的任务等同于把一块确定大小的内存从 Java 堆中划分出来。

**分配方式**有 **“指针碰撞”** 和 **“空闲列表”** 两种，**选择哪种分配方式由 Java 堆是否规整决定，而 Java 堆是否规整又由所采用的垃圾收集器是否带有压缩整理功能决定。**

**内存分配的两种方式**：

- 指针碰撞 ： 
  - 适用场合 ：堆内存规整（即没有内存碎片）的情况下。
  - 原理 ：用过的内存全部整合到一边，没有用过的内存放在另一边，中间有一个分界指针，只需要向着没用过的内存方向将该指针移动对象内存大小位置即可。
  - 使用该分配方式的 GC 收集器：Serial, ParNew
- 空闲列表 ： 
  - 适用场合 ： 堆内存不规整的情况下。
  - 原理 ：虚拟机会维护一个列表，该列表中会记录哪些内存块是可用的，在分配的时候，找一块儿足够大的内存块儿来划分给对象实例，最后更新列表记录。
  - 使用该分配方式的 GC 收集器：CMS

选择以上两种方式中的哪一种，取决于 Java 堆内存是否规整。而 Java 堆内存是否规整，取决于 GC 收集器的算法是"标记-清除"，还是"标记-整理"（也称作"标记-压缩"），值得注意的是，复制算法内存也是规整的。

**内存分配并发问题**

在创建对象的时候有一个很重要的问题，就是线程安全，因为在实际开发过程中，创建对象是很频繁的事情，作为虚拟机来说，必须要保证线程是安全的，通常来讲，虚拟机采用两种方式来保证线程安全：

- **CAS+失败重试：** CAS 是乐观锁的一种实现方式。所谓乐观锁就是，每次不加锁而是假设没有冲突而去完成某项操作，如果因为冲突失败就重试，直到成功为止。**虚拟机采用 CAS 配上失败重试的方式保证更新操作的原子性。**
- **TLAB：** 为每一个线程预先在 Eden 区分配一块儿内存，JVM 在给线程中的对象分配内存时，首先在 TLAB 分配，当对象大于 TLAB 中的剩余内存或 TLAB 的内存已用尽时，再采用上述的 CAS 进行内存分配

**Step3：初始化零值**

内存分配完成后，虚拟机需要将分配到的内存空间都初始化为零值（不包括对象头），这一步操作保证了对象的实例字段在 Java 代码中可以不赋初始值就直接使用，程序能访问到这些字段的数据类型所对应的零值。

**Step4：设置对象头**

初始化零值完成之后，**虚拟机要对对象进行必要的设置**，例如这个对象是哪个类的实例、如何才能找到类的元数据信息、对象的哈希码、对象的 GC 分代年龄等信息。 **这些信息存放在对象头中。** 另外，根据虚拟机当前运行状态的不同，如是否启用偏向锁等，对象头会有不同的设置方式。

**Step5：执行 init 方法**

在上面工作都完成之后，从虚拟机的视角来看，一个新的对象已经产生了，但从 Java 程序的视角来看，对象创建才刚开始，`<init>` 方法还没有执行，所有的字段都还为零。所以一般来说，执行 new 指令之后会接着执行 `<init>` 方法，把对象按照程序员的意愿进行初始化，这样一个真正可用的对象才算完全产生出来。

#### （2）对象的内存布局

在 Hotspot 虚拟机中，对象在内存中的布局可以分为 3 块区域：**对象头**、**实例数据**和**对齐填充**。

**Hotspot 虚拟机的对象头包括两部分信息**，**第一部分用于存储对象自身的运行时数据**（哈希码、GC 分代年龄、锁状态标志等等），**另一部分是类型指针**，即对象指向它的类元数据的指针，虚拟机通过这个指针来确定这个对象是哪个类的实例。

**实例数据部分是对象真正存储的有效信息**，也是在程序中所定义的各种类型的字段内容。

**对齐填充部分不是必然存在的，也没有什么特别的含义，仅仅起占位作用。** 因为 Hotspot 虚拟机的自动内存管理系统要求对象起始地址必须是 8 字节的整数倍，换句话说就是对象的大小必须是 8 字节的整数倍。而对象头部分正好是 8 字节的倍数（1 倍或 2 倍），因此，当对象实例数据部分没有对齐时，就需要通过对齐填充来补全。

#### （3）对象的访问定位

建立对象就是为了使用对象，我们的 Java 程序通过栈上的 reference 数据来操作堆上的具体对象。对象的访问方式由虚拟机实现而定，目前主流的访问方式有：**使用句柄**、**直接指针**。

**句柄**

如果使用句柄的话，那么 Java 堆中将会划分出一块内存来作为句柄池，reference 中存储的就是对象的句柄地址，而句柄中包含了对象实例数据与对象类型数据各自的具体地址信息。

![对象的访问定位-使用句柄](D:\Data\笔记\java\java.assets\access-location-of-object-handle.png)

**直接指针**

如果使用直接指针访问，reference 中存储的直接就是对象的地址。

![对象的访问定位-直接指针](D:\Data\笔记\java\java.assets\access-location-of-object-handle-direct-pointer.png)

这两种对象访问方式各有优势。使用句柄来访问的最大好处是 reference 中存储的是稳定的句柄地址，在对象被移动时只会改变句柄中的实例数据指针，而 reference 本身不需要修改。使用直接指针访问方式最大的好处就是速度快，它节省了一次指针定位的时间开销。

HotSpot 虚拟机主要使用的就是这种方式来进行对象访问。



## 二、Java 集合

![img](D:\Data\笔记\java\java.assets\java-collection-hierarchy.png)

### 1. List、Set、Queue、Map 四者的区别

- List（对付顺序的好帮手）：存储的元素是**有序的、可重复的**；
- Set（注重独一无二的性质）：存储的元素是**无序的、不可重复的**；
- Queue（实现排队功能）：按特定的排队规则来确定先后顺序，存储的元素是**有序的、可重复的**；
- Map（键值对）：key 是**无序的、不可重复的**，value 是**无序的、可重复的**。

### 2. 集合框架底层数据结构

#### （1）List

- `ArrayList`：`Object[]` 数组，线程不安全
- `Vector`：`Object[]` 数组，线程同步
- `LinkedList`：双向链表（JDK1.6之前为循环链表，JDK1.7取消了循环），线程不安全

#### （2）Set

- `HashSet`（无序，唯一）：基于 `HashMap` 实现，底层采用 `HashMap` 保存元素，线程不安全
- `LinkedHashSet`：`LinkedHashSet` 是 `HashSet` 的子类，并且其内部是通过 `LinkedHashMap` 来实现的，线程不安全
- `TreeSet`（有序，唯一）：红黑树（自平衡的排序二叉树），线程不安全

#### （3）Queue

- `PriorityQueue`：`Object[]` 数组来实现二叉堆
- `ArrayQueue`：`Object[]` 数组+双指针

#### （4）Map

- `HashMap`：JDK1.8 之前 `HashMap` 由数组+链表组成的，采用拉链法解决哈希冲突。**JDK1.8 以后，当链表长度大于阈值（默认为8）时，将链表转换为红黑树，以减少搜索时间**，线程不安全

  > 🎓补充知识：
  >
  > 将链表转换成红黑树前会判断，如果当前数组的长度小于 64，那么会选择先进行数组扩容，而不是转换为红黑树

- `LinkedHashMap`：`LinkedHashMap` 继承自 `HashMap`，所以它的底层仍然是基于拉链式散列结构即由数组和链表或红黑树组成。另外，`LinkedHashMap` 在上面结构的基础上，**增加了一条双向链表**，使得上面的结构可以保持键值对的插入顺序，同时通过对链表进行相应的操作，实现了访问顺序相关逻辑，线程不安全

- `Hashtable`：数组+链表组成，数组是 `Hashtable` 的主体，链表则主要为了解决哈希冲突而存在的，线程安全

- `TreeMap`：红黑树，线程不安全

### 3. 如何选用集合

主要根据集合的特点来选用，比如我们需要根据键值获取到元素值时就选用 `Map` 接口下的集合，需要排序时选择 `TreeMap`，不需要排序时就选择 `HashMap`,需要保证线程安全就选用 `ConcurrentHashMap`。

当我们只需要存放元素值时，就选择实现`Collection` 接口的集合，需要保证元素唯一时选择实现 `Set` 接口的集合比如 `TreeSet` 或 `HashSet`，不需要就选择实现 `List` 接口的比如 `ArrayList` 或 `LinkedList`，然后再根据实现这些接口的集合的特点来选用。

### 4. List 集合的那些事儿

#### （1）ArrayList 和 Vector 的区别

`ArrayList`  是  `List`  的主要实现类，底层使用 `Object[]` 存储，线程不安全 ；

`Vector` 是 `List` 的古老实现类，底层使用 `Object[]` 存储，线程安全的。

#### （2）ArrayList 和 LinkedList 的区别

**① 是否保证线程安全**

`ArrayList` 和 `LinkedList` 都不保证线程安全

**② 底层数据结构**

`ArrayList` 底层采用数组实现， `LinkedList` 在 JDK1.7 之后采用双向链表实现

**③ 是否支持随机访问**

`ArrayList` 支持随机访问， `LinkedList` 不支持高效的随机访问。

`ArrayList` 采用数组存储，插入和删除的时间复杂度受元素位置的影响，使用 `add(E e)`，默认追加到列表末尾，时间复杂度 O(1)，而 `add(int index, E e)`时间复杂度为 O(n-i)

`LinkedList` 采用双向链表存储，在头尾插入和删除不受位置的影响（`add(E e)`、`addFirst(E e)`、`addLast(E e)`、`removeFirst()`、`removeLast()`），时间复杂度为 O(1)，如果在指定位置插入删除元素的话，时间复杂度为 O(n)，因为索引需要先移动到指定位置

**④ 内存占用**

`ArrayList` 的空间浪费主要体现在**列表的结尾会预留一定的容量空间**，而 `LinkedList` 的空间花费则体现在它的每一个元素都需要消耗比 `ArrayList` 更多的空间（因为要存放直接后继和直接前驱以及数据）。

我们在项目中一般是不会使用到 `LinkedList` 的，需要用到 `LinkedList` 的场景几乎都可以使用 `ArrayList` 来代替，并且，性能通常会更好！

#### （3）ArrayList 扩容

`ArrayList` 默认容量为 10，每次扩容大约为1.5倍左右，调用`Arrays.copyof()`。

> 📙详情参考：
>
> [ArrayList 源码分析 ](https://javaguide.cn/java/collection/arraylist-source-code.html#一步一步分析-arraylist-扩容机制)

### 5. Set 集合的那些事儿

#### （1）无序性和不可重复性的含义

无序性不等于随机性 ，无序性是指存储的数据在底层数组中并非按照数组索引的顺序添加 ，而是根据数据的哈希值决定的。

不可重复性是指添加的元素按照 `equals()` 判断时 ，返回 false，需要同时重写 `equals()` 方法和 `hashCode()` 方法。

#### （2）比较 HashSet、LinkedHashSet 和 TreeSet 的异同

`HashSet`、`LinkedHashSet` 和 `TreeSet` 都是 `Set` 接口的实现类，都能保证元素唯一，并且都不是线程安全的。

`HashSet`、`LinkedHashSet` 和 `TreeSet` 的主要区别在于底层数据结构不同。`HashSet` 的底层数据结构是哈希表（基于 `HashMap` 实现）。`LinkedHashSet` 的底层数据结构是链表和哈希表，元素的插入和取出顺序满足 FIFO。`TreeSet` 底层数据结构是红黑树，元素是有序的，排序的方式有自然排序和定制排序。

`HashSet` 用于不需要保证元素插入和取出顺序的场景

`LinkedHashSet` 用于保证元素的插入和取出顺序满足 FIFO 的场景

`TreeSet` 用于支持对元素自定义排序规则的场景

### 6. Queue 集合的那些事儿

#### （1）Queue 与 Deque 的区别

**`Queue` 是单端队列，只能从一端插入元素，另一端删除元素。**

`Queue` 扩展了 `Collection` 的接口，根据 **因为容量问题而导致操作失败后处理方式的不同** 可以分为两类方法: 一种在操作失败后会抛出异常，另一种则会返回特殊值。

| `Queue` 接口 | 抛出异常  | 返回特殊值 |
| ------------ | --------- | ---------- |
| 插入队尾     | add(E e)  | offer(E e) |
| 删除队首     | remove()  | poll()     |
| 查询队首元素 | element() | peek()     |

**`Deque` 是双端队列，在队列的两端均可以插入或删除元素。**

`Deque` 扩展了 `Queue` 的接口, 增加了在队首和队尾进行插入和删除的方法，同样根据失败后处理方式的不同分为两类：

| `Deque` 接口 | 抛出异常      | 返回特殊值      |
| ------------ | ------------- | --------------- |
| 插入队首     | addFirst(E e) | offerFirst(E e) |
| 插入队尾     | addLast(E e)  | offerLast(E e)  |
| 删除队首     | removeFirst() | pollFirst()     |
| 删除队尾     | removeLast()  | pollLast()      |
| 查询队首元素 | getFirst()    | peekFirst()     |
| 查询队尾元素 | getLast()     | peekLast()      |

事实上，`Deque` 还提供有 `push()` 和 `pop()` 等其他方法，可用于模拟栈。

#### （2）ArrayDeque 与 LinkedList 的区别

`ArrayDeque` 和 `LinkedList` 都实现了 `Deque` 接口，两者都具有队列的功能

- `ArrayDeque` 是基于可变长的数组和双指针来实现，而 `LinkedList` 则通过链表来实现
- `ArrayDeque` 不支持存储 `NULL` 数据，但 `LinkedList` 支持
- `ArrayDeque` 是在 JDK1.6 才被引入的，而`LinkedList` 早在 JDK1.2 时就已经存在
- `ArrayDeque` 插入时可能存在扩容过程，不过均摊后的插入操作依然为 O(1)。虽然 `LinkedList` 不需要扩容，但是每次插入数据时均需要申请新的堆空间，均摊性能相比更慢

从性能的角度上，选用 `ArrayDeque` 来实现队列要比 `LinkedList` 更好。此外，`ArrayDeque` 也可以用于实现栈。

#### （3）说一说 PriorityQueue

`PriorityQueue` 是在 JDK1.5 中被引入的, 其与 `Queue` 的区别在于元素出队顺序是与优先级相关的，即总是优先级最高的元素先出队。

这里列举其相关的一些要点：

- `PriorityQueue` 利用了二叉堆的数据结构来实现的，底层使用可变长的数组来存储数据
- `PriorityQueue` 通过堆元素的上浮和下沉，实现了在 O(logn) 的时间复杂度内插入元素和删除堆顶元素。
- `PriorityQueue` 是非线程安全的，且不支持存储 `NULL` 和 `non-comparable` 的对象。
- `PriorityQueue` 默认是小顶堆，但可以接收一个 `Comparator` 作为构造参数，从而来自定义元素优先级的先后。

`PriorityQueue` 在面试中可能更多的会出现在手撕算法的时候，典型例题包括堆排序、求第 K 大的数、带权图的遍历等，所以需要会熟练使用才行。

### 7. Map 接口的那些事儿

#### （1）HashMap 和 Hashtable 的区别

**① 是否线程安全**

`HashMap` 是非线程安全的，`Hashtable` 是线程安全的。`Hashtable` 内部的方法基本都经过 `synchronized` 修饰，保证线程安全的话就用 `ConcurrentHashMap` 

因为线程安全的问题，`HashMap` 要比 `Hashtable` 效率高一点。另外，`Hashtable` 基本被淘汰，不要在代码中使用它

**② 对 Null key 和 Null value 的支持**

`HashMap` 可以存储 null 的 key 和 value，但 null 作为键只能有一个，null 作为值可以有多个；`Hashtable` 不允许有 null 键和 null 值，否则会抛出 `NullPointerException`。

**③ 初始化容量大小和每次扩充容量大小的不同**

- 创建时如果不指定容量初始值，`Hashtable` 默认的初始大小为 11，之后每次扩充，容量变为原来的 2n+1。`HashMap` 默认的初始化大小为 16。之后每次扩充，容量变为原来的 2 倍。

- 创建时如果给定了容量初始值，那么 `Hashtable` 会直接使用你给定的大小，而 `HashMap` 会将其扩充为 2 的幂次方大小。也就是说 `HashMap` 总是使用 2 的幂作为哈希表的大小。

**④ 底层数据结构**

 JDK1.8 以后的 `HashMap` 在解决哈希冲突时有了较大的变化，当链表长度大于阈值（默认为 8）时，将链表转化为红黑树，以减少搜索时间。

`Hashtable` 没有这样的机制。

#### （2）HashMap 和 TreeMap 的区别

`TreeMap` 和`HashMap` 都继承自`AbstractMap` ，但是需要注意的是 `TreeMap` 它还实现了`NavigableMap  `接口和 `SortedMap` 接口。

实现 `NavigableMap` 接口让 `TreeMap` 有了对**集合内元素的搜索的能力**。

实现 `SortedMap` 接口让 `TreeMap` 有了对**集合中的元素根据键排序的能力**。默认是按 key 的升序排序，不过我们也可以指定排序的比较器。

#### （3）HashMap 的长度为什么是 2 的幂次方

为了能让 `HashMap` 存取高效，尽量较少碰撞，也就是要尽量把数据分配均匀。Hash 值的范围值-2147483648 到 2147483647，前后加起来大概 40 亿的映射空间，只要哈希函数映射得比较均匀松散，一般应用是很难出现碰撞的。但问题是一个 40 亿长度的数组，内存是放不下的。所以这个散列值是不能直接拿来用的。用之前还要先对数组的长度取模运算，得到的余数才能用来要存放的位置也就是对应的数组下标。这个数组下标的计算方法是“ `(n - 1) & hash`”。（n 代表数组长度）。这也就解释了 `HashMap` 的长度为什么是 2 的幂次方。

**这个算法应该如何设计呢？**

我们首先可能会想到采用%取余的操作来实现。但是，重点来了：**“取余(%)操作中如果除数是 2 的幂次则等价于与其除数减一的与(&)操作（也就是说 hash%length==hash&(length-1)的前提是 length 是 2 的 n 次方；）。”** 并且 **采用二进制位操作 &，相对于%能够提高运算效率，这就解释了 HashMap 的长度为什么是 2 的幂次方**。

#### （4）HashMap 多线程操作导致死循环

主要原因在于并发下的 rehash 会造成元素之间会形成一个循环链表。不过，jdk 1.8 后解决了这个问题，但是还是不建议在多线程下使用 `HashMap`，因为多线程下使用 `HashMap` 还是会存在其他问题比如数据丢失。并发环境下推荐使用 `ConcurrentHashMap` 。

>🎓补充知识：
>
>Hash 表的尺寸和容量非常的重要。一般来说，Hash 表这个容器当有数据要插入时，都会检查容量有没有超过设定的 thredhold，如果超过，需要增大 Hash 表的尺寸，但是这样一来，整个 Hash 表里的无素都需要被重算一遍。这叫 rehash，这个成本相当的大。
>
>[HashMap的死循环 ](https://coolshell.cn/articles/9606.html)

#### （5）HashMap 的7种遍历方法

`HashMap` **遍历从大的方向来说，可分为以下 4 类**：

1. 迭代器（Iterator）方式遍历；
2. For Each 方式遍历；
3. Lambda 表达式遍历（JDK 1.8+）;
4. Streams API 遍历（JDK 1.8+）。

但每种类型下又有不同的实现方式，因此具体的遍历方式又可以分为以下 7 种：

1. 使用迭代器（Iterator）EntrySet 的方式进行遍历；
2. 使用迭代器（Iterator）KeySet 的方式进行遍历；
3. 使用 For Each EntrySet 的方式进行遍历；
4. 使用 For Each KeySet 的方式进行遍历；
5. 使用 Lambda 表达式的方式进行遍历；
6. 使用 Streams API 单线程的方式进行遍历；
7. 使用 Streams API 多线程的方式进行遍历。

接下来我们来看每种遍历方式的具体实现代码。

```java
// 迭代器 EntrySet
Iterator<Map.Entry<Integer, String>> iterator = map.entrySet().iterator();
while (iterator.hasNext()) {
    Map.Entry<Integer, String> entry = iterator.next();
    System.out.println(entry.getKey());
    System.out.println(entry.getValue());
}
```

```java
// 迭代器 KeySet
Iterator<Integer> iterator = map.keySet().iterator();
while (iterator.hasNext()) {
    Integer key = iterator.next();
    System.out.println(key);
    System.out.println(map.get(key));
}
```

```java
// ForEach EntrySet
for (Map.Entry<Integer, String> entry : map.entrySet()) {
    System.out.println(entry.getKey());
    System.out.println(entry.getValue());
}
```

```java
// ForEach KeySet
for (Integer key : map.keySet()) {
    System.out.println(key);
    System.out.println(map.get(key));
}
```

```java
// Lambda
map.forEach((key, value) -> {
    System.out.println(key);
    System.out.println(value);
});
```

```java
// Streams API 单线程
map.entrySet().stream().forEach((entry) -> {
    System.out.println(entry.getKey());
    System.out.println(entry.getValue());
});
```

```java
// Streams API 多线程
map.entrySet().parallelStream().forEach((entry) -> {
    System.out.println(entry.getKey());
    System.out.println(entry.getValue());
});
```

**存在阻塞时 parallelStream 性能最高, 非阻塞时 parallelStream 性能最低** 。

当遍历不存在阻塞时, parallelStream 的性能是最低的：

```text
Benchmark               Mode  Cnt     Score      Error  Units
Test.entrySet           avgt    5   288.651 ±   10.536  ns/op
Test.keySet             avgt    5   584.594 ±   21.431  ns/op
Test.lambda             avgt    5   221.791 ±   10.198  ns/op
Test.parallelStream     avgt    5  6919.163 ± 1116.139  ns/op
```

加入阻塞代码`Thread.sleep(10)`后, parallelStream 的性能才是最高的:

```text
Benchmark               Mode  Cnt           Score          Error  Units
Test.entrySet           avgt    5  1554828440.000 ± 23657748.653  ns/op
Test.keySet             avgt    5  1550612500.000 ±  6474562.858  ns/op
Test.lambda             avgt    5  1551065180.000 ± 19164407.426  ns/op
Test.parallelStream     avgt    5   186345456.667 ±  3210435.590  ns/op
```

#### （6）ConcurrentHashMap 和 Hashtable 的区别

`ConcurrentHashMap` 和 `Hashtable` 的区别主要体现在实现线程安全的方式上不同。

**① 底层数据结构**

JDK1.7 的 `ConcurrentHashMap` 底层采用 **分段的数组+链表** 实现，JDK1.8 采用的数据结构跟 `HashMap1.8` 的结构一样，数组+链表/红黑二叉树。`Hashtable` 和 JDK1.8 之前的 `HashMap` 的底层数据结构类似都是采用 **数组+链表** 的形式，数组是 HashMap 的主体，链表则是主要为了解决哈希冲突而存在的。

**② 实现线程安全的方式**

- 在 JDK1.7 的时候，`ConcurrentHashMap` 对整个桶数组进行了分割分段（Segment`，分段锁），每一把锁只锁容器其中一部分数据，多线程访问容器里不同数据段的数据，就不会存在锁竞争，提高并发访问率。

  ![Java7 ConcurrentHashMap 存储结构](D:\Data\笔记\java\java.assets\java7_concurrenthashmap.png)

- 到了 JDK1.8 的时候，`ConcurrentHashMap` 已经摒弃了 `Segment` 的概念，而是直接用 `Node` 数组+链表+红黑树的数据结构来实现，并发控制使用 `synchronized` 和 CAS 来操作。（JDK1.6 以后 `synchronized` 锁做了很多优化） 整个看起来就像是优化过且线程安全的 `HashMap`，虽然在 JDK1.8 中还能看到 `Segment` 的数据结构，但是已经简化了属性，只是为了兼容旧版本

  > 🎓补充知识：
  >
  > CAS 是英文单词 **CompareAndSwap** 的缩写，中文意思是：比较并替换。CAS 需要有 3 个操作数：内存地址 V，旧的预期值 A，即将要更新的目标值 B。
  >
  > CAS 指令执行时，当且仅当内存地址 V 的值与预期值 A 相等时，将内存地址 V 的值修改为 B，否则就什么都不做。整个比较并替换的操作是一个原子操作。

  ![Java8 ConcurrentHashMap 存储结构](D:\Data\笔记\java\java.assets\java8_concurrenthashmap.png)

- **`Hashtable`（同一把锁）** :使用 `synchronized` 来保证线程安全，效率非常低下。当一个线程访问同步方法时，其他线程也访问同步方法，可能会进入阻塞或轮询状态，如使用 put 添加元素，另一个线程不能使用 put 添加元素，也不能使用 get，竞争会越来越激烈效率越低。

