---
title: LeetCode 75-13 Maximum Average Subarray 1
author: KanghoonYi(pour)
name: KanghoonYi(pour)
date: 2024-09-26 17:14:00 +0900
categories: [Programming, rust, leetcode75]
tags: [programming, rust, leetcode]
pin: false
math: true
---

## 문제 요약
integer array인 `nums`안에서, `k`에 해당하는 길이만큼의 요소를 뽑습니다. 이때, 이 요소들의 평균값의 최대를 return합니다.  
즉, `k`길이의 subarray에 대한 average의 최댓값을 구합니다.

## 문제 풀이

### 첫번째 시도

#### Complexity
**시간복잡도(time complexity)** 는 $O(n)$입니다. 이는 `nums`를 2회 순회하지만, 각 순회가 독립적으로 이루어지 때문입니다.  
**공간복잡도(space complexity)** 는 $O(n)$입니다. 주어진 `nums`를 순회하며, 각 element의 합을 accumulation(축적, 누적)하여 별도의 array에 저장하기 때문입니다.

평균은 각 요소의 합에 비례합니다. 때문에, subarray 요소의 합이 최대가 되면, 평균(average) 값이 최대가 됩니다.  
또한, 'sum'의 값으로 데이터를 저장하고 있는게, 조금이나마 메모리 사용에 이점이 있습니다(실수인 f64는 64bit의 저장공간을 사용. 반면에 정수인 i32는 32bit의 저장공간을 사용.).  
subarray의 합을 구하기 위해, array를 1회 순회하며 각 단계마다 총합을 누적하여 별도의 array에 저장합니다.  
이는 아래의 수식을 통해 subarray의 합을 구하는데 사용됩니다.

`nums`의 index n부터 m 까지의 subarray의 합은, 다음과 같은 수식으로 표현됩니다.
$$$
\sum_{i=n}^{m}\text{num}_i = \sum_{i=0}^{m}\text{num}_i - \sum_{i=0}^{n}\text{num}_i
$$$
이를 이용하여, subarrary의 합을 구하는 것을 단순화하고 최댓값을 찾습니다.

```rust
fn find_max_average(nums: Vec<i32>, k: i32) -> f64 {
    if nums.len() <= 0 {
        panic!("num\'s length must greater than 0.")
    } else if nums.len() <= 1 {
        return nums[0] as f64;
    }

    let mut sum_array: Vec<i32> = Vec::with_capacity(nums.len());

    let mut sum_accu = 0;
    for element in &nums {
        sum_accu += element;
        sum_array.push(sum_accu)
    }

    let mut max_sum: i32 = i32::MIN;
    for start_pointer in 0..=nums.len() - k as usize {
        let end_pointer = start_pointer + k as usize - 1;
        let start_sum = sum_array[start_pointer];
        let end_sum = sum_array[end_pointer];

        max_sum = std::cmp::max(end_sum - start_sum + nums[start_pointer], max_sum);
    }

    return max_sum as f64 / k as f64;
}

fn main() {
    assert_eq!(find_max_average(Vec::from([1,12,-5,-6,50,3]), 4), 12.75000);
    assert_eq!(find_max_average(Vec::from([5]), 1), 5.00000);
    assert_eq!(find_max_average(Vec::from([4,0,4,3,3]), 5), 2.80000);
    assert_eq!(find_max_average(Vec::from([8860,-853,6534,4477,-4589,8646,-6155,-5577,-1656,-5779,-2619,-8604,-1358,-8009,4983,7063,3104,-1560,4080,2763,5616,-2375,2848,1394,-7173,-5225,-8244,-809,8025,-4072,-4391,-9579,1407,6700,2421,-6685,5481,-1732,-8892,-6645,3077,3287,-4149,8701,-4393,-9070,-1777,2237,-3253,-506,-4931,-7366,-8132,5406,-6300,-275,-1908,67,3569,1433,-7262,-437,8303,4498,-379,3054,-6285,4203,6908,4433,3077,2288,9733,-8067,3007,9725,9669,1362,-2561,-4225,5442,-9006,-429,160,-9234,-4444,3586,-5711,-9506,-79,-4418,-4348,-5891]), 93), -594.58065);
}
```

#### 결과
![leetcode-14-submission-1](/assets/img/for-post/Maximum Average Subarray 1/img.png)
_Leetcode 제출 결과_

## References

[Maximum Average Subarray 1 - LeetCode](https://leetcode.com/problems/maximum-average-subarray-i/description/?envType=study-plan-v2&envId=leetcode-75)
