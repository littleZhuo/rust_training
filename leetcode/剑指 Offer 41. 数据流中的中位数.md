依旧还是先用python实现了一遍，具体看[这里](https://github.com/littleZhuo/python_training/blob/main/leetcode/%E5%89%91%E6%8C%87%20Offer%2041.%20%E6%95%B0%E6%8D%AE%E6%B5%81%E4%B8%AD%E7%9A%84%E4%B8%AD%E4%BD%8D%E6%95%B0.md)

切换到Rust里，好在Rust有二叉堆的数据结构，不需要重头实现。这里遇到需要注意的有几个地方：

1. `python`里我们使用`取反`来保证堆排序顺序的颠倒，在`Rust`里有更优雅的办法--`std::cmp::Reverse`。
2. 因为使用`Reverse`封装了小顶堆，所以在小顶堆弹出堆顶给大顶堆的时候，就需要`解包`，我用的是直接`.0`来访问，没找到对应的方法。
3. rust不像python有隐式类型转换，堆里都是i32，但是最后的中位数是f64，所以需要显示的类型转换，确保除数、被除数都是f64才能计算。

```rust
use std::{cmp::Reverse, collections::BinaryHeap, convert::TryInto};

struct MedianFinder {
  big_heap: BinaryHeap<Reverse<i32>>,    // 较大部分的堆，使用小顶堆
  small_heap: BinaryHeap<i32>,  // 较小部分的堆，使用大顶堆
}


/**
* `&self` means the method takes an immutable reference.
* If you need a mutable reference, change it to `&mut self` instead.
*/
impl MedianFinder {

  /** initialize your data structure here. */
  fn new() -> Self {
      Self {big_heap: BinaryHeap::new(), small_heap: BinaryHeap::new()}
  }
  
  fn add_num(&mut self, num: i32) {
    if self.big_heap.len() != self.small_heap.len() {
      self.big_heap.push(Reverse(num));
      self.small_heap.push(self.big_heap.pop().unwrap().0);
    } else {
      self.small_heap.push(num);
      self.big_heap.push(Reverse(self.small_heap.pop().unwrap()));
    }
  }
  
  fn find_median(&self) -> f64 {
    if self.big_heap.len() == self.small_heap.len() {
      return (self.big_heap.peek().unwrap().0 + self.small_heap.peek().unwrap()) as f64 / 2f64;
    } else {
      return self.big_heap.peek().unwrap().0 as f64;
    }
  }
}

/**
* Your MedianFinder object will be instantiated and called as such:
* let obj = MedianFinder::new();
* obj.add_num(num);
* let ret_2: f64 = obj.find_median();
*/

fn main() {
  let mut obj = MedianFinder::new();
  obj.add_num(1);
  obj.add_num(2);
  println!("{}",obj.find_median());
  obj.add_num(3);
  println!("{}",obj.find_median());
}

```
