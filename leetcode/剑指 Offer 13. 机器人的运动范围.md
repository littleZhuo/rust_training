是比较简单图的遍历问题，但是由于不需要求最短路径，所以BFS和DFS都可以。

但是肯定是BFS最快,Rust里效率100%。

思路是：

1. 首先构造二维数组，默认所有位置都是`1`，即可达
2. 根据规则，把所有不可达的位置标`0`，即不可达
3. 然后就是典型的BFS，把所有走过的位置都置为0，一层一层递进

```rust
struct Solution;

fn cacl_bit_sum(mut m: i32, mut n: i32) -> i32 {
    let mut sum = 0i32;
    while m > 0 {
        sum += m % 10;
        m = m / 10;
    }
    while n > 0 {
        sum += n % 10;
        n = n / 10;
    }
    sum
}

fn build_matrix_with_vec(m: i32, n: i32, k: i32) -> Vec<Vec<u8>> {
    let mut map = vec![vec![1u8; n as usize]; m as usize];
    for i in 0..m {
        for j in 0..n {
            if cacl_bit_sum(i, j) > k {
                map[i as usize][j as usize] = 0u8;
            }
        }
    }
    map
}

#[derive(Clone)]
struct Point {
    x: usize,
    y: usize,
}

const DIR: [[i32; 4]; 2] = [[0, 1, 0, -1], [1, 0, -1, 0]];

fn get_arrived_points(map: &mut Vec<Vec<u8>>, cur_point: &Point) -> Vec<Point>
{
    let mut arrived_points = Vec::new();

    for i in 0..4 {
        let (x, y) = (cur_point.x as i32 + DIR[0][i], cur_point.y as i32 + DIR[1][i]);

        if x < 0 || y < 0 || x as usize >= map.len() || y as usize >= map[0].len() || map[x as usize][y as usize] == 0 {
            continue;
        }

        map[x as usize][y as usize] = 0;
        arrived_points.push(Point {x: x as usize, y: y as usize});
    }

    arrived_points    
}

fn bfs(map: &mut Vec<Vec<u8>>, cur_points: &Vec<Point>) -> u32 {
    if cur_points.len() == 0 {
        return 0;
    }

    let mut next_points: Vec<Point> = Vec::new();

    for cp in cur_points {
        next_points.append(&mut(get_arrived_points(map, cp)));
    }

    next_points.len() as u32 + bfs(map, &next_points)
}

impl Solution {
    pub fn moving_count(m: i32, n: i32, k: i32) -> i32 {
        if m == 0 || n == 0 { return 0; }
        let mut map = build_matrix_with_vec(m, n, k);
        let cur_points: Vec<Point> = vec![Point{x: 0, y: 0}];
        map[0][0] = 0;
        return 1 + bfs(&mut map, &cur_points) as i32;
    }
}

fn main() {
    let count = Solution::moving_count(2, 3, 1);
    println!("{}", count);
}

```
