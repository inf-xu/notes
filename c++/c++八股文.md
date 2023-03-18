## 一、在main执行之前和之后执行的代码可能是什么？

### 1. main 函数执行之前，主要就是初始化系统相关的资源

- 初始化栈指针（栈用于存储一些需要的局部变量或者其他数据）

- 初始化**静态变量**和**全局变量**，即 `.data` 数据段的数据

  > 补充知识：
  >
  > 一个可执行C程序在内存中主要包含5个区域：代码段（text）、数据段（data）、BSS段、堆段（heap）和栈段（stack）。
  >
  > 其中（text、data、bss）是程序编译完成就存在的，此时程序并未载入内存进行执行。（heap、stack）是程序被加载到内存才存在的。
  >
  > **务必详细阅读《[C++ 堆区，栈区，数据段，bss段，代码区](https://blog.csdn.net/JACKSONMHLK/article/details/114392343)》**

- 将**未初始化部分的全局变量**赋初值，即 `.bss`段的内容 ：数值型 `short = 0`，`int = 0` ，`long = 0` ，`bool=FALSE` ，指针为 `NULL`等等。

- **全局对象**初始化，在main之前调用构造函数，这是可能会执行前的一段代码

- 将main函数的参数 `argc`，`argv` 等传递给main函数，然后才真正运行main函数

  > 补充知识：
  >
  > `argc` 是 argument count 的缩写，表示传入main函数的参数个数
  >
  > `argv` 是 argument vector 的缩写，表示传入main函数的参数序列或指针，并且第一个参数 `argv[0]` 一定是程序的名称，并且包含了程序所在的完整路径，所以确切的说需要我们输入的main函数的参数个数应该是 `argc-1` 个

- `__attribte__((constructor))`

  >补充知识：
  >
  >`__attribute__` 可用于为函数或者数据声明赋属性值
  >
  >参看《[C/C++编程：__attribute__((constructor))、 __attribute__((destructor))](https://blog.csdn.net/zhizhengguan/article/details/111151773)》

### 2. main 函数执行之后

- **全局对象的析构函数**会在main函数之后执行

- 可以用 `atexit` 注册一个函数，它会在main之后执行

  > 我们需要在程序退出的时候做一些诸如释放资源的操作，但程序退出的方式有很多种，比如main函数运行结束、在程序的某个地方用exit()结束程序、用户通过 Ctrl+C 或 Ctrl+break 操作来终止程序等等，因此需要有一种与程序退出方式无关的方法来进行程序退出时的必要处理。方法就是用 `atexit` 函数来注册程序正常终止时要被调用的函数。
  >
  > 参看《[C/C++程序终止时执行的函数——atexit()函数详解](http://www.51testing.com/html/38/225738-235458.html)》

-  `__attribte__((destructor))`



## 二 、结构体内存对齐问题

### 1. 什么是内存对齐

理论上，在 32 位系统下，int 占4 byte，char 占1 byte，那么将它们放到一个结构体中应该占 4+1=5 byte；但是实际上，通过运行程序得到的结果是 8 byte，这就是内存对齐所导致的。

尽管内存是以字节为单位，但是大部分处理器并不是按字节块来存取内存的。它一般会以双字节、四字节、8 字节、16 字节甚至 32 字节为单位来存取内存，我们将上述这些存取单位称为**内存存取粒度**。

现在考虑4字节存取粒度的处理器取 int 类型变量（32位系统），该处理器只能从地址为 4 的倍数的内存开始读取数据。假如没有内存对齐机制，数据可以任意存放，现在一个 int 变量存放在从地址 1 开始的联系四个字节地址中，该处理器去取数据时，要先从 0 地址开始读取第一个 4 字节块，剔除不想要的字节（0 地址），然后从地址 4 开始读取下一个 4 字节块，同样剔除不要的数据（5，6，7地址），最后留下的两块数据合并放入寄存器，这需要做很多工作。

现在有了内存对齐的，int 类型数据只能存放在按照对齐规则的内存中，比如说 0 地址开始的内存。那么现在该处理器在取数据时一次性就能将数据读出来了，而且不需要做额外的操作，提高了效率。

### 2. 内存对齐规则

结构体内成员按照声明顺序存储，第一个成员地址和整个结构体地址相同。

每个特定平台上的编译器都有自己的默认“对齐系数”（也叫对齐模数）。gcc 中默认 `#pragma pack(4)` ，可以通过预编译命令 `#pragma pack(n)`，n = 1,2,4,8,16 来改变这一系数。

**有效对齐值**：是给定值 `#pragma pack(n)` 和 结构体 中最长数据类型长度中较小的那个，即 `min(#pragma pack(n), 结构体中最长数据类型长度)`。有效对齐值也叫**对齐单位**。

原则如下：

- 结构体第一个成员的**偏移量（offset）**为 0，以后每个成员相对于结构体首地址的 offset 都是**该成员大小与有效对齐值中较小那个的整数倍**，如有需要编译器会在成员之间加上填充字节。
- **结构体的总大小**为 有效对齐值 的**整数倍**，如有需要编译器会在最末一个成员之后加上填充字节。

```cpp
// 32 位系统  #pragma pack(4)
#include<stdio.h>

struct {
    int i;    
    char c1;  
    char c2;  
} x1;

struct {
    char c1;  
    int i;    
    char c2;  
} x2;

struct {
    char c1;  
    char c2; 
    int i;    
} x3;

int main() {
    printf("%d\n",sizeof(x1));  // 输出8
    printf("%d\n",sizeof(x2));  // 输出12
    printf("%d\n",sizeof(x3));  // 输出8
    return 0;
}
```

x2 分析如下：

对齐单位： `min(#pragma pack(4), sizeof(int)) = 4`

首先使用规则1，对成员变量进行对齐：

`sizeof(c1) = 1 <= 4（对齐单位）`，按照 1 字节对齐，占用第 0 单元；

`sizeof(i) = 4 <= 4（对齐单位）`，相对于结构体首地址的偏移要为 4 的倍数，占用第 4，5，6，7 单元；

`sizeof(c2) = 1 <= 4（对齐单位）`，相对于结构体首地址的偏移要为 1 的倍数，占用第 8 单元；

然后使用规则 2，对结构体整体进行对齐：

x2 中变量 i 占用内存最大占 4 字节，而有效对齐单位也为 4 字节，两者较小值就是 4 字节。因此整体也是按照 4 字节对齐。由规则 1 得到s2占9 个字节，此处再按照规则 2 进行整体的4字节对齐，所以整个结构体占用 12 个字节。

### 3. alignas 和 alignof

C++11 以后引入的 alignas 和 alignof 。

- alignof 可以计算出类型的对齐方式
- alignas 可以指定结构体的对齐方式

```cpp
// size:16
struct  struct_Test1 {
	char c;
	int  i;
	double d;
};

// size:16
struct alignas(8) struct_Test2 {
	char c;
	int  i;
	double d;
};

// size:16
struct alignas(16) struct_Test3 {
	char c;
	int  i;
	double d;
};

// size:32
struct alignas(32) struct_Test4 {
	char c;
	int  i;
	double d;
};
```

**alignas 失效情况：**

- 若指定的 alignas 小于自然对齐的最小单位，则被忽略。

- 如果想使用单字节对齐的方式，使用 alignas 是无效的。应该使用 `#pragma pack(push,1)` 或者使用 `__attribute__((packed))`。

  ```cpp
  #if defined(__GNUC__) || defined(__GNUG__)
    #define ONEBYTE_ALIGN __attribute__((packed))
  #elif defined(_MSC_VER)
    #define ONEBYTE_ALIGN
    #pragma pack(push,1)
  #endif
  
  struct Info {
    uint8_t a;
    uint32_t b;
    uint8_t c;
  } ONEBYTE_ALIGN;
  
  #if defined(__GNUC__) || defined(__GNUG__)
    #undef ONEBYTE_ALIGN
  #elif defined(_MSC_VER)
    #pragma pack(pop)
    #undef ONEBYTE_ALIGN
  #endif
  
  std::cout << sizeof(Info) << std::endl;   // 6 1 + 4 + 1
  std::cout << alignof(Info) << std::endl;  // 1
  ```

- 确定结构体中每个元素大小可以通过下面这种方式，这种处理方式是 alignas 处理不了的。

  ```cpp
  #if defined(__GNUC__) || defined(__GNUG__)
    #define ONEBYTE_ALIGN __attribute__((packed))
  #elif defined(_MSC_VER)
    #define ONEBYTE_ALIGN
    #pragma pack(push,1)
  #endif
  
  /**
  * 0 1   3     6   8 9            15
  * +-+---+-----+---+-+-------------+
  * | |   |     |   | |             |
  * |a| b |  c  | d |e|     pad     |
  * | |   |     |   | |             |
  * +-+---+-----+---+-+-------------+
  */
  struct Info {
    uint16_t a : 1;
    uint16_t b : 2;
    uint16_t c : 3;
    uint16_t d : 2;
    uint16_t e : 1;
    uint16_t pad : 7;
  } ONEBYTE_ALIGN;
  
  #if defined(__GNUC__) || defined(__GNUG__)
    #undef ONEBYTE_ALIGN
  #elif defined(_MSC_VER)
    #pragma pack(pop)
    #undef ONEBYTE_ALIGN
  #endif
  
  std::cout << sizeof(Info) << std::endl;   // 2
  std::cout << alignof(Info) << std::endl;  // 1
  ```



## 三、指针和引用的区别

- 指针是一个变量，存储的是一个地址；引用是原来变量的别名，本质是同一个东西

- 指针可以多级；引用只有一级

- 指针可以为空；引用不能为NULL，并且在定义的时候必须初始化

- 指针初始化之后可以改变指向（指针声明和定义可以分开）；引用在初始化之后不可再改变

- sizeof 指针得到的是指针的大小；sizeof 引用得到的是引用所指对象的大小

- 指针作为参数传递的时候，也是将实参的一个拷贝传递给形参，两者指向的地址相同，但不是同一个变量，在函数中改变这个变量的指向不影响实参；应用却可以

  > **函数内改变指针的值可作用于外部主函数内；函数内改变指针地址在该函数结束时就还原**
  >
  > ```cpp
  > void test(int *p) {
  >     int a = 2;
  >     p = &a;
  >     cout << "test: " << p << " " << *p <<endl;
  > }
  > 
  > int main(void) {
  >     int a = 1;
  >     int *p = &a;
  >     cout << "main: " << p << " " << *p <<endl;
  >     test(p);
  >     cout << "main: " << p << " " << *p <<endl;
  >     return 0;
  > }
  > 
  > /* 输出结果为：
  > main: 0x7ffdf35c04bc 1
  > test: 0x7ffdf35c0494 2
  > main: 0x7ffdf35c04bc 1
  > */
  > 
  > void test(int *p){
  >     int a = 2;
  >     *p = a;
  >     cout << "test: " << p << " " << *p <<endl;
  > }
  > 
  > int main(void) {
  >     int a = 1;
  >     int *p = &a;
  >     cout << "main: " << p << " " << *p <<endl;
  >     test(p);
  >     cout << "main: " << p << " " << *p <<endl;
  >     return 0;
  > }
  > 
  > /* 输出结果为：
  > main: 0x7ffe0eb8a13c 1
  > test: 0x7ffe0eb8a13c 2
  > main: 0x7ffe0eb8a13c 2
  > */
  > ```

  

## 四、传递函数参数中的值传递、指针传递和引用传递

### 1. 值传递、指针传递和引用传递的区别和效率

- 值传递：有一个形参向函数所属的栈拷贝数据的过程，如果值传递的对象是类对象或是大的结构体对象，将消耗一定的时间和空间（传值）
- 指针传递：同样有一个形参向函数所属的栈拷贝数据的过程，但拷贝的数据是一个固定为指针大小（4字节或8字节）的地址（传值，传递的是地址值）
- 引用传递：同样有一个形参向函数所属的栈拷贝数据的过程，但其是针对地址的，相当于为该数据所在的地址起了一个别名（传地址）

效率上讲，指针传递和引用传递比值传递效率高。一般主张使用引用传递，代码逻辑上更加紧凑、清晰。

### 2. 什么时候使用指针传递，什么时候使用引用传递

- **需要返回函数内局部变量的内存的时候使用指针**。使用指针传参需要开辟内存，用完需要释放，不然内存泄漏。而返回局部变量的引用没有意义。

  ```cpp
  void test(int *p){
      int **p2 = &p;
      cout << "test: " << p2 << " " << *p2 <<endl;
  }
  
  int main(void) {
      int a = 1;
      int *p = &a;
      int **p2 = &p;
      cout << "main: " << p2 << " " << *p2 <<endl;
      test(p);
      cout << "main: " << p2 << " " << *p2 <<endl;
      return 0;
  }
  
  /* 输出结果为：
  main: 0x7ffde69f6418 0x7ffde69f6414
  test: 0x7ffde69f63e8 0x7ffde69f6414
  main: 0x7ffde69f6418 0x7ffde69f6414
  */
  ```

- 对栈空间大小比较敏感（比如递归）的时候使用引用。使用**引用传递不需要创建临时变量**，开销要更小。

- **类对象作为参数传递的时候使用引用**，这是 C++ 类对象传递的标准方式。



## 五、堆和栈的区别

- 申请方式不同：栈是系统自动分配的；堆是自己申请和释放的
- 申请大小限制不同：
  - 栈顶和栈底是之前预设好的，栈是向栈底扩展，**大小固定**，可以通过 `ulimit -a` 查看，由 `ulimit -s` 修改
  - 堆向高地址扩展，是**不连续的内存区域**，大小可以灵活调整

- 申请效率不同：
  - 栈由系统分配，速度快，不会有碎片
  - 堆由程序员分配，速度慢，且会有碎片

|              | 堆                                                           | 栈                                                           |
| ------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 管理方式     | 堆中资源由程序员控制（容易产生  memory leak）                | 栈资源由编译器自动管理，无需手工控制                         |
| 内存管理机制 | 系统有一个记录空闲内存地址的链表，当系统收到程序申请时，遍历该链表，寻找第一个空间大于申请空间的堆结点，删除空闲结点链表中的该结点，并将该结点空间分配给程序（大多数系统会在这块内存空间首地址记录本次分配的大小，这样delete才能正确释放本内存空间，另外系统会将多余的部分重新放入空闲链表中） | 只要栈的剩余空间大于所申请空间，系统为程序提供内存，否则报异常提示栈溢出。（这一块理解一下链表和队列的区别，不连续空间和连续空间的区别，应该就比较好理解这两种机制的区别了） |
| 空间大小     | 堆是不连续的内存区域（因为系统是用链表来存储空闲内存地址，自然不是连续的），堆大小受限于计算机系统中有效的虚拟内存（32bit 系统理论上是4G），所以堆的空间比较灵活，比较大 | 栈是一块连续的内存区域，大小是操作系统预定好的，windows下栈大小是2M（也有是1M，在编译时确定，VC 中可设置） |
| 碎片问题     | 对于堆，频繁的 new/delete 会造成大量碎片，使程序效率降低     | 对于栈，它是有点类似于数据结构上的一个先进后出的栈，进出一一对应，不会产生碎片 |
| 生长方向     | 堆向上，向高地址方向增长                                     | 栈向下，向低地址方向增长                                     |
| 分配方式     | 堆都是动态分配（没有静态分配的堆）                           | 栈有静态分配和动态分配，静态分配由编译器完成（如局部变量分配），动态分配由 alloca 函数分配，但栈的动态分配的资源由编译器进行释放，无需程序员实现 |
| 分配效率     | 堆由 C/C++ 函数库提供，机制很复杂。所以堆的效率比栈低很多    | 栈是其系统提供的数据结构，计算机在底层对栈提供支持，分配专门寄存器存放栈地址，栈操作有专门指令 |



## 六、你觉得堆快一点还是栈快一点？

毫无疑问栈快一点。

因为操作系统会在底层对栈提供支持，会分配专门的寄存器存放栈的地址，栈的入栈出栈操作有专门的指令执行，所以栈的效率比较高也比较快。

而堆的操作是由 C/C++ 函数库提供，在分配堆内存的时候需要一定的算法寻找合适大小的内存。并且获取堆的内容需要两次访问，第一次访问指针，第二次更具指针保存的地址访问内存，因此堆会比较慢。



## 七、区别以下指针类型？

### 1. 看懂 C++ 类型声明的规律

（1） 第一步，找到变量名，如果没有变量名，找到最里面的结构

（2） 第二步，向右看，读出你看到的东西，但是不要跳过括号

（3） 第三步，再向左看，读出你看到的东西，但是也不要跳过括号

（4） 第四步，如果有括号的话，跳出一层括号

（5） 第五步，重复上述过程，直到你读出最终类型

```cpp
/* int * v[5]
（1）找到变量 v
（2）向右看，数组尺寸 5，右边没东西了
（3）向左看，是一个指针
所以：v 就是一个大小为 5 的数组，每个元素是指向 int 的指针
=> v 是一个有 5 个指向 int 的指针的数组
*/

/* int (*v)[5]
（1）找到变量 v
（2）向右看，没有东西；向左看是一个指针
（3）跳过括号，数组尺寸 5，右边没东西了
（4）向左看，数组元素为 int
所以：v 是一个指针，指向的是一个大小为5的数组，数组中元素类型为 int
=> v 是一个指向有 5 个 int 的数组的指针
*/

/* 函数指针 int (*func) ();
	  func   指针 -> 函数（返回值为 int）
*/

/* int (*v[]) ();
       v  数组 -> 数组的元素是指针 -> 指针执行的是函数 ->  函数的返回值是 int
   => v 是一个 int 型函数指针组成的数组
*/

/* int (*(*v)[]) ()
       v  指针  ->  指向的是数组  -> 数组中的元素是指针  ->  数组元素中的指针指向的是函数（返回值为 int）
   => v 是一个指向 int 型函数指针组成的数组的指针
*/
```

### 2. 加入 const 之后

`int const a;` 等价于 `const int a;`

但是加入指针之后，就有了变化

```cpp
/* int const *r;  指针常量
	r   指针  -> 指向的 int const
    (*r) 不能改变
    但 r 本身不是 const ，所以可以改变
*/

/* int * const r;  常量指针
    r 是个 const 
    r 是个 const 指针  ->  指向的 int
    r 不可以改变
*/
```



## 八、new / delete 与 malloc / free 

### 1. new / delete 是如何实现的

- new 的实现过程是：首先调用名为 `operator new` 的标准库函数，分配足够大的原始未类型化的内存，以保存指定类型的一个对象；接下来运行该类型的一个构造函数，用指定初始化构造对象；最后返回指向新分配并构造后的对象的指针
- delete 的实现过程：对指针指向的对象运行适当的析构函数；然后通过调用名为 `operator delete` 的标准库函数释放该对象所用内存

### 2. new / delete 与 malloc / free 相同点

都可以用于内存的动态申请和释放

### 3. new / delete 与 malloc / free 不同点

- new / delete 是 C++ 运算符；malloc / free 是 C/C++ 语言标准库函数

- new 自动计算要分配空间大小；malloc 需要手工计算

- new 是类型安全的；malloc 不是

  ```cpp
  int *p = new float[2]; //编译错误
  int *p = (int*)malloc(2 * sizeof(double));//编译无错误
  ```

- new 调用名为 `operator new` 的标准库函数，分配足够空间并调用相关对象的构造函数，delete 对指针所指对象运行适当的析构函数，然后通过调用名为 `operator delete` 的标准库函数释放该对象所用内存；malloc / free 均没有相关调用

- new / delete 不需要库文件支持；malloc / free 需要库文件支持

- new 是封装了 malloc，直接 free 不会报错，但是这只是释放内存，而不会析构对象

### 4. 既然有了 malloc/free，C++ 中为什么还需要 new/delete 呢？

在对非基本数据类型的对象使用的时候，对象创建的时候还需要执行构造函数，销毁的时候要执行析构函数。而 malloc / free 是库函数，是已经编译的代码，所以不能把构造函数和析构函数的功能强加给 malloc / free，所以 new/delete 是必不可少的。

### 5. 被 free 回收的内存是立即返还给操作系统吗？

不是的，被 free 回收的内存会首先被 ptmalloc 使用双链表保存起来，当用户下一次申请内存的时候，会尝试从这些内存中寻找合适的返回。这样就避免了频繁的系统调用，占用过多的系统资源。同时 ptmalloc 也会尝试对小块内存进行合并，避免过多的内存碎片。

### 6. delete p、delete[] p、allocator 都有什么作用

- 动态数组管理 new 一个数组的时，[] 中必须是一个整数，但是不一定时常量整数，普通数组必须是一个常量整数

- new 动态数组返回的并不是数组类型，而是一个元素类型的指针

- delete[] 时，数组中的元素按逆序的顺序进行销毁

- new 在内存分配上面有一些局限性，new 的机制是将内存分配和对象构造组合在一起，同样的，delete 也是将对象析构和内存释放组合在一起的。allocator 将这两部分分开进行，allocator申请一部分内存，不进行初始化对象，只有当需要的时候才进行初始化操作。

  > 补充知识：
  >
  > 分配器是**负责封装堆内存管理的对象**
  >
  > 每个容器实例中都有一个 Allocator 实例。它向分配器请求存储来存储元素。分配器应具备的基本成员函数如下：
  >
  > ```cpp
  > // T* allocate(size_t n); 分配足够的存储空间来存储 T 的 n 个实例，并返回指向它的指针
  > // void deallocate(T* p, size_t n) 释放分配的内存
  > // void construct(T* p, Args ... args); 使用 p 指向的 args 参数构造一个对象,该接口在C++20 中已被移除
  > // void destroy(T* p); 调用 p 指向的对象的析构函数，该接口在 C++20 中已被移除
  > ```

### 7. new[] / delete[] 是如何实现的

- 对于简单类型，new[] 计算好大小后调用 operator new；对于复杂数据结构，new[] 先调用 operator new[]分配内存，然后在 p 的前四个字节写入数组大小 n，然后调用 n 次构造函数。**针对复杂类型，new[] 会额外存储数组大小**
  - new 表达式调用一个名为 `operator new(operator new[])` 函数，分配一块足够大的、原始的、未命名的内存空间
  - 编译器运行相应的构造函数以构造这些对象，并为其传入初始值
  - 对象被分配了空间并构造完成，返回一个指向该对象的指针
- 针对简单类型，delete 和 delete[] 等同。假设指针 p 指向 new[] 分配的内存。因为要 4 字节存储数组大小，实际分配的内存地址为 [p-4]，系统记录的也是这个地址。delete[] 实际释放的就是 p-4 指向的内存。而delete 会直接释放 p 指向的内存，这个内存根本没有被系统记录，所以会崩溃

需要在 new[] 一个对象数组时，需要保存数组的维度，C++ 的做法是在分配数组空间时多分配了 4 个字节的大小，专门保存数组的大小，在 delete[] 时就可以取出这个保存的数，就知道了需要调用析构函数多少次了

### 8. new[] / delete[] 是如何实现的

malloc/free 的操作对象都是必须明确大小的，而且不能用在动态类上。new 和 delete 会自动进行类型检查和大小，malloc/free 不能执行构造函数与析构函数，所以动态对象它是不行的。

当然从理论上说使用 malloc 申请的内存是可以通过 delete 释放的。不过一般不这样写的。而且也不能保证每个C++ 的运行时都能正常。

### 9. malloc、realloc、calloc的区别

**malloc 函数**

```cpp
void* malloc(unsigned int num_size);
int *p = malloc(20 * sizeof(int));   // 申请 20 个 int 类型的空间；
```

**calloc 函数**

省去了人为空间计算；malloc 申请的空间的值是随机初始化的，calloc 申请的空间的值是初始化为 0 的

```cpp
void* calloc(size_t n,size_t size);
int *p = calloc(20, sizeof(int));
```

**realloc 函数**

给动态分配的空间分配额外的空间，用于扩充容量

```cpp
void realloc(void *p, size_t new_size);
```

### 10. malloc 与 free 的实现原理

- 在标准 C 库中，提供了 malloc/free 函数分配释放内存，这两个函数底层是由 brk、mmap、munmap 这些系统调用实现的


- brk 是将数据段 (.data) 的最高地址指针 _edata 往高地址推，mmap 是在进程的虚拟地址空间中（堆和栈中间，称为文件映射区域的地方）找一块空闲的虚拟内存。这两种方式分配的都是虚拟内存，没有分配物理内存。在第一次访问已分配的虚拟地址空间的时候，发生缺页中断，操作系统负责分配物理内存，然后建立虚拟内存和物理内存之间的映射关系；


- malloc 小于 128k 的内存，使用 brk 分配内存，将 _edata 往高地址推；malloc 大于 128k 的内存，使用mmap 分配内存，在堆和栈之间找一块空闲内存分配；brk 分配的内存需要等到高地址内存释放以后才能释放，而 mmap 分配的内存可以单独释放。当最高地址空间的空闲内存超过 128K（可由 M_TRIM_THRESHOLD 选项调节）时，执行内存紧缩操作（trim）。在上一个步骤 free 的时候，发现最高地址空闲内存超过 128K，于是内存紧缩。


- malloc 是从堆里面申请内存，也就是说函数返回的指针是指向堆里面的一块内存。操作系统中有一个记录空闲内存地址的链表。当操作系统收到程序的申请时，就会遍历该链表，然后就寻找第一个空间大于所申请空间的堆结点，然后就将该结点从空闲结点链表中删除，并将该结点的空间分配给程序。

> 2023-03-18 看不懂



## 九、#define、函数、typedef、const、内联函数

### 1. 宏定义 #define 和函数的区别

- 宏在预处理阶段完成替换，之后被替换的文本参与编译，相当于直接插入了代码，运行时**不存在函数调用**，执行起来更快；函数调用在运行时需要跳转到具体函数
- 宏定义属于在结构中插入代码，没有返回值；函数调用具有返回值
- 宏定义参数没有类型，不进行类型检查；函数参数具有类型，需要检查类型
- **宏定义不要再最后加分号**

### 2. 宏定义 #define 和 typedef 的区别

- #define 是**预处理器指令**；而 typedef 是 **C++ 关键字**

- #define 主要用来**定义常量，以及频繁使用的宏**，也可以用作别名；而 typedef 主要用来定义**数据类型的别名**

- #define 定义的别名只是简单的**文本替换，没有类型安全检查**；而 typedef 定义的别名是一个真正的类型，具有**类型安全检查**

- #define 不受作用域的约束，直到遇到 #undef 结束作用范围；而 typedef 存在作用域限制

- #define 不是语句，无需分号；typedef 定义的语句，需要分号

- 连续定义几个变量的时候，typedef 能够保证定义的所有变量均为同一类型；而 #define 则无法保证

  ```cpp
  #define PINT1 int*;
  P_INT1 p1,p2;  //即int *p1, p2;
  
  typedet int* PINT2;
  P_INT2 p1,p2;  //p1、p2 类型相同
  ```

### 3. 宏定义 #define 和 const 的区别

**编译阶段**

#define 是在编译的**预处理阶段**起作用；而 const 是在编译、运行的时候起作用

**安全性**

- #define 只做替换，不做类型检查和计算，也不求解，容易产生错误

- const 常量有数据类型，编译器可以对其进行类型安全检查

**内存占用**

- #define 只是将宏名称进行替换，在内存中会产生多份相同的备份；const 在程序运行中只有一份备份，且可以执行常量折叠，能将复杂的表达式计算出结果放到常量表

  > 补充知识：
  >
  > 2023-03-12不懂常量折叠，参看《[c++的常量折叠](https://blog.csdn.net/u011749929/article/details/107006922)》

- 宏定义的数据没有分配内存空间，只是替换掉；const 定义的变量只有值不能改变，但要分配内存空间

### 4. 宏定义 #define 和内联函数的区别

- 在使用时，宏只做简单字符串替换（编译前）；而内联函数可以进行参数类型检查（编译时），且具有返回值
- 内联函数在编译时直接将函数代码嵌入到目标代码中，省去函数调用的开销来提高效率，并且进行参数类型检查，具有返回值可以实现重载
- 宏定义使用时要注意书写（参数要括起来），否则容易出现歧义，内联函数不会产生歧义
- 内联函数有类型检测、语法判断等功能；而宏没有

**内联函数适用场景**

- 使用宏定义的地方都可以使用内联函数
- 内联函数作为类成员接口函数来读写类的私有成员或者保护成员，会提高效率



## 十、变量声明和定义的区别

- 声明仅仅是把变量的**声明的位置及类型提供给编译器**，并**不分配内存空间**；定义要在定义的地方为其分配存储空间
- 相同的**变量可以多处声明**（外部变量 extern），但**只能在一处定义**



## 十一、strlen 和 sizeof 区别

- **sizeof 是运算符**，并不是函数，结果在编译时得到而非运行时获得；**strlen 是处理字符的库函数**

- sizeof 的参数可以是任何数据的类型或者数据；strlen 的参数只能时候字符指针且结尾是 '\0' 的字符串

  > 数组作为参数传递给函数后，使用 sizeof 会产生问题
  >
  > ```cpp
  > #include <iostream>
  > #include <stdlib.h>
  > #include <string.h>
  >  
  > using namespace std;
  >  
  > void func(char s[], int n){
  >     cout<<"----------------- func -------------------"<<endl;
  >     cout<<"sizeof(s) = "<<sizeof(s)<<endl;
  >     cout<<"sizeof(s[0]) = "<<sizeof(s[0])<<endl;
  >     cout<<"The length of s is "<<sizeof(s)/sizeof(s[0])<<endl;
  >     cout<<"----------------- pointer -------------------"<<endl;
  >     char *p = NULL;
  >     cout<<"sizeof(p) = "<<sizeof(p)<<endl;
  > }
  >  
  > int main()
  > {
  >     cout<<"----------------- main -------------------"<<endl;
  >     char g[] = {'a', 'b', 'c', 'd', 'e'};
  >     cout<<"sizeof(g) = "<<sizeof(g)<<endl;
  >     cout<<"sizeof(g[0]) = "<<sizeof(g[0])<<endl;
  >     cout<<"The length of g is "<<sizeof(g)/sizeof(g[0])<<endl;
  >     func(g, 3);
  >     return 0;
  > }
  > 
  > /*
  > ----------------- main -------------------
  > sizeof(g) = 5
  > sizeof(g[0]) = 1
  > The length of g is 5
  > ----------------- func -------------------
  > sizeof(s) = 4
  > sizeof(s[0]) = 1
  > The length of s is 4
  > ----------------- pointer -------------------
  > sizeof(p) = 4
  > */
  > ```
  >
  > 在上面的输出中，第一部分 main 输出的信息符合预期。但是，将数组 g 作为参数传递到函数 func 时，再做同样的计算，结果就不一样了，为什么呢？
  >
  > 因为数组作为函数参数传递后，会退化为指针，所以计算 sizeof(s) = 4 ，实质等价于计算 sizeof(char *s) 的大小，指针变量大小为 4 字节（在 32 位平台上）。sizeof(s[0]) 计算的是第一个字符的字节大小，即为 1。我们可以看到指针变量 p 计算大小后也为 4。
  >
  > 所以在将数组作为参数传递到函数时，注意 sizeof() 的使用，最好的方式是一同传递一个数组元素个数的变量，比如上面例子中的 n。
  >
  > 参看《[C/C++ 数组作为参数传递到函数后，使用 sizeof 产生的问题](https://blog.csdn.net/nyist_zxp/article/details/116564651)》

- 因为 **sizeof 值在编译时确定**，所以不能用来得到动态分配（运行时分配）存储空间的大小



## 十二、a 和 &a 有什么区别

假设数组 `ina a[10]; int (*p)[10] = &a;` 其中：

- a 是数组名，是数组首元素地址，+1 表示地址值加上一个 int 类型的大小，如果 a 的值是 0x00000001，加 1 操作后变为 0x00000005。`*(a+1) = a[1]`
- &a 是数组的指针，其类型为 `int (*)[10]` ，其加 1 后，系统会认为是数组首地址加上整个数组的偏移（10 个 int 型变量），值为数组 a 尾后一个元素的地址
- 若 `int *p2 = (int *) p` ，此时输出 *p2 时，其值为 a[0] 的值，因为被转为 int * 类型，解引用时按照 int 类型大小来读取

```cpp
int main()
{
    int a[10] = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
    int (*p)[10] = &a;
    int *p2 = (int *)p;
    
    cout << "a: " << a << " a[0]: " << a[0] << " a+1: " << a+1 << endl;
    cout << "a+9: " << a+9 << endl;
    cout << "p: " << p << " p+1: " << p+1 << " *p: " << *p << endl;
    cout << "p2: " << p2 << " *p2: " << *p2 << endl;
    
    return 0;
}

/* 输出
a: 0x7ffefd0c3480 a[0]: 1 a+1: 0x7ffefd0c3484
a+9: 0x7ffefd0c34a4
p: 0x7ffefd0c3480 p+1: 0x7ffefd0c34a8 *p: 0x7ffefd0c3480
p2: 0x7ffefd0c3480 *p2: 1
*/

// 2023-03-13 疑问 p 和 *p 的输出为什么一样？
```



## 十三、C++ 语言和其他语言的区别

### 1. C++ 和 C 的区别

- C++ 中 new 和 delete 是对内存分配的运算符，取代了 C 中的 malloc 和 free
- 标准 C++ 中的字符串类取代了标准 C 函数库头文件中的字符数组处理函数（C 中没有字符串类型）
- C++ 中用来控制台输入输出的 iostream 类库替代了标准 C 中的 stdio 函数库
- C++ 中的 try/catch/throw 异常处理机制取代了标准 C 中的 setjmp() 和 longjmp() 函数
- C++ 中可以重载；C 语言不允许
- C++ 中允许变量定义语句在程序中的任何地方，只要在使用它之前即可；C 中必须要在函数开头部分
- C++ 中除了值和指针之外，新增了引用。引用型变量是其他变量的别名
- C++ 相对与 C 增加了一些关键字，比如 bool、using、dynamic_cast、namespace 等等

### 2. C++ 和 Python 的区别

- Python是一种脚本语言，是解释执行的，而 C++ 是编译语言，是需要编译后在特定平台运行的。python 可以很方便的跨平台，但是效率没有 C++ 高
- Python 使用缩进来区分不同的代码块，C++ 使用花括号来区分
- C++ 中需要事先定义变量的类型，而 Python 不需要，Python 的基本数据类型只有数字，布尔值，字符串，列表，元组等等
- Python 的库函数比 C++ 的多，调用起来很方便

### 3. C++ 和 Java 的区别

**语言特性**

- Java 语言给开发人员提供了更为简洁的语法；完全面向对象，由于 JVM 可以安装到任何的操作系统上，所以说它的可移植性强
- Java 语言中没有指针的概念，引入了真正的数组。不同于 C++ 中利用指针实现的“伪数组”，Java 引入了真正的数组，同时将容易造成麻烦的指针从语言中去掉，这将有利于防止在 C++ 程序中常见的因为数组操作越界等指针操作而对系统数据进行非法读写带来的不安全问题
- C++ 也可以在其他系统运行，但是需要不同的编码（这一点不如 Java，只编写一次代码，到处运行），例如对一个数字，在 windows 下是大端存储，在 unix 中则为小端存储。Java 程序一般都是生成字节码，在 JVM 里面运行得到结果
- Java 用接口 (Interface) 技术取代 C++ 程序中的抽象类。接口与抽象类有同样的功能，但是省却了在实现和维护上的复杂性

**垃圾回收**

- C++ 用析构函数回收垃圾，写 C 和 C++ 程序时一定要注意内存的申请和释放
- Java 语言不使用指针，内存的分配和回收都是自动进行的，程序员无须考虑内存碎片的问题

**应用场景**

- Java 在桌面程序上不如 C++ 实用，C++ 可以直接编译成 exe 文件，指针是 c++ 的优势，可以直接对内存的操作，但同时具有危险性 （操作内存的确是一项非常危险的事情，一旦指针指向的位置发生错误，或者误删除了内存中某个地址单元存放的重要数据，后果是可想而知的）

- Java 在 Web 应用上具有 C++ 无可比拟的优势，具有丰富多样的框架

- 对于底层程序的编程以及控制方面的编程，C++ 很灵活，因为有句柄的存在

  > 补充知识：
  >
  > 参考《[C++句柄](https://blog.csdn.net/qq_28350219/article/details/113373044)》



## 十四、C++ 中 struct 和 class 的区别

**相同点**

- 两者都拥有成员函数、公有和私有部分
- 任何可以使用 class 完成的工作，同样可以使用 struct 完成

**不同点**

- 两者中如果不对成员指定公私有，struct 默认时公有，class 则默认是私有
- class 默认是 private 继承；而 struct 默认是 public 继承

**引申：C++ 和 C 的 struct 区别**

- C 语言中：struct 是用户自定义数据类型（UDT）；C++ 中 struct 是抽象数据类型（ADT），支持成员函数的定义，（C++ 中的struct 能继承，能实现多态）

- C 中 struct 是没有权限设置的，且 struct 中只能是一些变量的集合体，可以封装数据却不可以隐藏数据，而且**成员不可以是函数**

- C++ 中，struct 增加了访问权限，且可以和类一样有成员函数，成员默认访问说明符为 public（为了与 C 兼容）

- struct 作为类的一种特例是用来自定义数据结构的。一个结构标记声明后，在 C 中必须在结构标记前加上 struct，才能做结构类型名（除：`typedef struct class{};` ）；C++ 中结构体标记（结构体名）可以直接作为结构体类型名使用，此外结构体 struct 在 C++ 中被当作类的一种特例

  > ```c
  > struct Books {
  >    char  title[50];
  >    char  author[50];
  >    char  subject[100];
  >    int   book_id;
  > };
  > 
  > int main(){
  >     struct Books Book;        /* 声明 Book1，类型为 Books */
  > }
  > ```

**引申：C/C++ 中 struct 和 union 区别**

- struct 和 union 都是由多个不同的数据类型成员组成，但在**任何同一时刻，union 中只存放了一个被选中的成员**，而 struct 的所有成员都存在。在 struct 中，各成员都占有自己的内存空间，它们是同时存在的。

- 一个 struct 变量的总长度等于所有成员长度之和（但需要注意内存对齐问题）。在 union 中，所有成员不能同时占用它的内存空间，它们不能同时存在。**union 变量的长度等于最长的成员的长度**。

- 对于 union 的不同成员赋值，将会对其它成员重写，原来成员的值就不存在了，而对于 struct 的不同成员赋值是互不影响的。



## 十五、C++ 中 const 和 static 的区别

### 1. static 

**不考虑类的情况**

- 所有**不加 static 的全局变量和函数具有全局可见性，可以在其他文件中使用**，加了 static 之后只能在**该文件所在的编译模块中使用**
- 默认初始化为 0，或者未初始化的全局静态变量和局部静态变量，都存在全局未初始化区
- 静态变量在函数内定义，始终存在，且只进行一次初始化具有记忆性，起作用范围与局部变量相同，函数退出后仍然存在，但不能使用

**考虑类的情况**

- static 成员变量：只与类关联，不与类的对象关联。定义时要分配空间，**不能在类声明中初始化**，必须在类定义体外部初始化，初始化时不需要标识为 static。可以被非 static 成员函数任意访问

- static 成员函数：**不具有 this 指针，无法访问类对象的非 static 成员变量和非 static 成员函数。不能被声明为 const，虚函数和 volatile**。可以被非 static 成员函数任意访问

  > 补充知识：
  >
  > 对于一个定义为 const 的函数，传递的是 const 的 this 指针，说明不能更改对象的属性，而对 static 成员的函数不需传递 this 指针，所有就不需要用 const 来修饰 static 的成员函数了！就说 const 属性的作用就是对被传递的 this 指针加以限定，而对 static 成员函数的调用根本不传递 this 指针，所有不需 const 来修饰 static 的成员函数

### 2. const 

**不考虑类的情况**

- const 常量在定义时必须初始化，之后无法更改

- const 形参可以接受 const 和非 const 类型的实参

- const 修饰变量也是于 static 有一样的隐藏作用。只能在该文件使用，其他文件不可以引用声明引用

  > ```cpp
  > void fun(const int& i) { //.... }
  > 
  > int main() {
  >         int a = 1;
  >         const int b = 2;
  >         fun(a);
  >         fun(b);
  > }
  > ```

**考虑类的情况**

- const 成员变量：只能通过构造函数**初始化列表进行初始化**，并且**必须有构造函数**。不同类对其 const 数据成员的值可以不同，所以**不能再类中声明时初始化**
- const 成员函数：常对象只能调用常成员函数。非 const 对象都可以调用。不可以改变非 mutable（用该关键字声明的变量可以在 const 成员函数中被修改）数据的值



## 十六、顶层 const 和 底层 const

### 1. 概念区分

- 顶层 const：指的是 const 修饰的**变量本身**是一个常量
- 底层 const：指的是 const 修斯和的**变量所指向的对象**是一个常量

```cpp
int a = 10;
int* const b1 = &a;        //顶层 const，b1 本身是一个常量
const int* b2 = &a;        //底层 const，b2 本身可变，所指的对象是常量
const int b3 = 20; 		   //顶层 const，b3 是常量不可变
const int* const b4 = &a;  //前一个 const 为底层，后一个为顶层，b4 不可变
const int& b5 = a;		   //用于声明引用变量，都是底层 const
```

### 2. 区分作用

- 执行对象拷贝时有限制，常量的底层 const 不能赋值给非常量的底层 const

  > ```cpp
  > int num_c = 3;
  > const int *p_c = &num_c;  //p_c为底层const的指针
  > //int *p_d = p_c;  //错误，不能将底层const指针赋值给非底层const指针
  > const int *p_d = p_c; //正确，可以将底层const指针复制给底层const指针
  > ```

- 使用命名的强制类型转换函数 const_cast 时，需要能够分辨底层 const 和顶层 const，因为 const_cast 只能改变运算对象的底层 const

  > ```cpp
  > int num_e = 4;
  > const int *p_e = &num_e;
  > //*p_e = 5;  //错误，不能改变底层 const 指针指向的内容
  > int *p_f = const_cast<int *>(p_e);  //正确，const_cast 可以改变运算对象的底层 const。但是使用时一定要知道num_e 不是 const 的类型。
  > *p_f = 5;  //正确，非底层 const 指针可以改变指向的内容
  > cout << num_e;  //输出 5
  > ```



## 十七、数组名和指针（指向数组首元素的指针）的区别

- 二者都可以通过增减偏移量来访问数组中的元素
- 数组名不是真正意义上的指针，可以理解为常指针，所以数组名没有自增、自减等操作
- 当数组名当作形参传递给调用函数后，就失去了原有特性，退化为一般指针，多了自增、自减操作， sizeof 运算符不能得到原数组的大小了



## 十八、final 和 override 的区别

### 1. override

当在父类中使用了虚函数时候，如果需要对这个虚函数进行重写，以下方法都可以：

```cpp
class A {
    virtual void foo();
};

class B : public A {
    void foo(); // OK
    virtual void foo(); // OK
    void foo() override; // OK
};
```

如果不使用 override，当手一抖将 `foo()` 写成了 `f00()` 会怎么呢？结果是编译器并不会报错，因为它并不知道你的目的是重写虚函数，而是把它当成了新的函数。如果这个虚函数很重要的话，那就会对整个程序不利。所以 override 的作用就出来了，它指定了子类的这个虚函数是重写的父类的，如果函数名拼写错误编译器无法通过：

```cpp
class A{
    virtual void foo();
};

class B : public A{
    virtual void f00(); // OK，这个函数是 B 新增的，不是继承的
    virtual void f0o() override; // Error, 加了 override 之后，这个函数一定是继承自 A 的，A 找不到就报错
};
```

### 2. final

当不希望某个类被继承，或不希望某个虚函数被重写，可以在类名和虚函数后添加 final 关键字，添加 final 关键字后被继承或重写，编译器会报错。

```cpp
class Base {
    virtual void foo();
};
 
class A : public Base {
    void foo() final; // foo 被override并且是最后一个override，在其子类中不可以重写
};

class B final : A {  // 指明B是不可以被继承的
    void foo() override; // Error: 在A中已经被final了
};
 
class C : B { // Error: B is final
};
```



## 十九、初始化

- 默认初始化
- 值初始化
- 拷贝初始化
- 直接初始化
- 列表初始化

### 1. 拷贝初始化和直接初始化

当用于类类型对象时，初始化的拷贝形式和直接形式有所不同：

- 直接初始化：直接调用与实参匹配的构造函数（在对象初始化时，通过括号给对象提供一定的参数，并且要求编译器使用普通的函数匹配来选择与我们提供的参数最匹配的构造函数）
- 拷贝初始化：总是调用拷贝构造函数（拷贝初始化首先使用指定构造函数创建一个临时对象，然后用拷贝构造函数将那个临时对象拷贝到正在创建的对象）
  - 使用赋值运算符定义变量
  - 将对象作为实参传递给一个非引用类型的形参
  - 返回类型为非引用类型的函数返回一个对象
  - 用花括号列表初始化一个数组中的元素或一个聚合类中的成员

```cpp
string str1("I am a string");//语句1 直接初始化
string str2(str1);//语句2 直接初始化，str1是已经存在的对象，直接调用拷贝构造函数对str2进行初始化
string str3 = "I am a string";//语句3 拷贝初始化，先为字符串”I am a string“创建临时对象，再把临时对象作为参数，使用拷贝构造函数构造str3
string str4 = str1;//语句4 拷贝初始化，这里相当于隐式调用拷贝构造函数，而不是调用赋值运算符函数
```

为了提高效率，允许编译器跳过创建临时对象这一步，直接调用构造函数构造要创建的对象，这样就完全等价于直接初始化（语句 1 和语句 3 等价），但是需要辨别两种情况：

- 当拷贝构造函数为 private 时：语句 3 和语句 4 在编译时会报错

- 使用 explicit 修饰构造函数时：如果构造函数存在隐式转换时，编译时会报错

  > 补充知识：
  >
  > C++ 中的 explicit 关键字主要是用来修饰类的构造函数，表明该构造函数是显式的，禁止单参数构造函数的隐式转换。
  >
  > 隐式转换：将构造函数**一个值**（其类型为构造函数对应的数据类型）转换为一个**类对象**。
  >
  > 如果C++类的构造函数**只有一个参数**，那么在编译的时候就会有一个缺省的转换操作：将该构造函数对应数据类型的数据转换为该类对象

### 2. 类成员初始化方式

- 赋值初始化：通过在函数体内进行赋值初始化
- 列表初始化：在冒号后使用初始化列表进行初始化

这两种方式的主要区别在于：

对于在函数体中初始化，是在所有的数据成员被分配内存空间后才进行的。

列表初始化是给数据成员分配内存空间时就进行初始化，就是说分配一个数据成员只要冒号后有此数据成员的赋值表达式（此表达式必须是括号赋值表达式），那么分配了内存空间后在进入函数体之前给数据成员赋值，就是说初始化这个数据成员此时函数体还未执行。

方法一是在构造函数当中做赋值的操作，而方法二是做纯粹的初始化操作。我们都知道，C++ 的赋值操作是会产生临时对象的。临时对象的出现会降低程序的效率。

> 补充知识：
>
> ![image-20230318115917333](D:\Data\笔记\c++\c++八股文.assets\image-20230318115917333.png)
>
> ![image-20230318115942976](D:\Data\笔记\c++\c++八股文.assets\image-20230318115942976.png)
>
> ![image-20230318120000248](D:\Data\笔记\c++\c++八股文.assets\image-20230318120000248.png)

### 3. 有哪些情况必须用到成员列表初始化？作用是什么？

**必须使用成员初始化的四种情况：**

- 当初始化一个引用成员时
- 当初始化一个常量成员时
- 当调用一个基类的构造函数，而它拥有一组参数时
- 当调用一个成员类的构造函数，而它拥有一组参数时

**成员初始化列表做了什么：**

- 编译器会一一操作初始化列表，以适当的顺序在构造函数之内安插初始化操作，并且在任何显示用户代码之前
- 初始化顺序是由类中的成员声明顺序决定的，不是由初始化列表的顺序决定的



## 二十、初始化和赋值的区别

- 对于简单类型来说，初始化和赋值没什么区别
- 对于类和复杂数据类型来说，初始化和赋值区别较大，具体如下

```cpp
class A {
public:
    int num1;
    int num2;
public:
    A (int a = 0, int b = 0): num1(a), num2(b) {}
    A (const A& a) {}
    // 重载 = 号操作符函数
    A& operator= (const A& a) {
        num1 = a.num1 + 1;
        num2 = a.num2 + 1;
        return *this;
    }
};

int main() {
    A a(1, 1);
    A a1 = a; // 拷贝初始化操作，调用拷贝构造函数
    A b;
    b = a; // 赋值操作，对象 a 中，num1 = 1, num2 = 1; 对象 b 中，num1 = 2, num2 = 2
    return 0;
}
```



## 二一、extern "C" 的用法

为了能够正确的在 C++ 代码中调用 C 语言的代码：在程序中加上 extern "C" 后，相当于告诉编译器这部分代码是 C 语言写的，因此要按照 C 语言进行编译，而不是 C++

**哪些情况下使用 extern "C"：**

- C++ 代码中调用 C 语言代码
- 在 C++ 中的头文件中使用
- 在多个人协同开发时，可能有人擅长 C 语言，有人擅长 C++

C++ 中调用 C 代码：

```cpp
#ifndef __MY_HANDLE_H__
#define __MY_HANDLE_H__

extern "C"{
    typedef unsigned int result_t;
    typedef void* my_handle_t;
    
    my_handle_t create_handle(const char* name);
    result_t operate_on_handle(my_handle_t handle);
    void close_handle(my_handle_t handle);
}
```

C 语言不支持 extern "C" 声明，在.c文件中包含了extern "C"时会出现编译语法错，使用 extern "C" 全部放于 cpp 程序相关文件或其头文件中

> 补充知识：
>
> `void*` 为 “无类型指针”，`void*` 指针变量可以指向任意变量的内存空间，任何类型指针变量都可以转为 `void*`
>
> ```cpp
> int num = 10;
> void* p = &num;
> int* p_num = &num;
> void* p1 = p_num;
> ```
>
> 不要对 void* 指针变量做解引用和算术运算，如果想通过 `void*` 指针变量取出内容，必须进行强制类型转换

总结如下形式：

1. C++ 调用 C 函数

   ```cpp
   //xx.h
   extern int add(...)
   
   //xx.c
   int add(){
       
   }
   
   //xx.cpp
   extern "C" {
       #include "xx.h"
   }
   ```

2. C 调用 C++ 函数

   ```cpp
   //xx.h
   extern "C"{
       int add();
   }
   
   //xx.cpp
   int add(){    
   }
   
   //xx.c
   extern int add();
   ```

   

## 二二、野指针和悬空指针

都是指向无效内存区域（不安全不可控）的指针，访问行为将会导致未定义行文。

### 1. 野指针

野指针：指的是未初始化过的指针

```cpp
int main(void) { 
    
    int* p;     // 未初始化
    std::cout<< *p << std::endl; // 未初始化就被使用
    
    return 0;
}
```

因此为了防止出错，对于指针初始化时都是赋值为 nullptr，这样在使用时编译器就不会直接报错，产生非法内存访问

### 2. 悬空指针

悬空指针：指针最初指向的内存已经被释放了的一种指针

```cpp
int main(void) { 
  int* p = nullptr;
  int* p2 = new int;
  
  p = p2;

  delete p2;
}
```

此时 p 和 p2 就是悬空指针，指向的内存已经被释放。继续使用这两个指针，行为不可预料。需要设置为 `p=p2=nullptr`。此时再使用，编译器会直接保错。 

避免野指针比较简单，但悬空指针比较麻烦。C++ 引入了智能指针，C++ 智能指针的本质就是避免悬空指针的产生。

### 3. 产生原因及解决办法

野指针：指针变量未及时初始化 => 定义指针变量及时初始化，要么置空

悬空指针：指针 free 或 delete 之后没有及时置空 => 释放操作后立即置空



## 二三、C 和 C++ 的内存安全

### 1. 什么是类型安全

类型安全很大程度上可以等价于内存安全，类型安全的代码不会试图访问自己没有授权的内存区域。“类型安全” 常被用来形容编程语言，其根据在于该门编程语言是否提供保障类型安全的机制；有的时候也用 “类型安全” 形容某个程序，判别的标准在于该程序是否隐含类型错误。

类型安全的编程语言与类型安全的程序之间，没有必然联系。好的程序员可以使用类型不那么安全的语言写出类型相当安全的程序，相反的，差一点的程序员可能使用类型相当安全的语言写出类型不太安全的程序。

绝对类型安全的编程语言暂时还没有。

### 2. C 的类型安全

C 只在局部上下文中表现出类型安全，比如试图从一种结构体的指针转换成另一种结构体的指针时，编译器将会报告错误，除非使用显式类型转换。然而，C 中相当多的操作是不安全的。

**printf 格式输出**

```cpp
#include <stdio.h>

int main()
{
    printf("整型输出: %d\n", 10);
    printf("浮点型输出: %f\n", 10);

    return 0;
}

/* 输出格式
整型输出: 10
浮点型输出: 0.000000
*/
```

上述代码中，使用 `%d` 控制整型数字的输出，没有问题，但是改成 `%f` 时，明显输出错误，再改成 `%s` 时，运行直接报 segmentation fault 错误

**malloc 函数的返回值**

malloc 是 C 中进行内存分配的函数，它的返回类型是 `void*` 即空类型指针，常常有这样的用法
`char* pStr = (char*)malloc(100 * sizeof(char))`，这里明显做了显式的类型转换。

类型匹配尚且没有问题，但是一旦出现 `int* pInt = (int*)malloc(100 * sizeof(char))` 就很可能带来一些问题，而这样的转换 C 并不会提示错误。

### 3. C++ 的类型安全

C++ 提供了一些新的机制保障类型安全：

- 操作符 new 返回的指针类型严格与对象匹配，而不是 `void*`
- C 中很多以 `void*` 为参数的函数可以改写为 C++ 模板函数，而模板函数是支持类型检查的
- 引入 const 关键字代替 #define constants，const 是有类型、有作用域的；而 #define constants 只是简单的文本替换
- 一些 #define 宏可以改写为 inline 函数，结合函数的重载，可在类型安全的前提下支持多种类型，当然改写为模板也能保证类型安全
- C++ 提供 dynamic_cast 关键字，使得转换过程更加安全，因为 dynamic_cast 比 static_cast 涉及更多具体的类型检查

**使用 `void *` 进行类型转换**

```cpp
#include <iostream>
using namespace std;

int main() {
    int i = 5;
    void* pInt = &i;
    double d = (*(double *)pInt);
    cout << "转换后输出: " << d << endl;

    return 0;
}

/*
输出：
转换后输出: -7.04429e+258
*/
```

**不同类型指针之间转换**

```cpp
#include<iostream>
using namespace std;
 
class Parent{};
class Child1 : public Parent
{
public:
	int i;
	Child1(int e):i(e){}
};
class Child2 : public Parent
{
public:
	double d;
	Child2(double e):d(e){}
};

int main()
{
	Child1 c1(5);
	Child2 c2(4.1);
	Parent* pp;
	Child1* pc1;
 	
	pp = &c1;
	pc1 = (Child1*)pp;   // 类型向下转换 强制转换，由于类型仍然为Child1*，不造成错误
	cout << pc1->i << endl;  // 输出：5
 
	pp = &c2;
	pc1 = (Child1*)pp;   // 强制转换，且类型发生变化，将造成错误
	cout << pc1->i << endl;  // 输出：1717986918
	return 0;
}
```

上面两个例子之所以引起类型不安全的问题，是因为程序员使用不得当。第一个例子用到了空类型指针 `void*`，第二个例子则是在两个类型指针之间进行强制转换。因此，想保证程序的类型安全性，应尽量避免使用空类型指针 `void*` ，尽量不对两种类型指针做强制转换。



## 二四、C++ 中的重载、重写和隐藏的区别

### 1. 重载（overload）

重载是指在**同一范围定义中的同名成员函数才存在重载关系**。主要特点是函数名相同，参数类型和数目有所不同，但仅仅依靠返回值不同不能称之为重载。重载和函数成员是否是虚函数无关。

```cpp
class A{
    // ...
    virtual int fun();
    void fun(int);
    void fun(double, double);
    static int fun(char);
    // ...
}
```

### 2. 重写（override）

重写指的是在派生类中覆盖基类中的同名函数，**重写就是重写函数体，要求基类函数必须是虚函数**且与基类的虚函数有相同的参数个数、参数类型和返回值类型

```cpp
// 父类
class A{
public:
    virtual int fun(int a){}
}

// 子类
class B : public A{
public:
    // 重写,一般加 override 可以确保是重写父类的函数
    virtual int fun(int a) override{}
}
```

**重载和重写的区别**

- 重写是父类和子类之间的垂直关系，重载是不同函数之间的水平关系
- 重写要求参数列表相同，重载则要求参数列表不同，返回值不要求
- 重写关系中，调用方法根据对象类型决定，重载根据调用时实参表与形参表的对应关系来选择函数体

### 3. 隐藏（hide）

隐藏指的是某些情况下，派生类中的函数屏蔽了基类中的同名函数，包括以下情况：

- 两个函数参数相同，但是基类函数不是虚函数。此时和重写的区别在于基类函数是否是虚函数。

  ```cpp
  // 父类
  class A {
  public:
      void fun(int a) {
  		cout << "A中的fun函数" << endl;
  	}
  };
  
  // 子类
  class B : public A {
  public:
      // 隐藏父类的 fun 函数
      void fun(int a) {
  		cout << "B中的fun函数" << endl;
  	}
  };
  
  int main() {
      B b;
      b.fun(2); // 调用的是 B 中的 fun 函数
      b.A::fun(2); // 调用 A 中 fun 函数
      return 0;
  }
  ```

- 两个函数参数不同，无论基类函数是不是虚函数，都会被隐藏。此时和重载的区别在于两个函数不在同一个类中

  ```cpp
  // 父类
  class A{
  public:
      virtual void fun(int a){
  		cout << "A中的fun函数" << endl;
  	}
  };
  
  // 子类
  class B : public A{
  public:
      // 隐藏父类的 fun 函数
     virtual void fun(char* a){
  	   cout << "A中的fun函数" << endl;
     }
  };
  
  int main(){
      B b;
      b.fun(2); // 报错，调用的是 B 中的 fun 函数，参数类型不对
      b.A::fun(2); // 调用 A 中 fun 函数
      return 0;
  }
  ```

  补充一下：

  ```cpp
  // 父类
  class A {
  public:
      virtual void fun(int a) { // 虚函数
          cout << "This is A fun " << a << endl;
      }  
      void add(int a, int b) {
          cout << "This is A add " << a + b << endl;
      }
  };
  
  // 子类
  class B: public A {
  public:
      void fun(int a) override {  // 覆盖
          cout << "this is B fun " << a << endl;
      }
      void add(int a) {   // 隐藏
          cout << "This is B add " << a + a << endl;
      }
  };
  
  int main() {
      // 基类指针指向派生类对象时，基类指针可以直接调用到派生类的覆盖函数，也可以通过 :: 调用到基类被覆盖的虚函数；而基类指针只能调用基类的被隐藏函数，无法识别派生类中的隐藏函数。
  
      A *p = new B();
      p->fun(1);      // 调用子类 fun 覆盖函数
      p->A::fun(1);   // 调用父类 fun
      p->add(1, 2);
      // p->add(1);      // 错误，识别的是 A 类中的 add 函数，参数不匹配
      // p->B::add(1);   // 错误，无法识别子类 add 函数
      return 0;
  }
  ```



## 二五、构造函数

- 默认构造函数
- 初始化构造函数（有参数）
- 拷贝构造函数
- 移动构造函数
- 委托构造函数
- 转换构造函数

**移动构造函数：**

当临时对象在被复制后，就不再利用了。我们完全可以把临时对象的资源直接移动，这样就避免了多余的复制操作。

C++11 标准中提供了一种新的构造方法：移动构造。C++11 之前如果要对源对象的状态转移到目标对象只能通过复制。在某些情况下我们没必要复制对象——只需要移动它们。

```cpp
class_name ( class_name && )
```

**委托构造函数：**

类中往往有多个构造函数，只是参数表和初始化列表不同，其初始化算法都是相同的，这时，为了避免代码重复，可以使用委托构造函数。

委托构造函数使用类的其他构造函数执行初始化过程

```cpp
Clock(int newH, int newM, int newS): hour(newH), minute(newM), second(newS){}
Clock(): Clock(0, 0, 0) { }
```

**转换构造函数：**

如果构造函数可以只传递一个参数，那么这个构造函数又叫做类型转换构造函数。常见于把其他类型转换为该类型。

```cpp
#include <iostream>
using namespace std;

class Student {
public:
    Student() { // 默认构造函数，没有参数
        this->age = 20;
        this->num = 1000;
    };  
    Student(int a, int n):age(a), num(n) {};  // 初始化构造函数，有参数和参数列表
    Student(const Student& s){  // 拷贝构造函数，这里与编译器生成的一致
        this->age = s.age;
        this->num = s.num;
    }; 
    Student(int r) {    // 转换构造函数,形参是其他类型变量，且只有一个形参
        this->age = r;
		this->num = 1002;
    };
    ~Student() {}
public:
    int age;
    int num;
};

int main() {
    Student s1;
    Student s2(18,1001);
    int a = 10;
    Student s3(a);
    Student s4(s3);
    
    printf("s1 age:%d, num:%d\n", s1.age, s1.num);
    printf("s2 age:%d, num:%d\n", s2.age, s2.num);
    printf("s3 age:%d, num:%d\n", s3.age, s3.num);
    printf("s2 age:%d, num:%d\n", s4.age, s4.num);
    return 0;
}

// 运行结果
// s1 age:20, num:1000
// s2 age:18, num:1001
// s3 age:10, num:1002
// s2 age:10, num:1002
```

- 默认构造函数和初始化构造函数在定义类的对象，完成对象的初始化工作
- 复制构造函数用于复制本类的对象
- 转换构造函数用于将其他类型的变量，隐式转换为本类对象

### 1. 什么情况下会调用拷贝构造函数

- 用类的一个实例化对象去初始化另一个对象的时候

- 函数的参数是类的对象时（非引用传递）

  > 调用函数时先根据传入的实参产生临时对象，再用拷贝构造去初始化这个临时对象，在函数中与形参对应，函数调用结束后析构临时对象

- 函数的返回值是函数体内局部对象的类的对象时，此时虽然发生（Named return Value优化）NRV 优化，但是由于返回方式是值传递，所以会在返回值的地方调用拷贝构造函数

  > 即使发生 NRV 优化的情况下，Linux + g++的环境是不管值返回方式还是引用方式返回的方式都不会发生拷贝构造函数，而Windows + VS2019在值返回的情况下发生拷贝构造函数，引用返回方式则不发生拷贝构造函数。
  >
  > 在 C++ 编译器发生 NRV 优化，如果是引用返回的形式则不会调用拷贝构造函数，如果是值传递的方式依然会发生拷贝构造函数。
  >
  > 
  >
  > 理论的执行过程是：产生临时对象，调用拷贝构造函数把返回对象拷贝给临时对象，函数执行完先析构局部变量，再析构临时对象， 依然会调用拷贝构造函数


### 2. 派生类构造函数的执行顺序

① 虚基类的构造函数（多个虚基类则按继承的顺序执行构造函数）

② 基类的构造函数（多个普通基类也按照继承的顺序执行构造函数）

③ 类类型的成员对象的构造函数（按照初始化顺序）

④ 派生类自己的构造函数



## 二六、浅拷贝和深拷贝

### 1. 浅拷贝

浅拷贝（位拷贝）：只是拷贝了基本类型的数据，而引用类型数据复制后也是会发生引用，我们把这种拷贝叫做 “浅拷贝（浅复制）”，换句话说浅复制仅仅是指向被复制的内存地址，如果原地址中对象被改变了，那么浅拷贝出来的对象也会相应改变

### 2. 深拷贝

**深拷贝不仅拷贝值，还开辟出一块新的空间用来存放新的值**，即使原先的对象被析构掉，释放内存了也不会影响到深拷贝得到的值。在自己实现拷贝赋值的时候，如果有指针变量的话是需要自己实现深拷贝的。

```cpp
#include <iostream>  
#include <string.h>
using namespace std;
 
class Student {
private:
	int num;
	char *name;
public:
	Student() {
        name = new char(20);
		cout << "Student" << endl;
    };
	~Student() {
        cout << "~Student " << &name << endl;
        delete name;
        name = NULL;
    };
	Student(const Student &s) {  // 拷贝构造函数
        // 浅拷贝，当对象的name和传入对象的name指向相同的地址
        name = s.name;
        // 深拷贝
        // name = new char(20);
        // memcpy(name, s.name, strlen(s.name));
        cout << "copy Student" << endl;
    };
};
 
int main() {
	{ // 花括号让s1和s2变成局部对象，方便测试
		Student s1;
		Student s2(s1);// 复制对象
	}
	system("pause");
	return 0;
}

//浅拷贝执行结果：
//Student
//copy Student
//~Student 0x7fffed0c3ec0
//~Student 0x7fffed0c3ed0
//*** Error in `/tmp/815453382/a.out': double free or corruption (fasttop): 0x0000000001c82c20 ***

//深拷贝执行结果：
//Student
//copy Student
//~Student 0x7fffebca9fb0
//~Student 0x7fffebca9fc0
```

从执行结果可以看出，浅拷贝在对象的拷贝创建时存在风险，即被拷贝的对象析构释放资源之后，拷贝对象析构时会再次释放一个已经释放的资源，深拷贝的结果是两个对象之间没有任何关系，各自成员地址不同。



## 二七、public、protected 和 private 访问权限和继承权限的区别

- public 的变量和函数在类的内部外部都可以访问
- protected 的变量和函数只能在类的内部和其派生类中访问
- private 修饰的元素只能在类内访问

### 1. 访问权限

| 访问权限  | 外部 | 派生类 | 内部 |
| --------- | ---- | ------ | ---- |
| public    | √    | √      | √    |
| protected | ×    | ×      | √    |
| private   | ×    | ×      | √    |

### 2. 继承权限

- 派生类继承自基类的成员权限有四种状态：public、protected、private 和不可见
- 派生类对基类成员的访问权限取决于两点：① 继承方式；② 基类成员在基类中的访问权限
- 派生类对基类成员的访问权限是取以上两点中的**更小的访问权限**

public 继承  +   private 权限  =>   private

private 继承  +   protected 权限  =>   private

private 继承 + private 成员 => 不可见



## 二八、如何用代码判断大小端存储

大端存储：字数据的高字节存储在低地址中

小端存储：字数据的低字节存储在低地址中

**在Socket编程中，往往需要将操作系统所用的小端存储的IP地址转换为大端存储，这样才能进行网络传输**

例如：32bit 的数字 0x12345678

![image-20230316202148816](D:\Data\笔记\c++\c++八股文.assets\image-20230316202148816.png)

```cpp
#include <iostream>
using namespace std;
int main()
{
    int a = 0x1234;
    // 由于int和char的长度不同，借助int型转换成char型，只会留下低地址的部分
    char c = (char)(a);
    
    if (c == 0x12)
        cout << "big endian" << endl;
    else if(c == 0x34)
        cout << "little endian" << endl;
}
```



## 二九、volatile、mutable 和 explicit 关键字的用法

### 1. volatile

volatile 关键字是一种类型修饰符，用**它声明的类型变量表示可以被某些编译器未知的因素更改**。比如：操作系统、硬件或者其它线程等。遇到这个关键字声明的变量，编译器对访问该变量的代码就不再进行优化，从而可以提供对特殊地址的稳定访问。

volatile 的本意是“易变的”，用于提示编译器使用 volatile 声明的变量随时有可能改变，因此编译器在代码编译时就不会对该变量进行某些激进的优化 ，故而编译生成的程序在每次存储或读取该变量时，都会直接从内存地址中读取数据，而不是读寄存器内的备份。即使它前面的指令刚刚从该处读取过数据。**多线程中被几个任务共享的变量需要定义为 volatile 类型。**

例如下面这个例子：

```cpp
#include <iostream>
#include <chrono>
void delay() {
    using namespace std::chrono;
    const auto start = std::chrono::system_clock::now();
    // volatile int i = INT_MAX;
    int i = INT_MAX;
    while (i--){}
    std::cout << duration_cast<microseconds>(std::chrono::system_clock::now() - start).count() << std::endl;

}

int main() {
    delay();
    return 0;
}

// 结果输出
// 0

// 如果加上 volatile 关键字给变量 i
// 结果输出
// 4193990 ns
```

**volatile 指针**

volatile 指针和 const 修饰词类似，const 有常量指针和指针常量的说法，volatile 也有相应的概念

```cpp
volatile char* vpch;
char* volatile pchv;
```

- 可以把一个非 volatile int 赋给 volatile int，但是不能把非 volatile 对象赋给一个 volatile 对象。
- 除了基本类型外，对用户定义类型也可以用 volatile 类型进行修饰。
- C++ 中一个有 volatile 标识符的类只能访问它接口的子集，一个由类的实现者控制的子集。用户只能用 const_cast 来获得对类型接口的完全访问。此外，volatile 向 const 一样会从类传递到它的成员。

**多线程下的 volatile**

有些变量是用 volatile 关键字声明的。当两个线程都要用到某一个变量且该变量的值会被改变时，应该用 volatile声明，**该关键字的作用是防止优化编译器把变量从内存装入 CPU 寄存器中。**如果变量被装入寄存器，那么两个线程有可能一个使用内存中的变量，一个使用寄存器中的变量，这会造成程序的错误执行。

### 2. mutable

mutable 的意思是 “可变的、易变的”，跟 constant（C++ 中的 const）是反义词。在 C++ 中，mutable 也是为了突破 const 的限制而设置的。被 mutable 修饰的变量，将永远处于可变的状态，即使在一个 const 函数中。如果类的成员函数不会改变对象的状态，那么这个成员函数一般会声明成const的。但是，有些时候，我们需要**在const函数里面修改一些跟类状态无关的数据成员，那么这个函数就应该被mutable来修饰，并且放在函数后后面关键字位置**。

```cpp
class person {
    int m_A;
    mutable int m_B; // 特殊变量 在常函数里值也可以被修改
public:
    void add() const { // 在函数里不可修改 this 指针指向的值 常量指针
        m_A=10; //错误  不可修改值，this 已经被修饰为常量指针
        m_B=20; //正确
    }
};

int main() {
    const person p; //修饰常对象 不可修改类成员的值
    p.m_A=10; //错误，被修饰了指针常量
    p.m_B=200; //正确，特殊变量，修饰了 mutable
}
```

### 3. explicit

explicit 关键字用来修饰类的构造函数，被修饰的构造函数的类，不能发生相应的隐式类型转换，只能以**显示的方式进行类型转换**

- explicit 关键字只能用于类内部的构造函数声明上
- explicit 关键字作用于单个参数的构造函数
- 被 explicit 修饰的构造函数的类，不能发生相应的隐式类型转换



## 三十、C++ 中有几种类型的 new

在 C++ 中，new 有三种典型的使用方法：plain new，nothrow new 和 placement new

### 1. plain new

plain new 就是我们常用的 new，在 C++ 中定义如下：

```cpp
void* operator new(std::size_t) throw(std::bad_alloc);
void operator delete(void *) throw();
```

因此 plain new 在空间分配失败的情况下，抛出异常 `std::bad_alloc` 而不是返回 NULL，因此通过判断返回值是否为 NULL 是徒劳的，举个例子：

```cpp
#include <iostream>
#include <string>
using namespace std;

int main() {
    try {
        char *p = new char[10e11];
        delete p;
    } catch (const std::bad_alloc &ex) {
        cout << ex.what() << endl;
    }
    
    return 0;
}

// 输出结果
// bad allocation
```

### 2. nothrow new

nothrow new 在空间分配失败的情况下是不抛出异常，而是返回 NULL，定义如下：

```cpp
void * operator new(std::size_t, const std::nothrow_t&) throw();
void operator delete(void*) throw();
```

举个例子：

```cpp
#include <iostream>
#include <string>
using namespace std;

int main() {
	char *p = new(nothrow) char[10e11];
	if (p == NULL) {
		cout << "alloc failed" << endl;
	}
	delete p;
	return 0;
}

// 运行结果
// alloc failed
```

### 3. **placement** new

这种 new 允许在一块已经分配成功的内存上重新构造对象或对象数组，placement new 不用担心内存分配失败，因为它根本不分配内存，它做的唯一一件事就是调用对象的构造函数。定义如下：

```cpp
void* operator new(size_t, void*);
void operator delete(void*, void*);
```

使用 placement new 需要注意两点：

- placement new 的主要用途就是**反复使用一块较大的动态分配的内存来构造不同类型的对象或者它们的数组**
- placement new 构造起来的对象数组，要显示的调用它们的析构函数来销毁（析构函数并不释放对象的内存），千万不要使用 delete，这是因为 placement new 构造起来的对象或数组大小并不一定等于原来分配的大小，使用 delete 会造成内存泄漏或者之后释放内存时出现运行时错误。

```cpp
#include <iostream>
#include <string>
using namespace std;
class ADT {
	int i;
	int j;
public:
	ADT() {
		i = 10;
		j = 100;
		cout << "ADT construct i=" << i << "j="<<j <<endl;
	}
	~ADT() {
		cout << "ADT destruct" << endl;
	}
};
int main() {
    // sizeof 可以省略括号
	char *p = new(nothrow) char[sizeof ADT + 1];
	if (p == NULL) {
		cout << "alloc failed" << endl;
	}
	ADT *q = new(p) ADT;  // placement new:不必担心失败，只要 p 所指对象的的空间足够 ADT 创建即可
	// delete q;  // 错误!不能在此处调用 delete q;
	q->ADT::~ADT();  // 显示调用析构函数
	delete[] p;
	return 0;
}

// 输出结果：
// ADT construct i=10j=100
// ADT destruct
```



## 三一、C++ 的异常处理方法

在程序执行过程中，由于程序员的疏忽或是系统资源紧张等因素都有可能导致异常，任何程序都无法保证绝对的稳定，常见的异常有：

- 数组下标越界
- 除法计算时除数为 0
- 动态分配空间时空间不足
- ...

如果不及时对这些异常进行处理，程序多数情况下都会崩溃

### 1. try、throw 和 catch 关键字

C++ 中的异常处理机制主要使用 try、throw 和 catch 三个关键字，其在程序中的用法如下：

```cpp
#include <iostream>
using namespace std;
int main() {
    double m = 1, n = 0;
    try {
        cout << "before dividing." << endl;
        if (n == 0)
            throw - 1;  //抛出 int 型异常
        else if (m == 0)
            throw - 1.0;  //拋出 double 型异常
        else
            cout << m / n << endl;
        cout << "after dividing." << endl;
    }
    catch (double d) {
        cout << "catch (double)" << d << endl;
    }
    catch (...) {
        cout << "catch (...)" << endl;
    }
    cout << "finished" << endl;
    return 0;
}

// 运行结果
// before dividing.
// catch (...)
// finished
```

catch 根据 throw 抛出的数据类型进行精确捕获（不会出现类型转换），如果匹配不到就直接报错，可以使用catch(...) 的方式捕获任何异常（不推荐）。

当然，如果 catch 了异常，当前函数如果不进行处理，或者已经处理了想通知上一层的调用者，可以在 catch 里面再 throw 异常。

### 2. 函数的异常声明列表

有时候，程序员在定义函数的时候知道函数可能发生的异常，可以在函数声明和定义时，指出所能抛出异常的列表，写法如下：

```cpp
int fun() throw(int,double,A,B,C) {...};
```

这种写法表名函数可能会抛出 int、double 型或者 A、B、C 三种类型的异常，如果 throw 中为空，表明不会抛出任何异常，如果没有 throw 则可能抛出任何异常

### 3. C++ 标准异常类

C++ 标准库中有一些类代表异常，这些类都是从 exception 类派生而来的，如下图：

<img src="D:\Data\笔记\c++\c++八股文.assets\image-20230317112023672.png" alt="image-20230317112023672" style="zoom:67%;" />

- bad_typeid：使用 typeid 运算符，如果其操作数是一个多态类的指针，而该指针的值为 NULL，则会拋出此异常

  > 补充知识：
  >
  > typeid 运算符：typeid 运算符用来获取一个表达式的类型信息
  >
  > - 对于基本类型的数据，类型信息所包含的内容比较简单，主要是指数据的类型。
  > - 对于类类型的数据，类型信息是指对象所属的类、所包含的成员、所在的继承关系等。
  >
  > typeid 的操作对象既可以是表达式，也可以是数据类型。typeid 会把获取到的类型信息保存到一个 type_info 类型的对象里面，并返回该对象的常引用；当需要具体的类型信息时，可以通过成员函数来提取。
  >
  > ```cpp
  > #include <iostream>
  > #include <typeinfo>
  > using namespace std;
  > 
  > class A {
  > public:
  >   virtual ~A();
  > };
  > 
  > int main() {
  > 	A* a = NULL;
  > 	try {
  >   		cout << typeid(*a).name() << endl; // Error condition
  >   	}
  > 	catch (bad_typeid){
  >   		cout << "Object is NULL" << endl;
  >   	}
  >     return 0;
  > }
  > 
  > // 运行结果
  > // bject is NULL
  > ```

- bad_cast：在用 dynamic_cast 进行从多态基类对象（或引用）到派生类的引用的强制类型转换时，如果转换是不安全的，则会拋出此异常

- bad_alloc：在用 new 运算符进行动态内存分配时，如果没有足够的内存，则会引发此异常
- out_of_range：用 vector 或 string 的 at 成员函数根据下标访问元素时，如果下标越界，则会拋出此异常



## 三二、static

### 1. static 的用法和作用

**作用一：隐藏（static 函数，static 变量均可）**

当同时编译多个文件时，所有未知 static 前缀的全局变量和函数都是全局可见性。

**作用二：保持变量内容的持久**

存储在静态数据区的变量会在程序刚开始运行时就完成初始化，也是唯一的一次初始化。共有两种变量存储在静态存储区：全局变量和static变量，只不过和全局变量比起来，static可以控制变量的可见范围，说到底static还是用来隐藏的。

**作用三：默认初始化为 0（static变量）**

其实全局变量也具备这一属性，因为全局变量也存储在静态数据区。在静态数据区，内存中所有的字节默认值都是 0x00，某些时候这一特点可以减少程序员的工作量。

**作用四：类成员声明为 static 的作用如下**

- 函数体内 static 变量的作用范围为该函数体，不同于 auto 变量，该变量的内存只被分配一次，因此其值在下次调用时仍维持上次的值
- 在模块内的 static 全局变量可以被模块内所有函数访问，但不能被模块外其它函数访问。在模块内的 static 函数只可被这一模块内的其他函数调用，这个函数的使用范围被限制在声明它的模块内
- 在类中的 static 成员变量属于整个类所拥有，对类的所有对象只有一份拷贝。
- static 修饰的变量先于对象存在，所以 static 修饰的变量要在类外初始化
- 由于 static 修饰的类成员属于类，不属于对象，因此 static 类成员函数是没有 this 指针的，this 指针是指向本对象的指针。所以 **static 类成员函数不能访问非 static 的类成员，只能访问 static 修饰的类成员**
- **static 成员函数不能被 virtual 修饰**，static 成员不属于任何对象或实例，所以加上 virtual 没有任何实际意义；静态成员函数没有 this 指针，虚函数的实现是为每一个对象分配一个 vptr 指针，而 vptr 是通过 this 指针调用的，所以不能为 virtual；虚函数的调用关系，this->vptr->ctable->virtual function

### 2. 静态变量什么时候初始化

- 初始化只有一次，但可以多次赋值，在主程序之前，编译器已经为其分配好了内存
- 静态局部变量和全局变量一样，数据都存放在全局区域，所以在主程序之前编译器已经为其分配好了内存，但在 C 和 C++ 中静态局部变量的初始化节点又有点不太一样：
  - 在 C 中，初始化发生在代码执行之前，编译阶段分配好内存之后就会进行初始化，所以我们看到在 C 语言中无法使用变量对静态局部变量进行初始化，在程序运行结束后变量所处的全局内存会被全部回收
  - 在 C++ 中，初始化时在执行相关代码时才会进行初始化，主要是由于 C++ 引入对象后，要进行初始化必须执行相应构造函数和析构函数，在构造函数或析构函数中经常会需要进行某些程序中需要进行的特定操作，并非简单地分配内存。所以 **C++ 标准定为全局或静态对象是有首次用到时才会进行构造，并通过atexit() 来管理。在程序结束，按照构造顺序反方向进行逐个析构**。所以在 C++ 中是可以使用变量对静态局部变量进行初始化的



## 三三、形参和实参的区别

- 形参变量只有在被调用时才分配内存空间，在调用结束后即刻释放所分配的内存单元。因此，**形参只有在函数内部有效**，函数调用结束返回主调函数后则不能再使用该形参变量
- 实参可以是常量、变量、表达式和函数等，无论实参是何种类型的量，**在进行函数调用时，它们都必须具有确定的值**，以便把这些值传送给形参。因此应先用赋值、输入等办法使实参获得确定值，会产生一个临时变量。
- 实参和形参在数量上、类型上、顺序上应严格一致，否则会发生 “类型不匹配” 的错误
- 函数调用中发生的数据传送是单向的
- 当形参和实参不是指针类型时，在该函数运行时，形参和实参是不同的变量，他们在内存中位于不同的位置，形参将实参的内容复制一份，在该函数运行结束的时候形参被释放，而实参内容不会改变



## 三四、const 关键字的作用有哪些

- 阻止一个变量被改变，可以使用 const 关键字。在定义该 const 变量时，通常需要对它进行初始化。
- 对指针来说，可以指定指针本身为 const，也可以指定指针所指的数据为 const，或二者同时指定为 const
- 在一个函数声明中，const 可以修饰形参，表明它是一个输入参数，在函数内部不能改变其值
- 对于类的成员函数，若指定其为 const 类型，则表明其是一个常函数，不能修改类的成员变量，类的常对象只能访问类的常成员函数。
- 对于类的成员函数，有时候必须指定其返回值为 const 类型，以使得其返回值不为 “左值”
- const 成员函数可以访问非 const 对象的非 const 数据成员、const 数据成员，也可以访问 const 对象内的所有数据成员；非 const 成员函数可以访问非 const 对象的非 const 数据成员、const 数据成员，但不可以访问 const 对象的任意数据成员
- 一个没有明确声明为 const 的成员函数被看作是将要修改对象中数据成员的函数，而且编译器不允许它为一个 const 对象所调用。因此 const 对象只能调用 const 成员函数
- const 类型变量可以通过类型转换符 const_cast 将 const 类型转换为非 const 类型
- const 类型变量必须定义的时候进行初始化，因此也导致如果类的成员变量有 const 类型的变量，那么该变量必须在类的初始化列表中进行初始化
- 对于函数值传递的情况，因为参数传递是通过复制实参创建一个临时变量传递进函数的，函数内只能改变临时变量，但无法改变实参。则这个时候无论加不加 const 对实参不会产生任何影响。但是在引用或指针传递函数调用中，因为传进去的是一个引用或指针，这样函数内部可以改变引用或指针所指向的变量，这时const 才是实实在在地保护了实参所指向的变量。因为在编译阶段编译器对调用函数的选择是根据实参进行的，所以，只有引用传递和指针传递可以用是否加 const 来重载。一个拥有顶层 const 的形参无法和另一个没有顶层 const 的形参区分开来



## 三五、C++ 中新增了 string，它与 C 语言中的 `char *` 有什么区别吗？它是如何实现的？

string 继承自 basic_string，其实是对 `char*` 进行了封装，封装的 string 包含了 `char*` 数组，容量，长度等等属性。

string 可以进行动态扩展，在每次扩展的时候另外申请一块原空间大小两倍的空间（2*n），然后将原字符串拷贝过去，并加上新增的内容。



## 三六、什么是内存泄漏，如何检测和避免

### 1. 内存泄漏

一般常说的内存泄漏是指**堆内存的泄漏**。堆内存是指程序从堆中分配的，大小任意的（内存块的大小可以在程序运行期决定）内存块，使用完后必须显式释放的内存。应用程序般使用 malloc、realloc、 new 等函数从堆中分配到块内存，使用完后，程序必须负责相应的调用 free 或 delete 释放该内存块，否则，这块内存就不能被再次使用，我们就说这块内存泄漏了

### 2. 避免内存泄漏的几种方式

- 计数法：使用 new 或者 malloc 时，让该数 +1。delete 或 free 时，该数 -1，程序执行完打印这个计数，如果不为 0 则表示内存泄漏
- 一定要将基类的析构函数为虚函数
- 对象数组的释放一定要用 delete[]
- 有 new 就有 delete，有 malloc 就有 free，保证它们一定成对出现

### 3. 检测工具

- Linux 下可以使用 Valgrind 工具
- Windows 下可以使用 CRT库



## 三七、对象复用、零拷贝

### 1. 对象复用

对象复用其本质是一种设计模式：Flyweight 享元模式

通过将对象存储到 “对象池” 中实现对象的重复利用，这样就可以避免多次创建重复的对象开销，节约系统资源

### 1. 零拷贝

零拷贝就是一种避免 CPU 将数据从一块存储拷贝到另外一块存储的技术。

零拷贝技术可以减少数据拷贝和共享总线操作的次数。

在 C++ 中，vector 的一个成员函数 `emplace_back()` 很好地体现了零拷贝技术，它跟 `push_back()` 函数一样可以将一个元素插入容器尾部，区别在于：**使用 `push_back()` 函数需要调用拷贝构造函数和转移构造函数，而使用 `emplace_back()` 插入的元素原地构造，不需要触发拷贝构造和转移构造**，效率更高。

```cpp
#include <vector>
#include <string>
#include <iostream>
using namespace std;

struct Person {
    string name;
    int age;
    // 初始构造函数
    Person(string p_name, int p_age): name(std::move(p_name)), age(p_age) {
         cout << "I have been constructed" <<endl;
    }
    // 拷贝构造函数
    Person(const Person& other): name(std::move(other.name)), age(other.age) {
         cout << "I have been copy constructed" <<endl;
    }
    // 转移构造函数
    Person(Person&& other): name(std::move(other.name)), age(other.age) {
         cout << "I have been moved"<<endl;
    }
};

int main() {
    vector<Person> e;
    cout << "emplace_back:" <<endl;
    e.emplace_back("Jane", 23); //不用构造类对象

    vector<Person> p;
    cout << "push_back:"<<endl;
    p.push_back(Person("Mike",36));
    return 0;
}

// 输出结果：
// emplace_back:
// I have been constructed
// push_back:
// I have been constructed
// I am being moved.

```

