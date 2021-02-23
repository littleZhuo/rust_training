## 语言基础

### 变量与可变性

Rust的变量默认是不可变的(Immutable)，即一旦值被绑定到一个名称上，你就不能改变这个值。也就是说变量不能再次被赋值。

如果要改变的话需要在变量名称前加`mut`，比如`let mut a: i32 = 10;`

这里需要注意，变量的不可变性有点类似于C语言的常量(constants)概念，但是Rust也有常量的概念，是另一回事儿。

- Rust的变量不可变性说到底还是变量，C中对应`const`常量一样，他们始终是变量。深层次来说C中的常量，只是说是**只读的**，而不是真的不能变，比如硬件就是可以改的。

  > 在C++中,`const`变量则是真正的常量了,定义时会将其放入符号表中。所以编译途中遇到使用`const`变量时,则直接从符号表中取出常量。这个定义就和Rust的`const`一致了。

- Rust的常量关键字也是`const`，它其实类似于C中的宏，但是有类型且必须注明类型。作用域是全局。只能被设置（赋值）为常量表达式[^1]，而不能是函数调用的结果，或者任何其他只能在运行时计算出的值。其生命周期是全局的，没有固定的内存地址，并且会被编译器有效地内联到每个使用到他的地方。

  [^1]: `Const`，只能使用支持CTFE（Compile-Time Function Evaluation）的表达式



### 所有权

Rust对于内存的管理秉承三个原则：

1. Rust 中的每一个值都有一个被称为其 **所有者**（*owner*）的变量。
2. 值有且只有一个所有者。
3. 当所有者（变量）离开作用域，这个值将被丢弃。

此原则不同于python和Java的GC，更类似于C++的RAII，只是做的更严格，内存的申请和释放都由编译器控制，程序员不能直接操作。

#### 变量与数据交互方式（一）：移动

举个例子

```rust
let x = 5;
let y = x;
```

结果是x和y都是5，主要是y=x时，x生成了一个拷贝绑定给了y。

那如果是String类型呢

```rust
let s1 = String::from("hello");
let s2 = s1;
```

![image-20200909103144353](Rust入门知识.assets/image-20200909103144353.png)

s1的内存如上，左边是栈内的情况，右边是堆上的情况。

当s2=s1时，内存不是下面的这样

![image-20200909103414556](Rust入门知识.assets/image-20200909103414556.png)

也不是这样

![image-20200909103424949](Rust入门知识.assets/image-20200909103424949.png)

而是这样

![image-20200909103533724](Rust入门知识.assets/image-20200909103533724.png)

也就是说Rust的默认赋值不是浅拷贝(shallow copy)、也不是深拷贝(deep copy)，而是移动(move)。没错就是C++ 11开始引入的move概念。

这样当s2离开作用域后，对应的内存就会被唯一的释放。



#### 变量与数据交互方式（二）：克隆

而对于`i32`这类已知大小的类型而言，默认实现了`Copy trait`，就会在赋值的时候做`Copy`的动作，而不是`Move`。

Rust规定，不允许自身或者其任何部分实现了`Drop trait`的类型使用`Copy trait`。自定义类型可以通过实现`Copy trait`来支持值传递。



## 基本数据类型

### 数组类型

数组类型（Array）是Rust内建的原始集合类型，特点是：

- 数组大小固定
- 元素类型一致
- 默认不可变（即元素不可变，所以即使定义为`mut`，也只能修改元素的值，而不是增删元素）

数组的定义方式 

```rust
let arr: [i32; 3] = [1, 2, 3]; 	// 定义类型T和长度，并初始化
let arr = [1, 2, 3];			// 直接初始化，推导类型T和长度
let arr = [0; 10];				// 批量初始化，长度10，元素都为0，类型自动推导
```

> 只有实现了Copy trait的类型才能作为数组的元素。
>
> 因为这里涉及到所有权的问题，Rust对于基本数据类型以及基本数据类型组成的数组、tuple执行的是复制操作。其他类似都默认是移动操作，除非改类型实现了Copy trait.

```
fn main() {
    let a = (1, 2, 3);
    let b = a;
    println!("{:?}", a); // 不会有问题，b拷贝了a对应的对象
    println!("{:?}", b); 

    let c = "Huawei".to_string();
    let d = c;
    println!("{}", c); // 会报错，因为c对应的对象已经被转移给d，c是未初始化的
    println!("{}", d);
}
```



### 范围类型

对应关系

