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

至于`PartialEq`和`Eq`感觉非必要，应该是为了测试方便，本地删除后树还是可以正常建立。

```rust
struct Solution;

// Definition for a binary tree node.
#[derive(Debug, PartialEq, Eq)]
pub struct TreeNode {
  pub val: i32,
  pub left: Option<Rc<RefCell<TreeNode>>>,
  pub right: Option<Rc<RefCell<TreeNode>>>,
}

impl TreeNode {
  #[inline]
  pub fn new(val: i32) -> Self {
    TreeNode {
      val,
      left: None,
      right: None
    }
  }
}

use std::rc::Rc;
use std::cell::RefCell;

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

fn main() {
    let r = Solution::build_tree(vec![3,9,20,15,7], vec![9,3,15,20,7]);
    println!("{:?}", r);
}
```
