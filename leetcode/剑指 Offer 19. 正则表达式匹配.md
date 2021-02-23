没有使用DP，直接使用DFS暴力解，50%的排名。

```rust
#[derive(Debug)]
enum PatternType {
    CHAR(char),
    DOT,
    ASTERISK(char)
}

fn parse_pattern(p: &String) -> Vec<PatternType> {
    let mut result = Vec::new();

    let mut last_c = ' ';

    for c in p.chars() {
        if c.is_alphabetic() {
            result.push(PatternType::CHAR(c));
        } else if c == '.' {
            result.push(PatternType::DOT);
        } else {
            result.pop();
            result.push(PatternType::ASTERISK(last_c));
        }
        last_c = c;
    }
    
    return result;
}

fn jump_index(index: usize, s: &Vec<char>, c: char) -> usize{
    let mut index = index;
    while index < s.len() && s[index] == c {
        index += 1;
    }
    return index;
}

fn help(s_index: usize, p_index: usize, s: &Vec<char>, p: &Vec<PatternType>) -> bool {
    if s_index >= s.len() && p_index >= p.len() {
        return true;
    } else if p_index >= p.len() {
        return false;
    }

    match p[p_index] {
        PatternType::CHAR(c) => {
            if s_index >= s.len() || s[s_index] != c {
                return false;
            } else {
                return help(s_index + 1, p_index + 1, s, p);
            }
        }

        PatternType::DOT => {
            if s_index >= s.len() {
                return false;
            } else {
                return help(s_index + 1, p_index + 1, s, p);
            }
        }

        PatternType::ASTERISK(c) => {
            if c == '.' {
                for i in s_index..=s.len() {
                    if help(i, p_index + 1, s, p) {
                        return true;
                    }
                }
                return false;
            }

            let new_s_index = jump_index(s_index, s, c);
            for i in s_index..=new_s_index {
                if help(i, p_index + 1, s, p) {
                    return true;
                }
            }
            return false;
        }
    }
}

impl Solution {
    pub fn is_match(s: String, p: String) -> bool {
        let p = parse_pattern(&p);
        let s = s.chars().collect::<Vec<char>>();

        help(0, 0, &s, &p)
    }

}
```
