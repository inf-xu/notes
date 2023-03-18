## 一、初始化

当对象在**创建时获得了一个特定的值**，我们说这个对象被**初始化**（initialized）。

**C++ 中的5种初始化**

- 默认初始化（default initialization）：`int a; string s;`
- 值初始化（value initialization）：`vector<int> ivec(2); vector<string> svec(2);`
- 直接初始化（direct initialization）：`string s("abc"); string s(10, 'c');`
- 拷贝初始化（copy initialization）：`string s("abc"), s1(s);`
- 列表初始化（list initialization）：`int a = {1};string str = {'a', 'b', 'c'};`
  



### 1. 默认初始化

在定义变量时**没有指定初始值**，则执行默认初始化：

```c++
int a;     // a的值与定义a的位置有关
string s;  // s是一个空串，调用了默认构造函数
```

（1）当变量类型是**内置类型**时：

​			定义在函数体**外**：**初始化0**；
​			定义在函数体**内**：**未初始化**（uninitialized），值是未定义的，由编译器决定。

（2）当变量类型是**自定义类型**时，会调用**类的默认构造函数**，由类自行决定它内部的各个属性的初始值。

​			定义为**成员变量**：**未初始化**，值是未定义的；
​			定义为**成员函数的局部变量**：**未初始化**，值是未定义的。

在 Visual Studio 2019 编译器中没有指定初始值时，直接输出会报错

```c++
#include<iostream>
#include<string>
using namespace std;

int aa;
char bb;
bool cc;
string dd;

int main() {
	int a;
	char b;
	bool c;
	string d;

    cout << aa << endl;  // ok, 0	
	cout << bb << endl;  // ok, ''
	cout << cc << endl;  // ok, 0	
	cout << dd << endl;  // ok, ""	
    
	cout << a << endl;  // error, 使用未初始化的内存“a”	
	cout << b << endl;  // error, 使用未初始化的内存“b”	
	cout << c << endl;  // error, 使用未初始化的内存“c”	
	cout << d << endl;  // error, 使用未初始化的内存“d”	
    
	return 0;
}
```



### 2. 值初始化（2022-10-16 未理解）

**内置类型**进行值初始化，与定义在函数体外部的内置类型类似，会**自动赋予内置变量某个值**。若是**自定义类型**，则会调用类型的**默认构造函数**。不同的是，这种初始化发生的地方，**不会出现未定义的变量值**。

```c++
vector<int> ivec(2);     // ivec中此时有两个int，且都为0
vector<string> svec(2);  // svec中有两个空的string，即""
```



### 3. 直接初始化

<u>在**标识符后面使用括号**，括号内填入对应的类型的值或构造函数的参数，即可使用直接初始化</u>

```c++
string s("abc");    // s = "abc"
string s(10, 'c');  // s = "cccccccccc"
```



### 2. 拷贝初始化

```c++
string s("abc"), s1 = s;
```

拷贝初始化通常使用拷贝构造函数来完成，通常发生在以下几种情况：

- 使用 `=` 定义变量，`=` 右边是**相同类型**
- 将一个对象作为实参传递给一个**非引用类型**的形参
- 从一个返回类型为**非引用类型**的函数返回一个对象
- 用花括号列表初始化一个数组中的元素或一个聚合类中的成员

```c++
#include<iostream>
#include<string>
using namespace std;

class Student {
public:
	Student(string name, int age):m_name(name),m_age(age) {
		cout << "参数构造函数~~" << endl;
	}
	Student(const Student & stu) {
		this->m_name = stu.m_name;
		this->m_age = stu.m_age;
		cout << "拷贝构造函数~~" << endl;
	}

public:
	string m_name;
	int m_age;
};

void display(Student stu) {
	cout << stu.m_name << "=>" << stu.m_age << endl;
}

void display_ref(Student & stu) {
	cout << stu.m_name << "=>" << stu.m_age << endl;
}

int main() {
	Student stu1("Tom", 12);  // 参数构造函数
	Student stu2 = stu1;  // 拷贝构造函数

	display(stu2);  // 拷贝构造函数
	cout << "----------------------" << endl;
	display_ref(stu2);  // 不是拷贝构造函数
	

	return 0;
}
```

输出结果：

