### Rust调用C语言的函数

涉及概念

- **外部函数接口** (*Foregin Function  Interface*, FFI)，其作用是以定义函数的方式，允许不同（外部）编程语言调用这些函数。
- **应用程序接口** (*Appliation Programming Interface*, ABI)，描述了应用程序和操作系统之间，一个应用和它的库之间，或者应用的组成部分之间的低接口。ABI涵盖了各种细节，如：
  - 数据类型的大小、布局和对齐；
  - 调用约定（控制着函数的参数如何传送以及如何接受返回值），例如，是所有的参数都通过栈传递，还是部分参数通过寄存器传递；哪个寄存器用于哪个函数参数；通过栈传递的第一个函数参数是最先push到栈上还是最后；
  - [系统调用](https://baike.baidu.com/item/系统调用)的编码和一个应用如何向操作系统进行系统调用；
  - 以及在一个完整的操作系统ABI中，[目标文件](https://baike.baidu.com/item/目标文件)的[二进制](https://baike.baidu.com/item/二进制/361457)格式、程序库等等。

具体举例如下，写一个调用C语言`int abs(int x)`的场景。

`lib.rs`

```rust
extern "C" {	// 关键字 extern + 语言，需要调用的函数的ABI
    fn abs(input: i32) -> i32;  // FFI
}

pub fn call_c_abs(input: i32) -> i32 {
    unsafe {
        abs(input)
    }
}
```



### C语言调用Rust的函数

#### 构建出静态库

##### `rustc`的方法

```powershell
rustc --crate-type=staticlib src\lib.rs
```



##### `cargo`的方法

修改`crate`的`Cargo.toml`文件，增加如下内容

```toml
[lib]
crate-type = ["staticlib"]
```

然后执行构建，在target里生成静态文件

```powershell
cargo build --lib
```



### 编写调用的C代码

`main.c`

```c
#include <stdint.h>
#include <stdio.h>

extern int32_t call_c_abs(int32_t input);

int main() {
    printf("%d\n", call_c_abs(-100));
    return 0;
}
```



#### 构建C代码

```powershell
gcc -o main .\main.c .\liblib.a
```



