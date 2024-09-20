---
title: LeetCode 75-8 String Compression
author: KanghoonYi(pour)
name: KanghoonYi(pour)
date: 2024-09-20 18:05:00 +0900
categories: [Programming, rust, leetcode75]
tags: [programming, rust, leetcode]
pin: false
math: true
---

## 문제 요약

주어진 `chars(char array)`를 압축(compression)합니다.  
이 `chars`는 여러 char의 group(반복되는)으로 이루어져 있으며, 이 반복되는 `char`를 압축하는 문제입니다.  
이때,   
- If the group's length is 1, append the character to s.
  - group의 길이가 1이면, 그냥 char 's' 하나만 붙입니다(숫자 없이. 1을 붙이지 않는다는 의미).
- Otherwise, append the character followed by the group's length.
  - 그렇지 않으면, char 's' 뒤에 반복되는 숫자를 붙입니다.
- You must write an algorithm that uses only constant extra space
  - 반드시 상수항(constant) 만큼의 공간(space)를 추가로 사용해야 합니다.
- Note that group lengths that are 10 or longer will be split into multiple characters in chars
  - 반복되는 횟수가 10회 이상인경우, 이를 chars에 반영할때, 각 자리수를 별개의 char로 만들어서 넣어야 합니다.
## 문제 풀이

### 첫번째 시도
#### 순회 방향의 문제
처음에는 `chars`를 1회 '정방향' 순회하면서 문제를 풀어낼 수 있다고 생각했습니다.  
이때의 문제는, 'extra space'에 대한 제약사항 때문에 입력받은 chars를 직접 수정해야하는데, '정방향'으로 순회하며 수정하면, array가 계속 변경되어, 탐색이 어려워지는 문제가 있습니다.  
그래서 생각한게, **'역방향'** 으로 순회하면 array의 뒷 부분만 변경되기 때문에, index를 활용한 순회가 좀 더 쉬워진다고 생각했습니다.

#### chars를 변경하는 시점의 문제
저는 char의 repeat을 count하고, 이를 chars에 반영하는 '시점'을 어떻게 해야할지 고민했습니다.  
문제의 요구사항을 참고하여, 역방향 탐색을 하면서, 바로 왼쪽(index가 더 작은쪽)의 char가 현재의 것과 다르면, 지금까지 세고 있던 count를 chars array에 반영하도록 하였습니다.  

#### 반복횟수가 10회 이상인 경우의 문제
아래 code의 `convert_repeat_count_to_chars()`가 이에 대해 작성한 solution입니다.  
반복을 count한 값이 10이상이면, 각 1의 자리부터 각 자릿수를 분리하여, 집어 넣는 방식입니다.

#### Complexity
시간복잡도는 


```rust
fn compress(chars: &mut Vec<char>) -> i32 {
    if chars.len() <= 1 {
        return 1;
    }

    let mut repeat_char: char = char::default();
    let mut repeat_count: u32 = 0;
    let mut idx: i32 = chars.len() as i32 - 1;

    while idx >= 0 {
        let char = chars[idx as usize];

        let before = if idx > 0 { chars[idx as usize - 1] } else { char::default() };

        if repeat_char == char {
            repeat_count += 1;
        } else {
            repeat_char = char;
            repeat_count = 1;
        }

        if before != char {
            let splice_start_idx = idx as usize;
            chars.splice(splice_start_idx..std::cmp::min(splice_start_idx + repeat_count as usize, chars.len()), convert_repeat_count_to_chars(repeat_char, repeat_count));
        }

        idx -= 1;
    }


    return chars.len() as i32;
}

fn convert_repeat_count_to_chars(repeat_char: char, count: u32) -> Vec<char> {
    let mut mut_count = count;
    let mut chars = vec![repeat_char];

    if count <= 1 {
        return chars;
    }

    while (mut_count > 0) {
        chars.insert(1, std::char::from_digit(mut_count % 10, 10).unwrap());
        mut_count = mut_count / 10;
    }
    return chars;
}

fn main() {
    assert_eq!(compress(&mut Vec::from(['a', 'a','b', 'b', 'c', 'c', 'c'])), 6);
    assert_eq!(compress(&mut Vec::from(['a'])), 1);
    assert_eq!(compress(&mut Vec::from(['a','b','b','b','b','b','b','b','b','b','b','b','b'])), 4)
}
```

#### 결과
![leetcode-8-submission-1](/assets/img/for-post/String Compression/img.png)

## References

[String Compression - LeetCode](https://leetcode.com/problems/string-compression/description/?envType=study-plan-v2&envId=leetcode-75)