```
参数构造函数~~
拷贝构造函数~~
拷贝构造函数~~
Tom=>12
----------------------
Tom=>12
```



### 2. 列表初始化

使用列表初始化，若用来初始化的值在执行类型转换时存在丢失信息的风险，则编译器将会报错。

```c++
int a = {1};
string str = {'a', 'b', 'c'};
vector<int> ivec = {1, 2, 3};
```





## 二、构造函数

构造函数（Constructor）是类的一种特殊的**成员函数**。

> **构造函数**是类控制其对象的**初始化过程**的**一个或几个**特殊的**成员函数**
>
> 构造函数的任务是**初始化类对象的数据成员**，无论何时**只要类的对象被创建，就会执行构造函数**
>
> -- 摘选自《C++ Primer》第5版本



构造函数时自动执行的，例如：

```c++
MyClass my_class;  // 此处执行了“默认构造函数”
MyClass my_class(5, 9.0);  // 此处执行了“参数构造函数”
MyClass my_class = 6;  // 此处执行了“转换构造函数”
MyClass my_class(other_my_class);  // 此处执行了“拷贝构造函数”
MyClass my_class(std::move(rhs_my_class));  // 此处执行了“移动构造函数”
```

构造函数的特殊性：

1. 自动执行
2. 名字和类名相同
3. 没有返回类型
4. 不能声明为`const`

**构造函数的类别**

- 默认构造函数
- 参数构造函数
- 转换构造函数
- 拷贝构造函数
- 移动构造函数
- `constexpr`构造函数



### 1. 默认构造函数

#### 1.1 编译器默认生成构造函数

定义一个 `MyClass` 类，如下：

```c++
class MyClass {
private:
	int num;
    float ratio;
    bool is_valid;
};

int main() {
    MyClass my_class;  // 调用编译器默认生成构造函数
}
```

如果我们没有写任何构造函数，编译器会默认生成一个构造函数，默认构造函数执行了所有数据成员的**默认初始化**。

只要我们写了一个构造函数，那么编译器便不会再生成构造函数。

#### 1.2 默认构造函数与初始化列表

##### 1.2.1 默认构造函数

为 `MyClass` 定义一个默认构造函数，不使用编译器的能力。

<u>函数名与类同名，**无返回值**，**无参数**，这样就定义了一个默认构造函数。</u>

```c++
class MyClass {
public:
    MyClass();
private:
	int num;
    float ratio;
    bool is_valid;
};

MyClass::MyClass():num(0),ratio(0.0f),is_valid(false){}   // 使用了初始化列表进行初始化
```

如果想让自己编写的默认构造函数与编译器生成构造函数的功能一样，可以使用 `default` 关键字：

```c++
class MyClass {
public:
    MyClass() = default;
private:
    int num;
    float ratio;
    bool is_valid;
};
```

##### 1.2.2 初始化列表

初始化列表负责为新创建的对象的一个或几个数据成员赋初值，它在**构造函数的函数体执行前执行**。

<u>每个变量赋值的顺序和变量定义的顺序相关</u>。

任何构造函数都可以使用初始化列表，且推荐使用初始化列表，一是效率，二是明确。

```c++
// 1. 初始化列表
MyClass::MyClass():num(0),ratio(0.0f),is_valid(false){}

// 2. 赋值
MyClass::MyClass() {
    num = 0;
    ratio = 0.0f;
    is_valid = false;
}
```

**初始化和赋值的区别**：

1. 使用初始化列表，**直接**将数据成员的值赋值为初始化列表中的值；
2. 未使用初始化列表，先对**数据成员执行默认初始化**，**再进行赋值**操作。

##### *1.2.3 必须使用初始化列表

某些数据成员，必须使用初始化列表，如下：

-  `const` 修饰的数据成员
- 引用类型成员

```c++
class MyClass {
private:
    int num;
    float ratio;
    bool is_valid;
    const int op;    // const修饰的数据成员
public:
    MyClass();    // 默认构造函数
};

MyClass::MyClass(): op(5) {
    // 仅示例必须使用初始值列表，其他数据成员请读者自行想象
}
```



### 2. 参数构造函数

#### 2.1 带参数的构造函数

参数构造函数，即在默认构造函数基础上添加函数参数。

