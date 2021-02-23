[TOC]

Rust BOOK里的一个例子，讲`trait`对象。

大概思路是有一个Draw的trait；然后有一个Screen类型，内部有一个存储不同组件类型的Vec，它还有一个方法run，可以依次调用每个组件的draw方法。

### trait对象

```rust
pub trait Draw {
    fn draw(&self);
}

pub struct Screen {
    pub components: Vec<Box<dyn Draw>>, // dyn，动态分发
}

impl Screen {
    pub fn run(&self) {
        for component in self.components.iter() {
            component.draw();
        }
    }
}

pub struct Button {
    pub width: u32,
    pub height: u32,
    pub label: String,
}

impl Draw for Button {
    fn draw(&self) {
        println!("This Button:({}, {}, {})", self.width, self.height, self.label);
    }
}

pub struct SelectBox {
    pub width: u32,
    pub height: u32,
    pub options: Vec<String>,
}

impl Draw for SelectBox {
    fn draw(&self) {
        println!("This SelectBox:({}, {}, {:?})", self.width, self.height, self.options);
    }
}

fn main() {
    let screen = Screen { // 可以同时放入SelectBox和Button
        components: vec![
            Box::new(SelectBox {
                width: 1,
                height: 1,
                options: vec![
                    String::from("Yes"),
                    String::from("No"),
                ]
            }),
            Box::new(Button {
                width: 2,
                height: 2,
                label: String::from("Lable"),
            })
        ]
    };

    screen.run();
}
```



### 泛型加trait bound

```rust
pub trait Draw {
    fn draw(&self);
}

pub struct Screen<T> where T: Draw { // 泛型Screen
    pub components: Vec<Box<T>>,
}

impl<T> Screen<T> where T: Draw { // 对泛型T做trait bound
    pub fn run(&self) {
        for component in self.components.iter() { 
            component.draw();
        }
    }
}

pub struct Button {
    pub width: u32,
    pub height: u32,
    pub label: String,
}

impl Draw for Button {
    fn draw(&self) {
        println!("This Button:({}, {}, {})", self.width, self.height, self.label);
    }
}

pub struct SelectBox {
    pub width: u32,
    pub height: u32,
    pub options: Vec<String>,
}

impl Draw for SelectBox {
    fn draw(&self) {
        println!("This SelectBox:({}, {}, {:?})", self.width, self.height, self.options);
    }
}

fn main() {
    let screen = Screen {
        components: vec![
            Box::new(SelectBox { // 第一个元素没有问题
                width: 1,
                height: 1,
                options: vec![
                    String::from("Yes"),
                    String::from("No"),
                ]
            }),
            Box::new(Button { // 这里会报错，提示期望是SelectBox，因为screen已经因为第一个元素而被推导为T=SelectBox
                width: 2,
                height: 2,
                label: String::from("Lable"),
            })
        ]
    };

    screen.run();
}
```

### 结论

综上，Trait对象可以使得**不同类型**的值行为统一。而对于泛型，Rust 通过在编译时进行泛型代码的 **单态化**（*monomorphization*）来保证效率。单态化是一个通过填充编译时使用的具体类型，将通用代码转换为特定代码的过程，所以注定了同时只能接受具体的一种类型。

- trait对象，动态分发（dynamic dispatch，**dyn**），灵活性高，性能差。
- 泛型加trait bound，静态分发（static dispatch），灵活性差，性能高。
