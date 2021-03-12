这是我完完整整刷的第一道有限状态机（FSM）的题目，对于个人而言知识点很多。

首先这里不再啰嗦状态的定义和转移方程，官方已经写得很清楚了。

总结一下此类题目的思路：

1. 枚举所有的状态，这里面需要注意如果状态由关联的时候，需要拆分，比如小数点左右必须至少一侧有数，那么就将小数点拆分成`左侧有数小数点`和`左侧无数小数点`；
2. 整理状态转移的规则；
3. 找到初始状态和结束状态；
4. 开始编码，最重要的就是使用数组或者Map实现状态转移规则。

在Rust里我选择了`HashMap`，但是很可惜没有很方便的方法能够快捷的初始化多层嵌套的`HashMap`，可以想到的办法有：

0. 手工不停的insert；
1. 自己写宏；
2. 使用别人的crates；
3. 利用[(key, value),...].iter().cloned().collect()嵌套生成；

我自己使用了`3`，整洁度还好，就是有点啰嗦。

代码我拆分成两部分，把`hashMap`初始化的代码拆开，不影响整体逻辑的阅读流程度。

主题部分：

```rust
struct Solution;

use std::collections::HashMap;
#[derive(Debug, Clone, Copy, PartialEq, Eq, Hash)]
enum State {
  HeadSpace,
  Integersign,
  Integer,
  LeftExistsDot,
  LeftNoneDot,
  Decimal,
  CharE,
  ExpSign,
  Exp,
  TailSpace,
}

#[derive(Debug, Clone, Copy, PartialEq, Eq, Hash)]
enum CharType {
  CharNumber,
  CharE,
  CharSign,
  CharDot,
  CharSpace,
  // CharIllegal,
}

trait ToCharType {
  fn to_char_type(&self) -> Option<CharType>;
}

impl ToCharType for char {
  fn to_char_type(&self) -> Option<CharType> {
      if self.is_numeric() { return Some(CharType::CharNumber); }
      if ['E', 'e'].contains(self) { return Some(CharType::CharE); }
      if ['+', '-'].contains(self) { return Some(CharType::CharSign); }
      if *self == ' ' { return Some(CharType::CharSpace); }
      if *self == '.' { return Some(CharType::CharDot); }
      return None;
  }
}

#[derive(Debug)]
struct MachineChg {
  rule: HashMap<State, HashMap<CharType, State>>,
}

impl MachineChg {
  fn next_state(&self, cur_state: &State, char_type: &CharType) -> Option<&State> {
    match self.rule.get(cur_state) {
      Some(nexts) => {
        nexts.get(char_type)
      },
      None => None
    }
  }
}

impl Solution {
  pub fn is_number(s: String) -> bool {
    let mut st = State::HeadSpace;
    let mc = MachineChg::new();
    for c in s.chars() {
      if let Some(char_type) = c.to_char_type() {
        if let Some(new_state) = mc.next_state(&st, &char_type) {
          st = *new_state;
          continue;
        }
      }
      return false;
    }
    if [State::Integer, State::LeftExistsDot, State::Decimal, State::Exp, State::TailSpace].contains(&st) {
      return true;
    }
    false
  }
}

fn main() {
  println!("{}", Solution::is_number(String::from("46.e3")));
}
```

状态转移的map初始化部分：

```rust
impl MachineChg {
  fn new() -> Self {
    MachineChg {rule:
      [
        (
          State::HeadSpace, [
            (CharType::CharSpace, State::HeadSpace),
            (CharType::CharSign, State::Integersign),
            (CharType::CharNumber, State::Integer),
            (CharType::CharDot, State::LeftNoneDot),
          ].iter().cloned().collect::<HashMap::<CharType, State>>()
        ),
        (
          State::Integersign, [
            (CharType::CharNumber, State::Integer),
            (CharType::CharDot, State::LeftNoneDot),
          ].iter().cloned().collect::<HashMap::<CharType, State>>(),
        ),
        (
          State::Integer, [
            (CharType::CharNumber, State::Integer),
            (CharType::CharDot, State::LeftExistsDot),
            (CharType::CharE, State::CharE),
            (CharType::CharSpace, State::TailSpace),
          ].iter().cloned().collect::<HashMap::<CharType, State>>()
        ),
        (
          State::LeftExistsDot, [
            (CharType::CharNumber, State::Decimal),
            (CharType::CharE, State::CharE),
            (CharType::CharSpace, State::TailSpace),
          ].iter().cloned().collect::<HashMap::<CharType, State>>()
        ),
        (
          State::LeftNoneDot, [
            (CharType::CharNumber, State::Decimal),
          ].iter().cloned().collect::<HashMap::<CharType, State>>()
        ),
    
        (
          State::Decimal, [
            (CharType::CharNumber, State::Decimal),
            (CharType::CharE, State::CharE),
            (CharType::CharSpace, State::TailSpace),
          ].iter().cloned().collect::<HashMap::<CharType, State>>()
        ),
        (
          State::CharE, [
            (CharType::CharSign, State::ExpSign),
            (CharType::CharNumber, State::Exp),
          ].iter().cloned().collect::<HashMap::<CharType, State>>()
        ),
        (
          State::ExpSign, [
            (CharType::CharNumber, State::Exp),
          ].iter().cloned().collect::<HashMap::<CharType, State>>()
        ),
        (
          State::Exp, [
            (CharType::CharNumber, State::Exp),
            (CharType::CharSpace, State::TailSpace),
          ].iter().cloned().collect::<HashMap::<CharType, State>>()
        ),
        (
          State::TailSpace, [
            (CharType::CharSpace, State::TailSpace),
          ].iter().cloned().collect::<HashMap::<CharType, State>>()
        ),
      ].iter().cloned().collect()
    }
  }
}
```