```c++
class MyClass {
private:
    int num;
    float ratio;
    bool is_valid;
    double &distance_;    // 引用类型数据成员
public:
    MyClass(double &dis);    // 参数构造函数
};

MyClass::MyClass(int nnum, double &dis): num(nnum), distance_(dis) { // distance_必须使用初始化列表进行初始化
    // 获取一个对象的引用通常以为这需要修改它，
    // 在逻辑上意味着该对象是MyClass的重要组成部分
}
```

若我们将参数构造函数中的每个参数都写有默认值，则该参数构造函数为我们的类提供了默认构造函数的功能，括号里不需要传入任何参数依旧可以正确初始化。例如：

```c++
class MyClass {
private:
    int num;
    float ratio;
    bool is_valid;
public:
    // 参数构造函数
    MyClass(int _num = 0, float _ratio = 0.0, _is_valid = false);
};

MyClass::MyClass(int _num = 0, float _ratio = 0.0, _is_valid = false):
    num(_num), ratio(_ratio), is_valid(_is_valid) {}
```

#### 2.2 委托构造函数

在构造函数的**初始值列表处，调用其他的构造函数**，将构造工作委托给了下面的参数构造函数，如：

```c++
class MyClass {
private:
    int num;
    float ratio;
    bool is_valid;
public:
    // 默认构造函数
    // 将构造工作委托给了下面的参数构造函数
    MyClass(): MyClass(0) {}
    // 参数构造函数
    MyClass(int _num, float _ratio = 0.0, _is_valid = false):
    	num(_num), ratio(_ratio), is_valid(_is_valid) {}
};
```

<u>一个委托构造函数使用它**所属类的其他构造函数执行它自己的初始化过程**，就称作委托构造函数</u>。

在实际项目中，要根据类的性质而设计构造函数，将初始化的过程统一管理，委托给其他的构造函数，方便以后的代码维护。



### 3. 转换构造函数

如果构造函数**只接受一个实参**，则它实际上定义了**转换为此类类型的隐式转换机制**，有时我们把这种构造函数称作转换构造函数。

```c++
class MyClass {
private:
    int num;
public:
    // 转换构造函数
    MyClass(int _num): num(_num) {}
};

int main() {
    MyClass my_class = 1;  // ok
    return 0;
}
```

表面上看，我们将数据类型 `int` 转换为 `MyClass` 了。

是否执行隐式转换，完全看类设计者的意图。

当然我们可以使用 `explicit` 关键字来**抑制**构造函数定义的隐式转换。

```c++
class MyClass {
private:
    int num;
public:
    // 显示转换构造函数
    explicit MyClass(int _num): num(_num) {}
};
```

标准库中，`string` 接受一个 `const char *` 的构造函数，且没声明 `explicit`，因此我们可以写成：

```c++
string s = "abc";
```



### 4. 拷贝构造函数

```c++
class MyClass {
private:
    int num;
public:
    // 默认构造函数
    MyClass(): num(0) {}
    // 拷贝构造函数
    MyClass(const MyClass &orig);    // 拷贝构造函数
};
```

定义拷贝构造函数需注意：

1.  第一个参数必须是**自身类类型的引用**；
2.  该参数几乎总是 `const` ；
3.  不应该将拷贝构造函数定义为 `explicit` ，因为拷贝构造函数几乎是隐式使用。

如果定义了其他类型的构造函数却没有定义拷贝构造函数时，编译器会自动帮我们生成一个**拷贝构造函数**。

**何时调用拷贝构造函数：**

- 使用 `=` 定义变量，`=` 右边是相同类类型
- 将一个对象作为实参传递给一个**非引用类型**的形参
- 从一个返回类型为**非引用类型**的函数返回一个对象
- 用花括号列表初始化一个数组中的元素或一个聚合类中的成员



### 5. 移动构造函数（2022-10-17 未理解）

#### 5.1 对象移动

若一个类管理了很多其他类类型的成员变量（ `string`, `vector` 等），还包括使用 `new`关键字开辟的内存空间，拷贝一个对象的代价是相当大的。

在其中某些情况下，<u>对象拷贝之后就立即被销毁了，此时使用移动而不是拷贝将大幅提升程序性能</u>。

想要理解对象移动的工作原理，需要先明白 C++ 中的左值与右值，请参考【左值与右值】。

