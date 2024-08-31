---
title: Bubble sort implementation with rust(rust로 버블정렬 구현)
author: KanghoonYi(pour)
name: KanghoonYi(pour)
date: 2024-08-30 19:00:00 +0900
categories: [Programming, rust]
tags: [programming, rust, sort]
pin: false
math: true
---
'bubble sort'는 가장 기본적인 sort algorithm(정렬 알고리즘)입니다.

시간 복잡도는 다음과 같습니다

|  | Best case | Average | Worst case |
| --- | --- | --- | --- |
| comparisions(비교연산 횟수) | O(n^2) | O(n^2) | O(n^2) |
| swaps(교환연산 횟수) | O(n^2) | O(n^2) | O(n^2) |

> 구현하는 방법에 따라, Best case의 시간복잡도를 $O(n)$(comparision 횟수)와 $O(1)$(swap 횟수)로 만들 수 있습니다
{: .prompt-info }

> ‘Best case‘는 array가 이미 정렬되어 있는 경우를 말합니다.
{: .prompt-info }

## Implementation

### Best case가 $O(n^2)$인 구현

```rust
use rand::Rng;

fn generate_rand_array(size: usize) -> Vec<u32> {
    let mut rng = rand::thread_rng();
    let mut rand_arr= Vec::with_capacity(size);
    for i in 0..size {
        rand_arr.push(rng.gen_range(1..100));
    }
    return rand_arr
}

fn bubble_sort(mut arr: Vec<u32>) -> Vec<u32> {
    for i in 0..arr.len() {
        let working_idx = arr.len() - 1 - i;
        for j in 0..working_idx + 1 {
            if j < working_idx && arr[j] > arr[j+1] {
                arr.swap(j, j + 1)
            }
        }
    }
    return arr;
}

fn main() {
    let arr = generate_rand_array(10);
    println!("{:?}", arr);
    let sorted_arr = bubble_sort(arr);
    println!("{:?}", sorted_arr);

    return;
}

```

### Best case가 $O(n)$인 구현

```rust
use rand::Rng;

fn generate_rand_array(size: usize) -> Vec<u32> {
    let mut rng = rand::thread_rng();
    let mut rand_arr= Vec::with_capacity(size);
    for i in 0..size {
        rand_arr.push(rng.gen_range(1..100));
    }
    return rand_arr
}

fn bubble_sort(mut arr: Vec<u32>) -> Vec<u32> {
    let mut is_swapped: bool = false;
    for i in 0..arr.len() {
        let working_idx = arr.len() - 1 - i;
        for j in 0..working_idx + 1 {
            if j < working_idx && arr[j] > arr[j+1] {
                arr.swap(j, j + 1);
                is_swapped = true;
            }
        }
        if is_swapped == false { // 1회 iteration후 swap이 일어났는지 판단합니다
            return arr;
        }
    }
    return arr;
}

fn main() {
    // random value로 이루어진 array 생성
    let arr = generate_rand_array(10);
    println!("{:?}", arr);

    // 정렬 수행
    let sorted_arr = bubble_sort(arr);
    println!("{:?}", sorted_arr);

    // 이미 정렬된 array인 경우
    let new_sorted_arr = bubble_sort(sorted_arr);
    println!("{:?}", new_sorted_arr);

    return;
}

```

## Reference

여러 sort algorithm의 과정을 animation으로 보기
: [Visualization and Comparison of Sorting Algorithms](https://youtu.be/ZZuD6iUe3Pc?si=FLtiIqBfjgZnO4B0)
{% include embed/youtube.html id='ZZuD6iUe3Pc' %}

source code repository
: [https://github.com/KanghoonYi/algorithm-study/blob/main/rust/src/bin/bubble_sort.rs](https://github.com/KanghoonYi/algorithm-study/blob/main/rust/src/bin/bubble_sort.rs)

