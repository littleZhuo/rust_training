这道题是[剪绳子I](https://leetcode-cn.com/problems/jian-sheng-zi-lcof/)的升华，但是可恶的是直接用DP会有问题，因为用例会有超大的`n`，导致中间计算出来的结果翻转。

所以只能走另一条路，就是求出最佳的切割方法，然后贪心。这个切割方法一句话就是能切3就切3，不行就切2，也就是n个3和1个2|2个2。

但是证明的过程还没搞懂。

先把DP的解法放出来，可以解决120之内的解，到127的解就不行了。

```rust
use std::cmp::max;

impl Solution {
  pub fn cutting_rope(n: i32) -> i32 {
    let n = n as usize;
    let mut dp = vec![0u64; n + 1];
    for i in 2..=n {
      for j in 1..i {
        let temp_value = max((i - j) as u64, dp[i - j]);
        dp[i] = max(dp[i], j as u64 * temp_value as u64);
      } 
    }
    for len in dp.iter() {
      println!("{}", len);
    }
    return (dp[n] % 1000000007) as i32;
  }
}
```