```rust
(1..5)         	std::ops::Range{start: 1, end: 5}
(1..=5)			std::ops::RangeInclusive::new(1, 5)
```



### 切片(slice)

简单来说就是一种引用类型，slice 允许你引用集合中一段连续的元素序列，而不用引用整个集合。比如`&str`切片，本身在栈中，分别存储了堆中字符串的首地址以及长度。这种同时包含了动态大小类型地址信息和长度信息的指针，称为胖指针(Fat Pointer )，所以切片就是一种胖指针。



## 字符串

字符串分为

- str字符串类型，不可变长
- String类型，可变长

### str字符串类型

固定长度的字符串，也就是`&str`字符串。

```rust
let astr = "我是一个例子。";
// let bstr: & 'static str = "我也是一个例子。"; // 注意这里奇怪的写法，& 'static str，其中& str表示str字符串， 'static 用来表达生命周期是静态的
let ptr = astr.as_ptr();
let len = astr.len();
```

字符串的本质是一段有效的UTF-8的字节序列，因此可以将一段字节序列转换成`&str`字符串。

```rust
let s = unsafe {
    let slice = std::slice::from_raw_parts(ptr, len);
    std::str::from_utf8(slice)
};
assert_eq!(s, Ok(astr));
```



### String类型

`String`类型从源码看就是封装了`Vec<u8>`，然后定义了一些方便使用的方法。

```rust
#[derive(PartialOrd, Eq, Ord)]
#[cfg_attr(not(test), rustc_diagnostic_item = "string_type")]
#[stable(feature = "rust1", since = "1.0.0")]
pub struct String {
    vec: Vec<u8>,
}
```

初始化的方法基本如下

```rust
fn main() {
    let name: String = String::from("Dyce");
    let name2: String = "Dyce".to_string();
    println!("{}", name);
    println!("{}", name2);
}
```



## 指针

表示内存地址的类型称为指针。具体分为

- 引用（Reference），Safe Rust的非空安全指针
- 原生指针（Raw Pointer），Unsafe Rust的可能为空非安全指针
- 函数指针（fn Pointer）
- 智能指针（Smart Pointer）

### 引用

> 和借用（Borrowing）的关系是什么？有人说引用是借用的唯一实现方式。

Rust的引用在语法上类似于C++的指针和引用的结合体。首先使用`&`获取引用，使用`*`解引用。默认是不可变。

同时 Rust的引用可以在编译期间就保证生命周期合法，例如如下场景，引用`r`借用的对象`x`生命周期太短，编译就会报错。

```rust
fn main() {
    let r: &i32;
    {
        let x = 10;
        r = &x;
    }
    assert_eq!(*r, 10);
}
```

产生报错

```rust
error[E0597]: `x` does not live long enough
 --> src/main.rs:5:13
  |
5 |         r = &x;
  |             ^^ borrowed value does not live long enough
6 |     }
  |     - `x` dropped here while still borrowed
7 |     assert_eq!(*r, 10);
  |     ------------------- borrow later used here

error: aborting due to previous error
```

那如果对象已经被借用后，对对象进行修改，同样可能产生垂悬指针的问题，如何应对？

```rust
fn main() {
    let v = vec!{1, 2, 3, 4};
    let r = &v;
    let aside = v;
    r[0];
}
// 编译结果如下，当对象有不可变借用存在时，禁止对此对象的任何部分进行修改。
error[E0505]: cannot move out of `v` because it is borrowed
 --> src/main.rs:4:17
  |
3 |     let r = &v;
  |             -- borrow of `v` occurs here
4 |     let aside = v;
  |                 ^ move out of `v` occurs here
5 |     r[0];
  |     - borrow later used here

error: aborting due to previous error; 1 warning emitted

For more information about this error, try `rustc --explain E0505`.
error: could not compile `hello`.
```

而当对象有可变借用存在时，对此对象的唯一访问方式是通过这个可变借用。甚至包括具有对象所有权的变量在内的所有其他方式均不可用。

```rust
fn main() {
    let mut v = vec!{1, 2, 3, 4};
    let r = &mut v;
    v[0] = 10;
    r[1] = 11;
}
// 编译报错，借用后又通过v修改了对象。
   Compiling hello v0.1.0 (/home/dongshuai/rust/hello)
error[E0499]: cannot borrow `v` as mutable more than once at a time
 --> src/main.rs:4:5
  |
3 |     let r = &mut v;
  |             ------ first mutable borrow occurs here
4 |     v[0] = 10;
  |     ^ second mutable borrow occurs here
5 |     r[1] = 11;
  |     - first borrow later used here

error: aborting due to previous error
```

