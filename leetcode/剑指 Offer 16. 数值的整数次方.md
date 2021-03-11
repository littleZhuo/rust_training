本题一开始使用暴力方法解，即for循环不停的乘，发现会超时。

然后根据`递归`标签提醒，加上`K神`的非递归解题思路，再加上有人的DP思路，整合了一个最终结果。

效率不差，但是因为用到了递归，所以内存占用不优。

思路：

对于x的n次幂，等价于 `x^n = x^(2*n/2) = (x^2)^(n/2)`，其中除号`/`是非地板除。

我们设`dp[x][n] = x^n`，则`dp[x][n] = dp[x^2][n/2] `。

进一步细化

- 当n为偶数时，`dp[x][n] = dp[x^2][n//2]`
- 当n为奇数时，`dp[x][n] = x * dp[x^2][n//2]`


```rust
fn help(x: f64, n: u32) -> f64 {
  if n  == 0 {
    return 1f64;
  } else if n == 1 {
    return x;
  }

  if n & 1 == 1 {
    return help(x * x, n / 2) * x;
  } else {
    return help(x * x, n / 2);
  }
}

impl Solution {
  pub fn my_pow(mut x: f64, n: i32) -> f64 {
    if x == 0f64 {
      return x;
    }

    let mut n = n as i64;
    if n < 0 {
      x = 1f64 / x;      
      n = -n;
    }

    return help(x, n as u32);
  }
}
```
