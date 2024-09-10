---
title: LeetCode 75-3 Kids With the Greatest Number of Candies
author: KanghoonYi(pour)
name: KanghoonYi(pour)
date: 2024-09-10 15:53:00 +0900
categories: [Programming, rust, leetcode75]
tags: [programming, rust, leetcode]
pin: false
math: true
---

## 문제 요약
`candies`라는 array에는 candy의 갯수(integer type)가 들어있습니다.   
`candies[i]`는 *i*th째 아이(kid)가 가지고 있는 candy의 갯수입니다.  
`extraCandies`는 여러분이 가지고 있는 여분의 candy를 의미하며,  
이를 각각의 아이에게'만' 주었을 때, 각각의 아이가 가장 많은 수(Greatest number)의 candy를 갖게 되는지 일종의 시뮬레이션 결과를 반환하는 문제입니다.

## 문제 풀이
간단한 문제라 작성한 solution은 1개 입니다.

'example1'로 예를 들어 설명하면,
: Input: candies = [2,3,5,1,3], extraCandies = 3
: Output: [true,true,true,false,true]

Input의 값을 고려하여 아래의 조건을 충족해야 합니다.
$$
\text{candies}+3\geqq5 \rightarrow candies \geqq 2
$$
즉, candies array에서, 각 요소(element)가 $candies \geqq 2$를 만족하는지 확인하면 됩니다.

```rust
impl Solution {
    pub fn kids_with_candies(candies: Vec<i32>, extra_candies: i32) -> Vec<bool> {
        // let max_candy = candies.iter().max().unwrap();
        let threshold: i32 = candies.iter().max().unwrap() - extra_candies;

        let results: Vec<bool> = candies.iter().map(|&candy_count| {
            return candy_count >= threshold;
        }).collect();

        return results;
    }
}
```

## 기타 의견
$candies \geqq 2$를 확인할 때에, CPU의 [SIMD](https://ko.wikipedia.org/wiki/SIMD)를 활용하여 병렬처리할 수 있을듯 하다.  
rust에선 [std::simd](https://doc.rust-lang.org/std/simd/index.html)모듈로 'experimental API'로서 제공하고 있다.

## References

[Kids With the Greatest Number of Candies - LeetCode](https://leetcode.com/problems/kids-with-the-greatest-number-of-candies/description/?envType=study-plan-v2&envId=leetcode-75)
