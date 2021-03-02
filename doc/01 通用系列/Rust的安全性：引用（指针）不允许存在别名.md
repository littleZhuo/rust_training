在学习Rust高级编程（The Rustonomicon，Rust死灵书）的"**别名**"一章时，发现了一块之前一直没有深究的黑暗区域，就是读写内存重叠。也就是连C标准库函数`memcpy`都害怕的问题。

这里放出来死灵书中对于别名的定义：
> 当变量和指针表示的内存区域有重叠时，它们互为对方的别名。

-----

先来看一段C代码

```c
void compute(const int * input, int *output)
{
    if (*input > 10) {
        *output = 1;
    }

    if (*input > 5) {
        *output *= 2;
    }
}
```

这段代码看起来并没有什么特别，输入和输出都是指针。主要逻辑：

-  判断如果输入大于10，则输出设置为1。
-  判断如果输入大于5，则输出翻倍。

---

现在我们决定优化代码

```c
void compute_improved(const int * input, int *output)
{
    if (*input > 10) {
        *output = 2;
    } else if (*input > 5) {
        *output *= 2;
    }
}

// 测试
    int input = 20;
    int output1 = 0;
    compute(&input, &output1);
    int output2 = 0;
    compute_improved(&input, &output2);
    printf("%d, %d\n", output1, output2); // 都是2
```

好了，现在逻辑清晰多了：

- 如果输入大于10，则输出为2（因为输入大于10则必然大于5，则`output = 1 * 2`）
- 否则，如果输入大于5，则输出为其翻倍。

---

当我为了自己的优化沾沾自喜时，我其实已经变更原函数的逻辑，因为原函数在输入和输出内存重叠时逻辑有很大的不同。比如

```c
    int input1 = 20;
    compute(&input1, &input1);
    int input2 = 20;
    compute_improved(&input2, &input2);
    printf("%d, %d\n", input1, input2); // 1 和 2
```

问题出在原函数对于输出有多次写操作，并且代码没有明确限制input和output不能重叠。那么就可能在第一次if判断成立后修改输入，导致后续逻辑变化。

我们可以通过优化输出的写操作来避免这种歧义：

```c
void compute_unambiguous(const int * input, int *output)
{
    int output_cache = output;
    
    if (*input > 10) {
        output_cache = 2;
    } else if (*input > 5) {
        output_cache *= 2;
    }
    
    *output = output_cache;
}
```

----

Rust里为什么安全？

```rust
fn compute(input: &u32, output: &mut u32) {
    if *input > 10 {
        *output = 1;
    }
    if *input > 5 {
        *output *= 2;
    }
}

fn main() {
    let mut x = 20;
    compute(&x, &mut x);  // 编译错误，cannot borrow `x` as mutable because it is also borrowed as immutable
}
```

Rust语言的规则就决定了：**不允许同时存在可变和不可变的引用**。所以这个问题在Rust中根本就不用担心，放心大胆的去重构就可以了。
