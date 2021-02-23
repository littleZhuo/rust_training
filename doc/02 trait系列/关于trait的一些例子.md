## 实现类似加法的函数

这个例子里，我们实现一个类似 `my_add(a, b) -> c` 的方法。



### 给i32实现加法

```rust
trait Add<RHS, Output> {
    fn my_add(&self, rhs: RHS) -> Output;
}

// 给i32增加my_add操作
impl Add<i32, i32> for i32 {
    fn my_add(&self, rhs: i32) -> i32 {
        *self + rhs
        // return (*self + rhs);
    }
}

fn main() {
    let ia = 10i32;
    let ib = 11i32;
    let ic = ia.my_add(ib);
    assert_eq!(ic, 21);
}
```



### 给自定义类型实现加法

```rust
struct Point {
    x: i32,
    y: i32,
}

trait Add<RHS, Output> {
    fn my_add(&self, rhs: RHS) -> Output;
}


// 给Point增加my_add操作
impl Add<Point, Point> for Point {
    fn my_add(&self, rhs: Point) -> Point {
        Point{x: self.x + rhs.x, y: self.y + rhs.y}
    }
}

fn main() {
    let ia = Point{x: 1, y: 1};
    let ib = Point{x: 10, y: 10};
    let ic = ia.my_add(ib);
    assert_eq!(ic.x, 11);
    assert_eq!(ic.y, 11);
}
```

这里发散一下，函数的参数，会导致所有权转移么？我们修改代码，在最后尝试访问`ia`和`ib`。

```rust
fn main() {
    let ia = Point{x: 1, y: 1};
    let ib = Point{x: 10, y: 10};
    let ic = ia.my_add(ib);
    assert_eq!(ic.x, 11);
    assert_eq!(ic.y, 11);
    assert_eq!(ia.x, 1);  // 没问题，因为ia的所有权没有被借走
    assert_eq!(ib.x, 10); // 这里报错了 value borrowed here after move
}
```

所以我们的代码需要改一下，把`rhs`改成引用类型。

```rust
struct Point {
    x: i32,
    y: i32,
}

trait Add<RHS, Output> {
    fn my_add(&self, rhs: &RHS) -> Output; // 这里改了
}


// 给Point增加my_add操作
impl Add<Point, Point> for Point {
    fn my_add(&self, rhs: &Point) -> Point { // 这里改了 
        Point{x: self.x + rhs.x, y: self.y + rhs.y}
    }
}

fn main() {
    let ia = Point{x: 1, y: 1};
    let ib = Point{x: 10, y: 10};
    let ic = ia.my_add(&ib);  // 这里改了
    assert_eq!(ic.x, 11);
    assert_eq!(ic.y, 11);
    assert_eq!(ia.x, 1);
    assert_eq!(ib.x, 10); 
}
```

### 给自定义模板类型实现加法

让我们来更进一步，给自定义模板类型实现加法，老实说，我第一次直接实现这个难度，失败了~，编译错误一直无法解决。这次一步一步到了这个程度，有点期待。

#### 第一个版本

第一个编译过的版本，功能没问题，但是因为所有权转移，会导致操作数未初始化。

```rust
use std::ops::Add;

struct Point<T> {
    x: T,
    y: T,
}

trait MyAdd<RHS, Output> {
    fn my_add(self, rhs: RHS) -> Output;  // 改成了所有权转移
}


// 给Point<T>增加my_add操作
impl<T: Add<Output = T>> MyAdd<Point<T>, Point<T>> for Point<T> {
    fn my_add(self, rhs: Point<T>) -> Point<T> {
        Point{x: self.x + rhs.x, y: self.y + rhs.y}
    }
}

fn main() {
    let ia = Point{x: 1i32, y: 1i32};
    let ib = Point{x: 10i32, y: 10i32};
    let ic = ia.my_add(ib);
    assert_eq!(ic.x, 11);
    assert_eq!(ic.y, 11);
    // assert_eq!(ia.x, 1);   // 这里所有权转移了
    // assert_eq!(ib.x, 10);  // 同样
}
```

#### 优化重复声明

这里先不解决所有权问题，先解决代码复杂的问题，可以看到代码里`impl`的时候有大量的`Point<T>`重复书写，可以用`Self`类型替换。

```rust
impl<T: Add<Output = T>> MyAdd<Self, Self> for Point<T> {
    fn my_add(self, rhs: Self) -> Self {
        Self{x: self.x + rhs.x, y: self.y + rhs.y}
    }
}
```