对象移动需要 C++ 引入的新类型，右值引用。

#### *5.2 右值引用

```c++
int a = 2;
int &&rr_a = a * 4;
```

上述代码，将一个右值引用绑定到了 `a * 4` 的表达式结果上，因为 `a * 4` 的结果是一个右值，因此可以讲 `rr_a` 绑定到这个值上。

对于返回左值的表达式：

> 返回左值引用的函数，连同赋值、下标、解引用和前置递增/递减运算符，都是返回左值的表达式的例子。我们可以将一个左值引用绑定到这类表达式的结果上。

对于返回右值的表达式：

> 返回非引用类型的函数，连同算术、关系、位以及后置递增/递减运算符，都生成右值。我们不能将一个左值引用绑定到这类表达式上，但是我们可以将一个const的左值引用或者一个右值引用绑定到这类表达式上。

#### 5.3 移动构造函数

声明移动构造函数如下：

```c++
class MyClass {
private:
    int *data;
public:
    MyClass(MyClass &&rhs) noexcept;  // 移动构造函数
};

// 移动构造函数
MyClass::MyClass(MyClass &&rhs) noexcept  // 1. 移动构造函数不应该抛出任何异常
    : data(rhs.data) {       // 2. 调用成员初始化器接管rhs中的资源
    rhs.data = nullptr;      // 3. 令rhs进入这样的状态，对其运行析构函数是安全的
}
```

在移动构造函数中，我们**没有分配任何新的内存**，它接管了 `rhs` 中的 `data`，并让 `rhs` 中的 `data` 置为 `nullptr` 。

此时 `rhs` 中的对象处于一种可以析构的状态，因为我们的移动构造函数就是来处理**对象拷贝之后就销毁**的情况。

除了将源对象处于可析构的状态，移动操作还必须保证源对象仍然有效，有效指的是可以安全地为其赋值。但是我们不应该再使用或者依赖源对象的任何值，因为已经将源对象所管理的内容移动到了新的对象中去了。

定义移动构造函数需注意：

- 移动构造函数不应该抛出任何异常；
- 令移后源进入对其运行析构函数是安全的状态，且不再对移后源中的值作任何假设。

#### 5.4 何时调用移动构造函数

编译器使用普通的函数匹配规则来确定使用的是**拷贝构造函数**还是**移动构造函数**，通常是：

> **移动右值，拷贝左值**

```c++
MyClass c1;
MyClass c2(c1);    // 调用 拷贝构造函数

MyClass get_my_class();    // 声明一个返回右值 MyClass 的函数
MyClass c3(get_my_class());    // 调用 移动构造函数
```

若一个类没有定义移动构造函数，编译器会为类生成一个移动构造函数。

有一些情况，编译器不会为类生成移动构造函数，此时调用上述的最后一行代码，将调用拷贝构造函数。



### 6. `constexpr` 构造函数

`constexpr` 构造函数是字面值常量类中的一种特殊的构造函数。

关于字面值常量类，参考【常量表达式 `constexpr` 】。

一个字面值常量类，必须至少定义一个 `constexpr` 构造函数。

`constexpr` 构造函数的函数体是空的，只能**使用初始化列表进行所有成员的初始化**。**提供的初始值必须是一条常量表达式**，或者使用`constexpr` 构造函数（因为字面值常量类可以嵌套字面值常量类）。

可以声明为 `=default` 或者 `=delete` 。

```c++
class MyConstexprClass {
private:
    int sz_;
    double ratio_;
public:
    constexpr MyConstexprClass(int sz = 0, double ratio = 0.0) :
            sz_(sz), ratio_(ratio) {}
};
```



## 三、Const

如果希望定义一种变量，它的值不能被改变，比如缓冲区的大小、π（3.1415926）等。既可以随时调整，又可以防止程序在执行过程中一不小心改变了这个重要的值。

在 C++ 中，可以使用 `const` 修饰变量来实现：

```c++
const int buff_size = 32;
```

`const` 不仅仅可以修饰变量，它还有其他很多用法：

1. 修饰基本内置类型
2. 修饰自定义对象
3. `const` 与数组
4. `const` 与引用
5. `const` 与指针（顶层 `const` 与底层 `const` ）
6. 修饰函数参数
7. 修饰函数返回值
8. 修饰成员函数