关于解引用，Rust支持在访问成员的时候省略

```rust
struct Obj {
    name: & 'static str,
    age: u32,
}

fn main() {
    let a = Obj{name: "Dyce", age: 10};
    let b = &a;
    assert_eq!(b.name, "Dyce");
    assert_eq!(b.age, 10);
    assert_eq!((*b).age, 10);
}
```





### 原生指针

原生指针（裸指针）不受Rust的安全检查规则限制。可以和引用（胖指针）互转，如下代码。

```rust
let mut x = 10;
let ptr_x  = &mut x as *mut i32;	// 将x的引用强转成可变i32原生指针

let y = Box::new(20);				// 堆内存上存储数字20 
let ptr_y = & *y as *const i32;		// 还不清楚

unsafe {							// 重点！原生指针的操作一定要放到unsafe块中
    *ptr_x += *ptr_y;
}
assert_eq!(x, 30);
```



### 函数指针

函数名称就是其指针，是Rust里的一等公民。函数指针的存在，使得很多高阶函数得以实现。

```rust
fn call_tree_times(fn_ptr: fn(f32, f32) -> f32) {
    for _ in 0..3 {
        fn_ptr(1.0, 2.0);
    }
}

fn callee(_x: f32, _y: f32) -> f32 {
    println!("callee Enter!");
    return 0.0;
}

fn main() {
    call_tree_times(callee);
}
```





### 智能指针

Rust中的值默认被分配到栈内存，Rust通过`Box<T>`将值**装箱**（在堆内存中分配）。Rust的这个特性源自C++语言，所以会在超出作用域范围时，自动调用其析构函数，销毁内部对象，并自动释放堆中的内存。

**还有其他的智能指针，待补充。**

```rust
    #[derive(PartialEq, Debug)]

    struct Point {
        x: f64,
        y: f64,
    }

    let box_point = Box::new(Point {x: 0.0, y: 0.0});
    let unboxed_point: Point = *box_point;

    assert_eq!(unboxed_point, Point {x: 0.0, y: 0.0});
```





## 特殊类型

### never类型

`nerver`类型也叫底类型(Bottom Type)或者(Bang Type)，它用来表示“无”的概念，同时也可以表示任何类型，有点道家思想：阴阳互生，相生相克。

底类型的存在使得Rust的类型系统更加完善，方便抽象。比如后面介绍的Set类型和Map类型的关系。

> 实验特性，必须在Nightly版本中才能显示使用never类型。



## 复合数据类型

- 元组（Tuple）
- 结构体（Struct）
- 枚举体（Enum）
- 联合体（Union）

### 元组

元组是一种异构、有限序列，即元素类型可以不同，元素数目固定的类型。形如`(T, U, M, N)`，其中TUMN均为泛型。

> Rust的元组类似于python和C++11的tuple。

```rust
    let tuple: (& 'static str, i32, char) = ("hello", 5, 'c'); // 显示声明类型
    assert_eq!(tuple.0, "hello");   // 通过.操作符+下标来访问元素
    assert_eq!(tuple.1, 5);
    assert_eq!(tuple.2, 'c');

    let coords = (3, 7);
    let result = move_coords(coords);
    assert_eq!(result, (4, 8));

    let (x, y) = move_coords(coords);  // let 支持模式匹配，所以可以用来解构。
    assert_eq!(x, 4);
    assert_eq!(y, 8);
```

需要注意，当元祖中只有一个值的时候，需要加逗号，即`(0,)`，这是为了和括号中的其他值区分。这个要求和python对于tuple的要求一致。



### 结构体

Rust提供了三种结构体

- 具名结构体 (Named-Field Struct)
- 元组结构体 (Tuple-Like Struct)
- 单元结构体 (Unit-Like Struct)

#### 具名结构体

最常见的结构体类型，可以理解是C++中struct和class的合体。

先看纯数据的结构体用法

```rust
struct People {			  // 结构体名称必须是驼峰格式，否则会有编译告警
    name: & 'static str,  // 这里是逗号不是分号
    gender: u32           // 这里可以有一个逗号
} // 这里没有分号

fn main() {
    let one = People{name: "dyce", gender: 1}; // 初始化，这里是花括号，不是圆括号。而且必须对所有的字段赋值
    assert_eq!(one.name, "dyce");
    assert_eq!(one.gender, 10);
}
```

再看类型定义