更进一步，可以在trait定义的时候就把Self设置为默认值。

```rust
trait MyAdd<RHS = Self, Output = Self> { // 改了这里
    fn my_add(self, rhs: RHS) -> Output;
}

impl<T: Add<Output = T>> MyAdd for Point<T> {
    fn my_add(self, rhs: Self) -> Self {
        Self{x: self.x + rhs.x, y: self.y + rhs.y}
    }
}
```

#### 解决所有权的问题

第一种快准狠，好像也是标准库里Add的方式，用`Copy`和`Clone`的方法，这样在赋值的时候就不是所有权转移，而是值拷贝。

```rust
use std::ops::Add;

#[derive(Debug, Copy, Clone)] // 通过derive自动添加trait
struct Point<T> {
    x: T,
    y: T,
}

trait MyAdd<RHS = Self, Output = Self> {
    fn my_add(self, rhs: RHS) -> Output;
}

impl<T: Add<Output = T>> MyAdd for Point<T> {
    fn my_add(self, rhs: Self) -> Self {
        Self{x: self.x + rhs.x, y: self.y + rhs.y}
    }
}

fn main() {
    let ia = Point{x: 1, y: 1};
    let ib = Point{x: 10, y: 10};
    let ic = ia.my_add(ib);
    assert_eq!(ic.x, 11);
    assert_eq!(ic.y, 11);
    assert_eq!(ia.x, 1);
    assert_eq!(ib.x, 10); 
}
```

能不能用引用来解决这个问题，可以先就是引用的玩法。

```rust
use std::ops::Add;

#[derive(Debug, Copy, Clone)]
struct Point<T> {
    x: T,
    y: T,
} 

trait MyAdd<RHS = Self, Output = Self> {
    fn my_add(&self, rhs: &RHS) -> Output;
}

// 给Point<T>增加my_add操作
impl<T: Add<Output = T>> MyAdd for Point<T> where {
    fn my_add(&self, rhs: &Self) -> Self {
        Self{x: self.x + rhs.x, y: self.y + rhs.y}
    }
}

fn main() {
    let ia = Point{x: 1, y: 1};
    let ib = Point{x: 10, y: 10};
    let ic = ia.my_add(&ib);
    assert_eq!(ic.x, 11);
    assert_eq!(ic.y, 11);
    assert_eq!(ia.x, 1);
    assert_eq!(ib.x, 10); 
}
```

#### 进一步优化

这里采用的方式是标准库里`Add`的方式，因为一般返回值和self的类型是一致的，所以可以这么写

```rust
trait MyAdd<RHS = Self> {
    type Output;
    fn my_add(self, rhs: RHS) -> Self::Output;
}

// 给Point<T>增加my_add操作
impl<T: Add<Output = T>> MyAdd for Point<T> {
    type Output = Self;
    fn my_add(self, rhs: Output) -> Self::Output {
        Self{x: self.x + rhs.x, y: self.y + rhs.y}
    }
}
```



## 实现+操作符

```rust
use std::ops::Add;

#[derive(Debug, Copy, Clone)]
struct Point<T> {
    x: T,
    y: T,
}

// 给Point<T>增加add操作
impl<T: Add<Output = T>> Add for Point<T> {
    type Output = Self;
    fn add(self, rhs: Self) -> Self::Output {
        Self{x: self.x + rhs.x, y: self.y + rhs.y}
    }
}

// 给Point<T>增加一个和i32类型相加的操作
impl Add<i32> for Point<i32> {
    type Output = Self;
    fn add(self, rhs: i32) -> Self::Output {
        Self{x: self.x + rhs, y: self.y + rhs}
    }
}

fn main() {
    let ia = Point{x: 1i32, y: 1i32};
    let ib = Point{x: 10, y: 10};
    let ic = ia + ib;       // 直接走加号
    assert_eq!(ic.x, 11);
    assert_eq!(ic.y, 11);
    assert_eq!(ia.x, 1);
    assert_eq!(ib.x, 10); 
    let id = ia + 20i32;    // i32的Point和i32直接相加
    assert_eq!(id.x, 21);
}
```

这里还可以应用`where`关键字，提高代码的可读性。

```rust
// 给Point<T>增加add操作
impl<T> Add for Point<T> where T: Add<Output = T> { // 把前面的trait限定放到后面，前面就很清爽了。
    type Output = Self;
    fn add(self, rhs: Self) -> Self::Output {
        Self{x: self.x + rhs.x, y: self.y + rhs.y}
    }
}
```

