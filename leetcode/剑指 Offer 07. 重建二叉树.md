这道题目首先用python实现了一遍，比较简单，[链接](https://github.com/littleZhuo/python_training/blob/main/leetcode/%E5%89%91%E6%8C%87%20Offer%2007.%20%E9%87%8D%E5%BB%BA%E4%BA%8C%E5%8F%89%E6%A0%91.md)。
但是一下就被题目里Rust的嵌套容器给吓着了。

```rust
// Definition for a binary tree node.
// #[derive(Debug, PartialEq, Eq)]
// pub struct TreeNode {
//   pub val: i32,
//   pub left: Option<Rc<RefCell<TreeNode>>>,
//   pub right: Option<Rc<RefCell<TreeNode>>>,
// }
// 
// impl TreeNode {
//   #[inline]
//   pub fn new(val: i32) -> Self {
//     TreeNode {
//       val,
//       left: None,
//       right: None
//     }
//   }
// }
use std::rc::Rc;
use std::cell::RefCell;
impl Solution {
    pub fn build_tree(preorder: Vec<i32>, inorder: Vec<i32>) -> Option<Rc<RefCell<TreeNode>>> {

    }
}
```

参考python的代码，rust的代码差异主要有两点：

1. 因为所有权的关系，树是用`Option<Rc<RefCell<TreeNode>>>`来封装的，Option是因为节点可能为空，Rc感觉非必须，所以RefCell也感觉非必须，还没实验。
2. 另外为了减少递归过程中不必要的拷贝或者所有权转移，所有用了引用。

> 打脸来得很快，对智能指针的学习还是不认真，对于递归定义的类型，如果不引入智能指针，编译器就无法知道类型大小。报错如下：
```rust
5 | pub struct TreeNode {
  | ^^^^^^^^^^^^^^^^^^^ recursive type has infinite size
6 |   pub val: i32,
7 |   pub left: Option<TreeNode>,
  |             ---------------- recursive without indirection
8 |   pub right: Option<TreeNode>,
  |              ---------------- recursive without indirection
```
> 所以Rc是必须的，因为不存在共享所有权的情况，换成Box也行。
> 
> 但是如果没有RefCell，那么修改就成了问题。

至于`PartialEq`和`Eq`感觉非必要，应该是为了测试方便，本地删除后树还是可以正常建立。

```rust
fn helper(preorder: &[i32], inorder: &[i32]) -> Option<Rc<RefCell<TreeNode>>> {
    if preorder.len() == 0 {
        return None;
    }

    let mut root = TreeNode::new(preorder[0]);

    let in_pos = inorder.iter().position(|&i| i == preorder[0]).unwrap();
    let left_in = &inorder[..in_pos];
    let right_in = &inorder[in_pos+1..];

    let out_pos = 1 + left_in.len();

    root.left = helper(&preorder[1..out_pos], left_in);
    root.right = helper(&preorder[out_pos..], right_in);

    return Some(Rc::new(RefCell::new(root)));
}

impl Solution {
    pub fn build_tree(preorder: Vec<i32>, inorder: Vec<i32>) -> Option<Rc<RefCell<TreeNode>>> {
        helper(&preorder[..], &inorder[..])
    }
}
```
