---
title: Merge sort implementation with rust(rust로 병합정렬 구현)
author: KanghoonYi(pour)
name: KanghoonYi(pour)
date: 2024-09-01 19:00:00 +0900
categories: [Programming, rust]
tags: [programming, rust, sort]
pin: false
math: true
---
'merge sort'는 'Divide and conquer(분할과 정복)'을 사용한 대표적인 알고리즘입니다.
이 'Divide and conquer'는 문제를 작게 쪼개서 해결하고, 이를 다시 합침으로서 문제를 해결해 나가는걸 말합니다.
> 'Divide and conquer'는 여러 문제해결의 기본이 되곤 하며, 앞으로 부딪히는 문제가 너무 크게 느껴진다면, 이 전략으로 접근하면 도움이 됩니다.

'merge sort'에서는 총 2단계로 알고리즘의 단계를 구분합니다.
1. 분할(divide) 단계
: array를 반으로 나눕니다.

2. 정복(conquer) 단계(merge 단계)
: 나뉘어진 2개의 array를 하나로 합칩니다. 이때, 정렬이 이루어집니다.(2개의 array를 각각 보면서 크기를 비교하여 합칩니다)

## Time complexity and space complexity

### Time complexity(시간복잡도)
| Best case | Average  | Worst case |
|-----------|----------|------------|
| $O(n\log{n})$  | $O(n\log{n})$ | $O(n\log{n})$   |

어떤 상황이든 $O(n\log{n})$으로 안정적인 성능을 보여줍니다.  
여기서 n은 정렬 대상이 되는 array 전체를 1회 탐색해야하는것을 의미합니다.

알고리즘의 단계로 보면,

1. 분할(divide) 단계
: $O(\log{n})$, array를 반으로 줄여나가면서 함수의 실행횟수가 $O(\log{n})$이 됩니다.

2. 정복(conquer) 단계
: $O(n)$, 이 과정에서 정렬이 이루어지므로, 대상이 되는 array의 요소들을 1회씩 읽어야 합니다.

### Space Complexity(공간복잡도)
$$
O(n)
$$

array를 분할하는 과정에서 추가 공간이 필요합니다.

#### Merge sort with $O(1)$ space complexity
array의 element가 '양의 정수(unsigned integers)'일때, 수학적 trick을 이용해서 $O(1)$의 알고리즘을 만들 수 있다고 합니다.
[https://www.geeksforgeeks.org/merge-sort-with-o1-extra-space-merge-and-on-lg-n-time/](https://www.geeksforgeeks.org/merge-sort-with-o1-extra-space-merge-and-on-lg-n-time/)


## Implementation

### 첫번째 시도
```rust
use std::ops::DerefMut;
use rand::Rng;


fn generate_rand_array(size: usize) -> Vec<u32> {
    let mut rng = rand::thread_rng();
    let mut rand_arr= Vec::with_capacity(size);
    for i in 0..size {
        rand_arr.push(rng.gen_range(1..100));
    }
    return rand_arr
}

fn merge_sort(mut arr: Vec<u32>) -> Vec<u32> {

    merge_sort_partition(arr.deref_mut());
    return arr;
}

fn merge_sort_partition(arr: &mut [u32]) -> &mut [u32] {
    if (arr.len() <= 1) {
        return arr;
    }
    let mid = ((arr.len() / 2) as f64).floor() as usize;

    let (left_arr, right_arr) = arr.split_at_mut(mid);
    merge_sort_partition(left_arr);
    merge_sort_partition(right_arr);

    let sorted_left = left_arr.to_vec();
    let sorted_right = right_arr.to_vec();

    let mut left_idx = 0;
    let mut right_idx = 0;
    for idx in 0..arr.len() {
        let mut value;
        let left_val = if left_idx < sorted_left.len() {sorted_left[left_idx % sorted_left.len()]} else { u32::MAX };
        let right_val = if right_idx < sorted_right.len() { sorted_right[right_idx % sorted_right.len()] } else { u32::MAX };
        if (left_val < right_val) {
            value = left_val;
            left_idx += 1;
        } else {
            value = right_val;
            right_idx += 1;
        }

        arr[idx] = value;
    }

    return arr
}


fn main() {
    // random value로 이루어진 array 생성
    let mut arr = generate_rand_array(10);
    println!("{:?}", arr);
  // [94, 10, 54, 27, 47, 4, 69, 6, 93, 87] 출력

    let sorted_arr = merge_sort(arr);
    println!("{:?}", sorted_arr);
  // [4, 6, 10, 27, 47, 54, 69, 87, 93, 94] 출력
}
```

평가
알고리즘은 잘 작동하지만,chatGPT로부터 아래와 같은 feedback을 받았습니다.

- 불필요한 메모리 사용 제거할것
: `left_arr.to_vec()` 이 부분이 기존 Vec의 slice에 대한 pointer를 기준으로 새로운 Vec를 생성함으로써, 불필요하게 저장공간을 사용하고 있다.

- `deref_mut`사용하지 말것.
: 'Vec'의 Slice 기능을 사용할것.

### 두번째 시도
```rust

```
