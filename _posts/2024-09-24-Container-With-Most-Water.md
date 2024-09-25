---
title: LeetCode 75-12 Container With Most Water
author: KanghoonYi(pour)
name: KanghoonYi(pour)
date: 2024-09-24 20:12:00 +0900
categories: [Programming, rust, leetcode75]
tags: [programming, rust, leetcode]
pin: false
math: true
---

## 문제 요약
integer array인 `height`안에서, 2개의 height를 골라 '물을 담을 수 있는 면적(2개의 `height`중 작은것 기준으로 측정해야함을 의미함.)'을 측정하여 가장 큰 면적을 return합니다.

## 문제 풀이

### 첫번째 시도

#### Complexity
**시간복잡도(time complexity)** 는 $O(n)$입니다. `height` array를 1회 순회합니다.  
**공간복잡도(space complexity)** 는 $O(1)$입니다. `height`의 크기와 비례해서 커지는 space는 없습니다.

왼쪽(height 시작점)에서 출발하는 pointer와 오른쪽(height 끝)에서 출발하는 pointer, 총 2개의 pointer를 이용합니다.  
여기서, pointer를 변경하는 부분이 중요한데, 이때, 문제의 요구사항인 'maximum amount of water a container can store'를 기억해야 합니다. 즉, height가 클수록 좋습니다(면적이 최대가 되어야 하기 때문에).  
이에 따라, `height`가 작은 쪽의 pointer를 이동시킵니다.

```rust
fn max_area(height: Vec<i32>) -> i32 {
    let mut left_pointer: usize = 0;
    let mut right_pointer: usize = height.len() - 1;

    let mut max_area: i32 = 0;
    while right_pointer - left_pointer > 0 {
        let area_width: i32 = (right_pointer - left_pointer) as i32;
        let area_height: i32 = std::cmp::min(height[left_pointer], height[right_pointer]);
        let area = area_width * area_height;
        max_area = std::cmp::max(area, max_area);

        if height[left_pointer] <= height[right_pointer] {
            left_pointer += 1;
        } else {
            right_pointer -= 1;
        }
    }
    return max_area;
}

fn main() {
    assert_eq!(max_area(Vec::from([1,8,6,2,5,4,8,3,7])), 49);
    assert_eq!(max_area(Vec::from([1, 1])), 1);
    assert_eq!(max_area(Vec::from([2,3,10,5,7,8,9])), 36);
}
```

#### 결과
![leetcode-13-submission-1](/assets/img/for-post/Container With Most Water/img.png)
_Leetcode 제출 결과_

## References

[Container With Most Water - LeetCode](https://leetcode.com/problems/container-with-most-water/description/?envType=study-plan-v2&envId=leetcode-75)
