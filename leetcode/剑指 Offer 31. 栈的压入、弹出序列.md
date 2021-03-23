```rust
impl Solution {
    pub fn validate_stack_sequences(pushed: Vec<i32>, popped: Vec<i32>) -> bool {
        let mut secondary_stack: Vec<i32> = Vec::with_capacity(pushed.len());
        let mut i_pop = 0;
        for num in pushed {
            secondary_stack.push(num);
            while !secondary_stack.is_empty() && *secondary_stack.last().unwrap() == popped[i_pop] {
                secondary_stack.pop();
                i_pop += 1;
            }
        }
        return secondary_stack.is_empty();
    }
}
```