```rust
#[derive(Debug, PartialEq)]

struct People {
    name: & 'static str,  // 这里不是分号
    gender: u32           // 这里多一个逗号
} // 这里没有分号

impl People {
    fn new(name: & 'static str, gender: u32) -> Self {  // 类似于c++的构造函数，或者python的new，但是Rust实际没有构造函数。
        return People{name: name, gender: gender};
    }

    fn name(&self) {                                    // 类似于getter方法
        println!("name: {:?}", self.name);
    }

    fn set_name(&mut self, name: & 'static str) {       // 类似于setter方法
        self.name = name;
    }
}

fn main() {
    let dyce = People::new("dyce", 1);
    dyce.name();
    // dyce.set_name("new_name"); // 这里编译不过，因为dyce默认是不可变的。

    let mut alice = People::new("alice", 0);
    alice.name();
    alice.set_name("new_name");
}
```

- `impl`中定义的叫做方法，否则叫做自由函数。
- `fn new`类似于构造函数，可以理解为是类方法，类似于C++中类的静态方法和python中的类方法。所以不需要有&self参数。调用的时候使用`::`操作符。
- `fn name`和`fn set_name`类似于类实例的方法，类比C++的类方法和python中的类方法。需要有&self参数，如果需要修改类成员则必须加mut。调用的时候使用`.`操作符。



#### 元组结构体

元组结构体从名字就可以看出来是元组和结构体的混合体，特点是字段没有名称，只有类型。

```rust
struct Color(i32, i32, i32);  // 圆括号，分号结尾

fn main() {
    let blue = Color(0, 1, 2);
    assert_eq!(blue.0, 0);
    assert_eq!(blue.1, 1);
    assert_eq!(blue.2, 2);
}
```



#### 单元结构体

单元结构体、单元类型和空枚举体都属于**零大小类型**(Zero Sized Type, ZST)，他们的值就是其本身（也就是没有内容，只有类型），运行时并不占用内存空间。

```rust
struct Empty;       // 单元结构体
struct Empty2 {}    // 单元结构体的另一种写法
struct Empty3 ();   // 单元元组结构体的写法
enum Void {}        // 空枚举体，不支持没有大括号的写法，不知道为什么没做到统一。。。
struct Baz {
    empty: Empty,
    empty2: Empty2,
    qux: (),        // 单元类型
    baz: [u8; 0],   // 空数组
}

fn main() {
    assert_eq!(std::mem::size_of::<()>(), 0);
    assert_eq!(std::mem::size_of::<Empty>(), 0);
    assert_eq!(std::mem::size_of::<Empty2>(), 0);
    assert_eq!(std::mem::size_of::<Empty3>(), 0);
    assert_eq!(std::mem::size_of::<Void>(), 0);
    assert_eq!(std::mem::size_of::<Baz>(), 0);
    assert_eq!(std::mem::size_of::<[(); 10]>(), 0);
}
```



#### 枚举体

枚举体的四种形式

```rust
// 无参数枚举体
enum Number {       
    Zero,           // 需要首字母大写，否则有编译器告警
    One,            
    Two,
}                   // 没有分号

// 类C枚举体
enum Color {
    Red = 0xff0000,
    Green = 0x00ff00,
    Blue = 0x0000ff,
}

// 带参数枚举体
enum IpAddr {
    V4(u8, u8, u8, u8),
    V6(String),
}

// 最近又学到一种，里面是结构体
enum Message {
    Quit,
    Move { x: i32, y: i32},  // 看这里，秀~
    Write(String),
    ChangeColor(i32, i32, i32),
}

fn main() {
    let n = Number::One;
    match n {
        Number::Zero => println!("0"),
        Number::One => println!("1"),
        Number::Two => println!("2"),
    }

    println!("roses are #{:06x}", Color::Red as i32); // 输出 roses are #ff0000

    // let x: fn(u8, u8, u8, u8) -> IpAddr = IpAddr::V4;
    // let y: fn(String) -> IpAddr = IpAddr::V6;
    let home = IpAddr::V4(127, 0, 0, 1);
    let office = IpAddr::V6(String::from("11:11:11:11"));
    if let IpAddr::V4(x, y, z, q) = home {println!("This is IPV4: {}.{}.{}.{}", x, y, z, q)}; // 输出 This is IPV4: 127.0.0.1
}
```

其中需要说明的是参数枚举体，其中的枚举值如31和32行所示，本质属于函数类型，是枚举类型的不同形式的构造器。

