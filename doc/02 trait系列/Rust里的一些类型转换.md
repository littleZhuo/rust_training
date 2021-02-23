写此文章的时候，还没有深入学习死灵书，所以有些理解和表述有问题。暂做记录。

## 遇到的问题

前几天搞workshop，遇到一个很棘手的问题：很多函数需要字符串作为入参，我一开始就是按照`&String`类型来定义，然后发现这样很多时候传过来的参数又都是`&str`。

这个问题就类似于C++里，我到底是要用`string`类型还是`char *`，如果也能像c++一样隐式类型转换该多好。



## C++的隐式类型转换是怎么样的?

C++中 `operator casting` 的效果

```c++
// c++ 代码
class A {
public: 
    operator B*() {
        return this->_b;
    }
    
    operator const B*() {
        return this->_b;
    }
        
private:
    B* _b;
}
```

或者构造函数的隐式类型转换的效果

```c++
// c++ 代码
class A {
public:
    A(): _num(0){}

    A(int num): _num(num){}

    void hello() const
    {
        std::cout << _num << std::endl;
    }

private:
    int _num;
};

void func(const A& a)
{
    a.hello();
}

int main() {
    func(A());  // print 0
    func(13);	// print 13
    return 0;
}
```



## 其实，Rust也支持隐式类型转换

常规操作，失败了。

```rust
fn func(input: &String) {
    println!("{}", input);
}

fn main() {
    let a = String::from("I'm String");
    func(&a);

    let b = "I'm str";
    func(b); // ^ expected struct `std::string::String`, found `str`
}
```

`From`  和 `Into` 是定义于 `std::convert` 模块中的两个`trait`。他们定义了互反操作 `from` 和 `into` ，被用来做类型的转换。

利用`Into trait`，全部OK~

```rust
use std::convert::Into;
use std::fmt::Display;

fn func<T>(input: T) where T : Into<String> + Display { // Into<String> 限制了类型T，表明可以转换到String类型上。 Display限制可以输出。
    println!("{}", input);
}

fn main() {
    let a = String::from("I'm String");
    func(&a);

    let b = "I'm str";
    func(b); 
}
```



​	
