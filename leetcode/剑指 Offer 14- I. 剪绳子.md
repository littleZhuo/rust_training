DP类问题最重要的就是找到转移方程，这道题也不例外。而且相对以往的其他题目，转移方程要简单不少。

首先题目里没有明确绳子具体可以切几段，只是要求乘积最大。

那我们来整理一个一般的公式：假设有一段绳子，长度是`i(i >= 2)`，在左侧切一刀，长度是`j`；剩下的一部分长度就是`i - j`，可以选择继续切或者保留整段。

定义这段绳子切分后的最大长度是`dp(i)`，那么对于切固定的一刀`j`来说，`dp(i) = j * max(i - j, dp(i - j))`。

同时`j`的范围是`[1, i - 1]`，所以需要遍历所有的j的情况求最大值，就是`dp(i)`

边界条件是`dp(0)和dp[1]`等于0，以及`dp[2]=1`。

```rust
use std::cmp::max;

fn dp(i: usize, mem: &Vec<usize>) -> usize {
  if i <= 1 {
    return 0;
  }

  if i == 2 { // 按照要求应该不会小于2
    return 1;
  }

  let mut max_value = 0;
  for j in 1..i-1 {
    let temp_value = max(i - j, mem[i - j]);
    max_value = max(max_value, j * temp_value);
  }
  return max_value;
}

impl Solution {
  pub fn cutting_rope(n: i32) -> i32 {
    let mut mem = vec![0usize; n as usize + 1];
    for i in 1..=n {
      mem[i as usize] = dp(i as usize, &mem);
    }

    return mem[n as usize] as i32;
  }
}
```

精简以后的代码如下：

```rust
use std::cmp::max;

impl Solution {
  pub fn cutting_rope(n: i32) -> i32 {
    let n = n as usize;
    let mut dp = vec![0usize; n + 1];
    for i in 2..=n {
      for j in 1..i {
        let temp_value = max(i - j, dp[i - j]);
        dp[i] = max(dp[i], j * temp_value);
      } 
    }
    return dp[n] as i32;
  }
}
```