*这种用法我自己是不好理解的，为什么会这么设计？不知道随着学习的深入是否能解答这个疑问。*



##### Option类型

详细见泛型章节。

标注库提供的`Option<T>`类型已经通过`std::prelude::v1::*`自动引入到每个Rust包里，所以可以直接使用`Some(T)`或者`None`来表示一个`Option<T>`类型，而不需要写`Option::Some(T)`或者`Option::None`。



##### Result类型

详细见错误处理章节。



## 常用集合类型

- 线性序列：向量、双端队列、链表
- Key-Value映射表：无序哈希表、有序映射表
- 集合类型：无序集合、有序集合
- 优先度列：二叉堆



### 向量(Vec)

向量的三种创建方法。

```rust
fn main() {
    let mut v1 = vec![];
    // let mut v11 = vec!{};  这两种都可以
    // let mut v12 = vec!();

    v1.push(1);
    v1.push(2);
    v1.push(3);

    let mut v2 = vec![0; 10]; // 这个类似于定长数组
    v2.push(1);

    let mut v3 = Vec::new();
    v3.push(2);

    v3[1]; // thread 'main' panicked at 'index out of bounds: the len is 1 but the index is 1', src/main.rs:16:5
}
```



### 双端队列(VecDeque)

双端队列是一种具有队列（先进先出）和栈（后进先出）性质的数据结构。Rust中使用`RingBuffer`算法实现双端队列。

```rust
use std::collections::VecDeque;     // VecDeque非默认引入

fn main() {
    let mut vd = VecDeque::new();
    // 先进先出
    vd.push_back(1);
    vd.push_back(2);
    assert_eq!(vd.pop_front(), Some(1));
    assert_eq!(vd.pop_front(), Some(2));

    // 先进后出
    vd.push_back(1);
    vd.push_back(2);
    assert_eq!(vd.pop_back(), Some(2));
    assert_eq!(vd.pop_back(), Some(1));

    // push front
    vd.push_front(1);
    assert_eq!(vd.get(0), Some(&1));	// 这里是一个引用
    vd.pop_front();
    assert_eq!(vd.pop_back(), None);
    assert_eq!(vd.pop_front(), None);
}
```



### 链表(LinkedList)

双向链表，提供了`push_back()、push_front()`两类方法，同时提供了`append()`方法，可以合并两个链表。

> 有意思的是目前不支持`insert()`和`erase()`，熟悉C++的同学一定会很好奇，为什么？
>
> 现在有说法是因为考虑性能和安全性，也有说是单纯还没实现。没办法，现在Rust还在飞速的发展，很多新特性都在`nightly`版本才有，提供了Cursor_mut可以做类似的操作。
>
> ![image-20200829111413925](Rust入门知识.assets/image-20200829111413925.png)



### HashMap和BTreeMap

这两个类型和C++中定义比较相似。`HashMap`要求key必须是可hash的，元素是无序的。`BTreeMap`要求key是可排序的，元素是有序的。两者都要求value是编译期间可知大小的类型。



### HashSet和BTreeSet

这两个类型其实就是`HashMap<K, V> 和 BTreeMap<K, V>`把Value设置为空元组的特定类型。等价于`HashMap<K, ()> 和 BTreeMap<K, ()>`。



### 二叉最大堆(BinaryHeap)

优先队列的实现。

```
use std::collections::BinaryHeap;

fn main() {
    let mut bh = BinaryHeap::new();
    assert_eq!(bh.peek(), None);

    let arr = [93, 25, 77, 46, 1, 89, 0];
    for &i in arr.iter() {
        bh.push(i);
    }

    assert_eq!(bh.peek(), Some(&93));
}
```



## 泛型和trait

### 泛型

前面介绍的`Option<T>`就是典型的泛型，这个和C++中的泛型定义类似，对于python，因为是动态语言，所以类型可以随时变，自带泛型。

```rust
use std::fmt::Debug; // 引入Debug trait

fn match_option<T: Debug>(o: Option<T>) { // Debug限定类型T，只有实现了Debug trait的类型才拥有使用"{:?}"打印的行为
    match o {
        Some(i) => println!("{:?}", i),
        None => println!("nothing"),
    }
}

fn main() {
    let a: Option<i32> = Some(3);
    match_option(a);            // 3

    let b: Option<&str> = Some("woshishei");
    match_option(b);            // woshishei

    let c: Option<char> = None;
    match_option(c);            // nothing

}
```

另外像前面介绍的集合类型也都是泛型。

