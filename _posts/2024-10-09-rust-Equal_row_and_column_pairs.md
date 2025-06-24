---
title: LeetCode 75-23 Equal Row and Column Pairs by Rust
author: KanghoonYi
name: KanghoonYi(pour)
date: 2024-10-09 18:24:00 +0900
categories: [Programming, rust, leetcode75]
tags: [programming, rust, leetcode]
pin: false
math: true
---

## 문제 요약
interger로 이루어진 2차원 배열이 주어지는데, 여기서 row 전체와 column전체가 일치하는 경우의 수를 구하는 문제입니다.

## 문제 풀이

### 첫번째 시도

#### Complexity
**시간복잡도(time complexity)** 는 $O(n^2)$입니다. 여기서 n은 `grid`matrix의 1차원(row혹은 column) 크기 입니다. 이는 `grid`를 전체 순회해야하기 때문입니다(column의 배열을 별도로 만들어야 하기 때문.).
**공간복잡도(space complexity)** 는 $O(n)$입니다. `grid`의 row를 순회하며 frequency hashmap을 생성하기 때문입니다.

먼저 row를 순회하며, interger pattern을 파악하고, frequency를 기록합니다.  
이때, row의 pattern을 hashmap의 key값으로 사용합니다.  
이후 column을 순회하며 hashmap에 이미 있는 pattern인지 확인한 후 기록합니다.  
column에 대한 key를 만들기 위해, 별도의 buffer array를 사용하여, 매번 array가 생성되는것을 방지하였습니다.

```rust
use std::collections::HashMap;

fn equal_pairs(grid: Vec<Vec<i32>>) -> i32 {
    let n = grid.len();
    let mut frequency_table = HashMap::new();
    for row in &grid {
        frequency_table.entry(row).and_modify(|counter| *counter += 1).or_insert(1);
    }

    let mut count = 0;
    // let mut buffer_vec: Vec<i32> = Vec::with_capacity(n);
    let mut buffer_vec: Vec<i32> = vec![0; n];
    buffer_vec.fill(0);
    for col_idx in 0..n {
        for row_idx in 0..n {
            buffer_vec[row_idx] = grid[row_idx][col_idx];
        }
        let row_count = *frequency_table.get(&buffer_vec).unwrap_or(&0);
        if row_count >= 1 {
            count += row_count * 1;
        }
    }

    return count;
}

fn main() {
    assert_eq!(equal_pairs(Vec::from([Vec::from([3,2,1]), Vec::from([1,7,6]), Vec::from([2,7,7])])), 1);
    assert_eq!(equal_pairs(Vec::from([Vec::from([3,1,2,2]), Vec::from([1,4,4,5]), Vec::from([2,4,2,2]), Vec::from([2,4,2,2])])), 3);
}
```

#### 결과
![leetcode-23-submission-1](/assets/img/for-post/Equal%20Row%20and%20Column%20Pairs/rust_submission_1.png)
_Leetcode 제출 결과_

## References

[Equal Row and Column Pairs - LeetCode](https://leetcode.com/problems/equal-row-and-column-pairs/?envType=study-plan-v2&envId=leetcode-75)
