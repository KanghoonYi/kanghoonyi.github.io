---
title: LeetCode 75-11 Is Subsequence
author: KanghoonYi(pour)
name: KanghoonYi(pour)
date: 2024-09-22 14:48:00 +0900
categories: [Programming, rust, leetcode75]
tags: [programming, rust, leetcode]
pin: false
math: true
---

## 문제 요약
function의 parameter인 s와 t는 string을 받습니다.  
s는 t의 subsequence인지 확인해야 합니다.  
이 문제에서의 'subsequence'의 의미는 다음과 같습니다.  
- a new string that is formed from the original string by deleting some (can be none) of the characters without disturbing the relative positions of the remaining characters.
    - subsequence는 원문(original)에서 특정 문자를 삭제하면 만들어질 수 있는걸 말합니다. 즉 순서가 지켜져야 합니다. 
- 예를 들면, "ace" is a subsequence of "abcde" while "aec" is not

## 문제 풀이

### 첫번째 시도

#### Complexity
**시간복잡도(time complexity)** 는 $O(n)$입니다.  
**공간복잡도(space complexity)** 는 $O(1)$입니다.

s와 t를 각각 바라보는 cursor를 총 2개 운영하였으며, 조건을 만족하면, cursor의 위치가 바뀝니다.  
이때, t를 순회하는 동안, s를 모두 순회했는지 확인하여, 문제가 요구하는 결과값을 bool로 return합니다.

```rust
fn is_subsequence(s: String, t: String) -> bool {
    let mut s_cursor: usize = 0;
    let mut t_cursor: usize = 0;

    while s_cursor < s.len() && t_cursor < t.len() {
        let s_char = &s[s_cursor..s_cursor+1];
        let t_char = &t[t_cursor..t_cursor+1];
        if s_char == t_char {
            s_cursor += 1;

        }
        t_cursor += 1;
    }

    if s_cursor >= s.len() {
        return true;
    }

    return false;
}

fn main() {
    assert_eq!(is_subsequence(String::from("abc"), String::from("ahbgdc")), true);
    assert_eq!(is_subsequence(String::from("axc"), String::from("ahbgdc")), false);
    assert_eq!(is_subsequence(String::from("aaaaaa"), String::from("bbaaaa")), false);
}
```

#### 결과
![leetcode-11-submission-1](/assets/img/for-post/Is Subsequence/img.png)


## References

[Is Subsequence - LeetCode](https://leetcode.com/problems/is-subsequence/description/?envType=study-plan-v2&envId=leetcode-75)