### 1. `const` 修饰基本内置类型

将基本内置类型声明为 `const` 后，在以后的代码不能如下面修改 `buff_size` 。

```c++
const int buff_size = 32;
buff_size = 64; // 错误，编译不通过
```

在定义 `const` 变量的时候，**必须进行初始化**。编译器在编译过程中，会把用到该变量的地方**全部替换成相应的值**。

> 默认情况下，const仅在当前文件中有效。若希望多个文件使用同一个const变量，可以使用extern来修改，在任何声明和定义处使用（当然只需要定义一次）。



### 2. `const` 修饰自定义对象

```c++
const MyClass obj;
```

这样的声明意味着 `obj` 中的**成员变量不可以被修改**，且**只能调用常成员函数**（见第8部分）。

但是 C++ 还提供了一个关键字 `mutable`（参考【mutable】），它可以允许我们修改常量对象中的成员变量。



### 3. `const` 与数组

`const` 与数组只有一种形式，即：

```c++
const int arr[] = {2, 3};
```

同样，**必须初始化**；不可以执行 `arr[0] = 1` 类似的代码。

参考【数组】，我们了解编译器通常把数组处理为<u>指向首元素的相应类型的指针</u>，这里也是，类似于 `const int *` 类型。

`const` 与指针相关内容，请看第5节。



### 4. `const` 与引用

#### 4.1 定义

可以把引用绑定到 `const` 对象上，我们称之为**对常量的引用**。

```c++
const int ci = 422;
const int &r_ci = ci;

r_ci = 500;  // 错误
```

上述代码第二行将一个引用绑定到了一个常量，为了正确的绑定，必须将引用声明成 `const` 。

我们通常称**对常量的引用**为**常量引用**。这样的叫法，非常非常非常容易和**常量指针**搞混，建议仔细品尝。

这样我们就不可以通过引用 `r_ci ` 修改 `ci` 。

#### 4.2 常量引用的初始化

C++ 的引用必须与其所引用的对象的类型严格一致。但是在初始化常量引用的时候，可以用任意表达式作为初始值，只要该表达式的结果**能转换成**引用的类型即可。

例如非常量对象、字面值、表达式。

```c++
int i = 42;
const int &r1 = i;  // 正确，常量引用绑定在一个非const变量上
const int &r2 = 42;  // 正确，常量引用绑定在字面值上
const int &r3 = r1 * 2;  // 正确，常量引用绑定在表达式结果上
```

当一个常量引用被绑定到另外一种数据类型上的时候，编译器执行了如下的过程：

1. 生成另一个临时量，用目标对象的值初始化临时量；
2. 将常量引用绑定到上述的临时量上。

```c++
double d = 3.14;
const int &ri = d;

// 会被编译器解释成如下代码
double d = 3.14;
const int temp = d;  // 执行double -> int的隐式转换
const int &ri = temp;  // 将常量引用 ri 绑定到临时量上
```

若变量类型和常量引用不需要执行转换，则没有生成临时量这一步。

```c++
double d = 3.14;
const double &rd = d;  // 不需要生成临时量
```

C++规定，将左值引用绑定到临时量是非法的；但是却可以将**常量引用绑定在临时量**上。因此，我们可以将函数的返回值声明成常量引用：

```c++
const int &get_a() {
    int a = 3;
    return a;
}
```

调用 `get_a` 将得到数值3。这一点也在【引用】中初始化引用的例外情况1提及：可以用临时量初始化常量引用。

这里其实可以用 C++11 的右值引用来解释。我们可以写成：

```c++
double d = 3.14;
const int &&ri = d;
```

但是却不能写成：

```c++
double d = 3.14;
const double &&rd = d;
// 编译不通过
// 初始化右值引用的必须是一个右值
```



### 5. `const` 与指针

#### 5.1 指向常量的指针

和**对常量的引用**（常量引用）一样，我们可以定义一个**指向常量的指针**：

```c++
const double pi = 3.14;
const double *ptr_pi = &pi;

// 错误调用，不可以通过ptr_pi修改pi
*ptr_pi = 3.1415926;
```