说到泛型，怎么能不说说特化(specialization)呢。很可惜的时候，目前特化属于实验特性，只有在nightly版本中才有。而且只支持trait特化。比如如下代码，我想根据类型的不同而提供不同的new方法。

```rust
#![feature(specialization)] // 引入特化特性

struct Point<T> {x: T, y: T}

trait New<T> {
    fn new(x: T, y: T) -> Self;
}

impl<T> New<T> for Point<T> {
    // 必须存在此default fn
    default fn new(x: T, y: T) -> Self {
        Point{x: x, y: y}
    }
}

impl New<f32> for Point<f32> {
    // 针对于f32类型的特化版本
    fn new(_x: f32, _y: f32) -> Self {
        Point{x: 10.0, y: 10.0}
    }
}

fn main() {
    let a = Point::new(1.0, 2.0);
    assert_eq!(a.x, 10.0);
    assert_eq!(a.y, 10.0);
}
```



### trait

这个就厉害了，对于传统的OO语言来说，继承是核心概念之一，但是在Rust里，没有“传统”继承的概念。

那么接口定义，以及接口的实现就靠trait[^2]

[^2]: trait本身是可以继承的，而且可以多重继承。但是这种继承实际上代表一种限制，也就是说`trait A: B`，代表的是任何类型实现了A都必须实现B。

trait和类型的行为有关。看下面的例子。

```rust
struct Duck;
struct Pig;

trait Fly {
    fn fly(&self) -> bool;  // 只有函数的签名，没有实现。实现不是必须的
}

impl Fly for Duck { 		// 把Fly的能力组合到Duck身上，并且返回true
    fn fly(&self) -> bool { 
        return true;
    }
}

impl Fly for Pig {			// 把Fly的能力组合到Fly身上，并且返回false。如果不组合，那么实际上30行会报错
    fn fly(&self) -> bool {
        return false;
    }
}

fn fly_s<T: Fly>(s: T) -> bool { // 静态分发机制。类似于C++特化，在编译时泛型已经被展开。
    return s.fly();
}

fn fly_d(s: &Fly) -> bool { 	// 动态分发机制。这里新的版本会提示告警，需要改成&dyn Fly， 具体见 https://zhuanlan.zhihu.com/p/109990547
    return s.fly();
}

fn main() {
    let pig = Pig;
    assert_eq!(fly_s(pig), false); //调用也可以写成fly_s<Pig>(pig)，实际使用可以省略类型，因为可以推导出来

    let duck = Duck;
    assert_eq!(fly_s(duck), true);
    assert_eq!(fly_d(&Duck), true);
}
```

Rust内置了很多trait，比如`Debug trait`，下面的例子给自己的类型添加trait，从而支持debug打印。

```rust
use std::fmt::*;
// fn fmt(&self, f: &mut Formatter<'_>) -> Result; // trait Debug中fmt方法的签名

struct Point {
    x: i32,
    y: i32,
}

impl Debug for Point {
    fn fmt(&self, f: &mut Formatter) -> Result {
        write!(f, "Point {这里应该没空格{ x: {}, y: {} }这里应该没空格}", self.x, self.y) // 由于gitbook的bug，不能连写两个{，会认为是变量，导致上传不上去。。。
    }
}

fn main() {
    let p = Point {x: 0, y:10};
    println!("{:?}", p);
}
```



## 错误处理

Rust没有异常，只有一个硬说很像但是极度暴力的panic（恐慌），它一出程序基本就挂了。

Rust通过返回`Result<T, E>`类型的方式来进行的，如果调用方不处理，会有编译告警。同时使用`?`运算符可以快速处理。

```rust
fn my_func_ret_err() -> Result<i32, String> {
    // return Err(String::from("cuola cuola"));
    return Ok(1);
}

fn caller() -> Result<i32, String> { // 处理方式1，使用？
    let ret = my_func_ret_err()?;
    return Ok(ret);
}

fn main() {
    let ret = caller();
    match ret {	// 处理方案2，使用match
        Ok(i) => {
            println!("OK, i={}", i);
        }
        Err(e) => {
            println!("Err, e={}", e);
        }
    }
}
```



具体可以参考这两个文章，对于Rust的错误处理机制讲的比较透彻、深入。

 [《Rust 错误处理》](https://zhuanlan.zhihu.com/p/25506762) 、 《[Rust 竟然没有异常处理？》](https://xie.infoq.cn/article/e9f2c50a00341279c204ed88c)
