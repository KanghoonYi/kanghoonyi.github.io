---
title: LeetCode 75-23 Equal Row and Column Pairs by python
author: KanghoonYi
name: KanghoonYi(pour)
date: 2024-10-09 19:00:00 +0900
categories: [Programming, python, leetcode75]
tags: [programming, python, leetcode]
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
`rust` code와 다른점은, hashmap의 key를 string으로만 설정할 수 있어, 'type casting(형 변환, 타입 변환)'을 추가하였습니다.


```python
from typing import List

def equalPairs(grid: List[List[int]]) -> int:
    n = len(grid)
    frequency_dict = dict()
    for row_idx in range(0, n):
        row_str = ','.join(map(str, grid[row_idx]))
        if row_str in frequency_dict:
            frequency_dict[row_str] += 1
        else:
            frequency_dict[row_str] = 1

    count = 0

    buffers = [''] * n
    for col_idx in range(0, n):
        for row_idx in range(0, n):
            buffers[row_idx] = str(grid[row_idx][col_idx])
        key = ','.join(buffers)
        if key in frequency_dict:
            count += frequency_dict[key] * 1

    return count

assert equalPairs([[3,2,1],[1,7,6],[2,7,7]]) == 1
assert equalPairs([[3,1,2,2],[1,4,4,5],[2,4,2,2],[2,4,2,2]]) == 3
```

#### 결과
![leetcode-23-submission-1](/assets/img/for-post/Equal%20Row%20and%20Column%20Pairs/python_submission_1.png)
_Leetcode 제출 결과_

## References

[Equal Row and Column Pairs - LeetCode](https://leetcode.com/problems/equal-row-and-column-pairs/?envType=study-plan-v2&envId=leetcode-75)