<u>指针的类型必须与其所指向的类型严格一致。但是允许另一个指向常量的指针指向一个非常量对象。</u>例如：

```c++
double pi = 3.14;
const double *ptr_pi = &pi;

const int *ptr_p = &pi;  // error "double *" 类型的值不能用于初始化 "const int *" 类型
```

这种用法，和常量引用类似，但是没有临时量的概念加入，基本数据类型必须是严格匹配的。

其实不论是**常量引用**还是**指向常量的指针**，都是引用或指针的“自以为是”。它们觉得自己引用或指向了一个常量，所以自觉不去改变目标的值。

#### 5.2 常量指针

因为指针是 C++ 中的对象，引用仅仅是别名，因此不可以将引用设置为 `const` ，但是可以**将一个指针声明为 `const`**。

实现的效果就是，该指针一旦初始化完成之后，就不能更改指针内容，即不允许将该指针指向别的对象。

```c++
int buff_size = 512;
int *const pti = &buff_size;  // pti 将始终指向 buff_size

// 允许通过常量指针修改原对象的值
// 因为原对象也是非常量
*pti = 1024;

const double pi = 3.14;
const double *const pt_pi = &pi;  // pt_pi是一个常量指针，指向常量的常量指针
```

#### 5.3 顶层 `const` 和底层 `const`

指针本身是不是常量及所指的是不是一个常量，是两个毫无关联的问题。

- 使用术语**顶层 `const` **，表示指针本身是个常量；
- 使用术语**底层 `const` **，表示指向的对象是个常量。

