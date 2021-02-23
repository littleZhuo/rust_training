疑难点见注释

```rust
impl Solution {
    pub fn find_number_in2_d_array(matrix: Vec<Vec<i32>>, target: i32) -> bool {
        let (n, m) = (matrix.len(), if matrix.is_empty() { 0 } else { matrix[0].len() });
        let mut row: usize = 0;
        let mut column: usize = m - 1;
        while row < n && column < m { // column < m 其实就是 column >= 0， 但是由于其为usize类型，如果运算 0 - 1 则会翻转编为很大一个数
            match matrix[row][column].cmp(&target) { // 这里用match比较好，之前我是写的if判断，很啰嗦
                std::cmp::Ordering::Less => row += 1,
                std::cmp::Ordering::Greater => column -= 1,
                std::cmp::Ordering::Equal => return true,
            }
        }
        false
    }
}
```
