---
title: LeetCode 75-10 Move Zeroes
author: KanghoonYi(pour)
name: KanghoonYi(pour)
date: 2024-09-21 18:55:00 +0900
categories: [Programming, rust, leetcode75]
tags: [programming, rust, leetcode]
pin: false
math: true
---

## 문제 요약
주어진 `nums`의 '0'을 모두 우측으로 몰고, '0'이 아닌 값은 원래 있던 순서대로 두어야 합니다.

## 문제 풀이

### 첫번째 시도
단순하게, '0'을 매우 큰 값으로 취급하여, array를 정렬하였습니다. 

#### Complexity
**시간복잡도(time complexity)** 는 $O(n\log{n})$입니다.    
'rust'의 array sort 기능을 이용해서 구현했고, rust의 sort algorithm은 [driftsort](https://doc.rust-lang.org/std/vec/struct.Vec.html#method.sort) 이기 때문에, 이와 같은 time complexity를 갖습니다. 

**공간복잡도(space complexity)** 는 $O(1)$입니다.

```rust
use std::cmp::Ordering;

fn move_zeroes(nums: &mut Vec<i32>) -> &Vec<i32> {
    nums.sort_by(|a, b| {
        if *a == 0 {
            return Ordering::Greater;
        } else if *b == 0 {
            return Ordering::Less;
        }
        return Ordering::Equal;
    });

    return nums;
}

fn main() {
    assert_eq!(*move_zeroes(&mut Vec::from([0,1,0,3,12])), [1,3,12,0,0]);
    assert_eq!(*move_zeroes(&mut Vec::from([0])), [0]);
}
```

#### 결과
![leetcode-10-submission-1](/assets/img/for-post/Move Zeroes/img.png)

### 두번째 시도
array를 역방향(마지막 부터 순회) 순회하여, '0'을 만나면, `remove`하고, 다시 `push`합니다.
이전보다 메모리 사용량은 개선되었지만, 여전히 'time complexity' 결과가 안 좋게 나옵니다.

#### Complexity
**시간복잡도(time complexity)** 는 $O(n^{2})$입니다.    
'rust'의 `remove`는 array의 element를 제거하고, 그 뒤를 잇는 나머지 element를 shifting(메모리 위치를 이동시킴)해야 합니다.  
때문에, `nums`순회 이외에, 그 길이 만큼 추가로 순회해야 합니다.

**공간복잡도(space complexity)** 는 $O(1)$입니다.

```rust
fn move_zeroes(nums: &mut Vec<i32>) -> &Vec<i32> {
    let mut idx: usize = nums.len() - 1;
    while idx >= 0 {
        let element = nums[idx];
        if element == 0 {
            nums.remove(idx);
            nums.push(element);
        }
        if idx == 0 {
            break;
        }
        idx -= 1;
    }

    return nums;
}

fn main() {
    assert_eq!(*move_zeroes(&mut Vec::from([0,1,0,3,12])), [1,3,12,0,0]);
    assert_eq!(*move_zeroes(&mut Vec::from([0])), [0]);
    assert_eq!(*move_zeroes(&mut Vec::from([0, 0, 1, 2, 3])), [1,2,3,0,0]);
}
```

#### 결과
![leetcode-10-submission-2](/assets/img/for-post/Move Zeroes/img_1.png)


### 세번째 시도
'0'의 위치를 기억하는 cursor 변수를 별도로 사용합니다(아래 코드에선, `zero_pos`).  
이후 `nums`를 순회하면서, '0'이 **아닌** 값을 만날때, 현재의 `zero_pos`에 위치한 값과 swap합니다.  

#### Complexity
**시간복잡도(time complexity)** 는 $O(n)$입니다.  
`nums`를 1회 순회하기 때문입니다.

**공간복잡도(space complexity)** 는 $O(1)$입니다.

```rust
fn move_zeroes(nums: &mut Vec<i32>) -> &Vec<i32> {
    let mut zero_pos: usize;
    match nums.iter().position(|num| *num == 0) { // 0인 값이 없으면, 함수 종료
        Some(value) => {
            zero_pos = value;
        }

        _ => { // default 실행
            return nums;
        }
    }

    if zero_pos >= nums.len() - 1 {
        return nums;
    }

    for idx in zero_pos+1..nums.len() {
        let element = nums[idx];
        if element != 0 {
            nums.swap(idx, zero_pos);
            zero_pos += 1;
        }
    }

    return nums;
}

fn main() {
    assert_eq!(*move_zeroes(&mut Vec::from([0,1,0,3,12])), [1,3,12,0,0]);
    assert_eq!(*move_zeroes(&mut Vec::from([0])), [0]);
    assert_eq!(*move_zeroes(&mut Vec::from([0, 0, 1, 2, 3])), [1,2,3,0,0]);
    assert_eq!(*move_zeroes(&mut Vec::from([2,1])), [2,1]);
}
```

#### 결과
![leetcode-10-submission-3](/assets/img/for-post/Move Zeroes/img_2.png)


## References

[Move Zeroes - LeetCode](https://leetcode.com/problems/move-zeroes/?envType=study-plan-v2&envId=leetcode-75)