![image-20221020094040598](http://cdn.starfun.top/img/image-20221020094040598.png)

更一般的，任意对象的 `const` ，都是顶层 `const`

```c++
int i = 0;
// 顶层const，修饰普通对象
// 不允许改变ci的值
const int ci = 5;
// 其实可以写成 int const ci = 5;

// 顶层const，修饰指针对象
// 不允许改变cp_i的值
int *const cp_i = &i;

// 底层const，修饰指针指向的对象
// 不允许通过p_ci改变ci的值，即*p_ci = 10
const int *p_ci = &ci;

// 左侧的是底层const，右侧的是顶层const
// 既不允许改变cp_ci的值，也不能使用*cp_ci的改变ci的值
const int *const cp_ci = &ci;

// 底层const，引用都是底层const
// 不允许通过r_ci改变ci的值
const int &r_ci = ci;
```

<u>更通俗的理解：`p` 不能更改 -> 顶层  `const` ；`*p` 不能更改 -> 底层  `const` ；</u>

注意：只有两处有底层 `const` ，一个是指针，一个是引用。

当执行对象的拷贝操作时，常量是**顶层 `const` **还是**底层 `const`** 区别很大。

**顶层 `const`**  不受什么影响，对拷贝源不做过多要求，因为拷贝只关心拷贝源的值。

```c++
// ci是顶层const的常量
// 可以赋值给变量i
i = ci;

// cp_ci有顶层const
// 可以赋值给没有顶层const的p_ci
p_ci = cp_ci;
```

但是，<u>拷贝目标的**底层 `const` **要求拷贝源也是一个常量，拷贝目标和拷贝源必须具有相同的**底层 `const`**</u>：

- 可以把非底层 `const` 的变量赋值给一个底层 `const`

  ```c++
  int a = 20;
  int *p_a = &a;
  // 非底层const赋值给底层const
  const int *p_ca = p_a;
  ```

- 但是不可以使用底层 `const` 赋值给非底层 `const`

  ```c++
  int a = 20;
  const int *p_ca = &a;
  // 编译出错
  // 底层const赋值给非底层const
  int *p_a = p_ca;
  ```

那么，我们什么时候需要这么严格的匹配呢？答案就是函数调用时传递的函数参数。



### 6. `const` 修饰函数参数

我们可以将函数参数声明为 `const` ，且在函数调用处，利用传递给函数的实参对函数形参初始化。

<u>形参的初始化方式和变量的初始化方式是一样的</u>，因此调用函数传递 `const` 实参的规则，和上述的顶层 `const` 和底层 `const` 的规则一模一样。

因为 C++ 支持函数的重载，又因为顶层 `const` 的要求很松，若如下同时声明了两个函数：

```c++
void func(const int i);
void func(int i);  // 错误
```

当我们调用的时候编译器无法区分，因此上述声明重复定义了 `func` ，是不允许的，这一点在【函数重载】提及，在【函数匹配】之中详细介绍了上述示例错误的原因。

C++ 标准建议，<u>当我们不需要修改某个函数参数的值的时候，尽量将其声明为常量引用</u>，即：

```c++
void func(const int &i);
```



### 7. `const` 修饰函数返回值

<u>函数返回一个值的方式和初始化一个变量或者形参的方式完全一样：返回的值用于初始化调用点的一个临时量，该临时量就是函数调用的结果。</u>

通常情况下，返回基本内置类型，不需要添加 `const` ，因为函数返回的是一个右值。

返回指针和引用的时候，如果不添加 `const` ，我们可能会写出如下代码：

```c++
string &shorter_string(string &s1, string &s2) {
    return s1.size() <= s2.size() ? s1 : s2;
}

shorter_string(s1, s2) = "new string";
```

虽然正确，但是很迷惑，通常添加 `const` 。

```c++
const string &shorter_string(const string &s1, const string &s2) {
    return s1.size() <= s2.size() ? s1 : s2;
}
```

在第 4 节我们也提及了返回常量引用的函数，可以返回函数体内的临时量：

```c++
// 调用get_a会得到数值3
const int &get_a() {
    int a = 3;
    return a;
}
```

要想做到这种效果，必须添加 `const` 。



### 8. `const` 修饰成员函数

可以如以下方式定义一个 `const` 成员函数（常量成员函数）：

```c++
class MyClass {
private:
    int i = 0;
public:
    int func() const;
};
```

紧随参数列表之后的 `const` 关键字，其作用是**修改隐式常量指针this的类型**。

默认情况下，`this` 的类型是指向类类型**非常量版本的常量指针**：

```c++
// 这是一个顶层const，this指针的值是不允许被修改的
// 即只能指向本类的一个实例自己
MyClass *const this
```

假如我们将 `MyClass` 对象声明为了常量，指针变成了：

```c++
// 既包含顶层const，又包含底层const
const MyClass *const this
```

导致不能在一个常量对象上调用普通的成员函数，包括常量对象的引用和常量对象的指针：

```c++
const MyClass mc;
// 无法调用，底层const约束不能改变任何成员变量
mc.non_const_func();
```

当我们把 `const` 添加在成员函数之后，就可以使用常量对象、及其引用和指针调用了。

此时，我们不能再在 `const` 成员函数中**修改任何的成员变量**。

```c++
const MyClass my_class;
my_class.func();
```

关于函数的调用，有以下规则：

- 若 `my_class` 不是常量，且类只有 `func` 的常量版本，则会调用 `func` 的常量版本；
- 若 `my_class` 不是常量，同时声明了 `func` 的**非常量版本**和常量版本，则会调用 `func` 的**非常量版本**。

见如下两个示例：

```c++
class MyClass {
public:
    int func() const { return 1; }
};

int main() {
    // 不是常量
    MyClass mc;
    // 输出1
    std::cout << mc.func() << std::endl;
}
```

```c++
class MyClass {
public:
    int func() { return 0; }
    int func() const { return 1; }
};

int main() {
    // 不是常量
    MyClass mc;
    // 调用普通版本，输出0
    std::cout << mc.func() << std::endl;
    // 是常量
    const MyClass c_mc;
    // 输出1
    std::cout << c_mc.func() << std::endl;
}
```



### 9. `const` 总结

`const` 的是将对象和函数声明为只读状态的一个关键字。

`const` 可以与基本内置类型使用，此时定义的变量初始化之后就不会改变，编译器会将用到该变量的地方都替换成变量的值。

`const` 可以修饰自定义对象，此时只能调用 `const` 成员函数。

`const` 可以与引用使用，表明这是一个对常量的引用，通常称作常量引用。

`const` 可以与指针使用，要区分指向常量的指针和常量指针，以及顶层 `const` 和底层`const` 的定义。

`const` 可以修饰函数的参数，通常传递常量引用时使用。

`const` 可以修饰函数的返回值，通常在返回指针和引用时使用。

`const` 可以修饰成员函数，修改隐式常量指针 `this`的类型，使常量对象可以调用成员函数。







