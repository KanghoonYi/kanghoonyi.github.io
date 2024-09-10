---
title: LeetCode 75-2 Greatest Common Divisor of Strings
author: KanghoonYi(pour)
name: KanghoonYi(pour)
date: 2024-09-09 18:05:00 +0900
categories: [Programming, rust, leetcode75]
tags: [programming, rust, leetcode]
pin: false
math: true
---

## 문제 요약

두 문자열, str1과 str2에서 각각 반복되는 최대 길이의 문자열중 공통된 부분(’x’로 표기)을 찾습니다.

## 문제 풀이

### 첫번째 시도

처음에는 문제를 이해하지 못하고(최대공약수 문제인지 모르고), 풀려고 했습니다.  
여러 접근법이 머리에 떠오릅니다.

- str1과 str2를 byte code로 바꾸어서, 각각 ‘-’연산을 하면 0이 아닌 지점을 통해 뭔가를 파악할 수 있지 않을까?

> ‘easy’난이도 임에도, 한참 헤맸는데 문제를 잘 이해하는게 얼마나 중요한지 깨달았습니다.
{: .prompt-tip }

```rust
fn gcd_of_strings_v1(str1: String, str2: String) -> String {
    let mut long_len: usize = std::cmp::max(str1.len(), str2.len());

    let mut diff_cursor: usize = 0;

    for i in 1..=long_len {
        if i > str1.len() || i > str2.len() {
            diff_cursor = i - 1;
            break;
        }
        let sliced_str1 = &str1[0..i];
        let sliced_str2 = &str2[0..i];

        if sliced_str1 != sliced_str2 {
            diff_cursor = i;
            break;
        }
    }

    let x = (&str1[0..diff_cursor]).to_string();
    if x.len() == 1 {
        return String::from("");
    }
    if x.len() % 2 != 0 {

        return x;
    }

    let mut i: usize= x.len() / 2;
    while i > 0 {
        let part = &x[0..i];
        let compare_part: &str = &x[i..i+part.len()];
        if part == compare_part {
            return part.to_string();
        }
        i = (i / 2) as usize;
    }

    return x;
}

fn main() {
    assert_eq!(gcd_of_strings_v1("ABCABC".to_string(), "ABC".to_string()), "ABC");
    assert_eq!(gcd_of_strings_v1("ABABAB".to_string(), "ABAB".to_string()), "AB");
    assert_eq!(gcd_of_strings_v1("LEET".to_string(), "CODE".to_string()), "");
}
```

### 두번째 시도

문제의 이름을 보니 ‘Great Common Divisor’가 보입니다. ‘최대공약수’를 의미하는듯 합니다.  
이를 문제에 활용할 수 있는 방법을 찾아봅니다.

```rust
fn gcd_of_strings_v1(str1: String, str2: String) -> String {
    let common_divisors = get_common_divisors(str1.len(), str2.len());
    let mut gcd: usize = 0;
    for &divisor  in common_divisors.iter().rev() {
        let str1_part = &str1[0..divisor as usize];
        let str2_part = &str2[0..divisor as usize];

        if str1_part != str2_part {
            continue;
        }

        let str1_repeat = String::from(str1_part).repeat(str1.len() / (divisor as usize));
        if str1_repeat != str1 {
            continue;
        }

        let str2_repeat = String::from(str2_part).repeat(str2.len() / (divisor as usize));
        if str2_repeat != str2 {
            continue;
        }

        gcd = divisor as usize;
        break;
    }

    if gcd == 0 {
        return "".to_string();
    }

    return String::from(&str1[0..gcd]);
}

fn get_common_divisors(num_1: usize, num_2: usize) -> Vec<u32> {
    let num1_divisors = get_divisors_of_num(num_1);
    let num2_divisors = get_divisors_of_num(num_2);

    let mut num1_idx: usize = 0;
    let mut num2_idx: usize = 0;

    let mut common_divisors: Vec<u32> = Vec::new();

    while (num1_idx < num1_divisors.len() && num2_idx < num2_divisors.len()) {
        let num1_candidate = num1_divisors[num1_idx];
        let num2_candidate = num2_divisors[num2_idx];

        if num1_candidate == num2_candidate {
            common_divisors.push(num1_candidate);
            num1_idx = std::cmp::min(num1_idx + 1, num1_divisors.len());
            num2_idx = std::cmp::min(num2_idx + 1, num2_divisors.len());
            continue;
        }

        if num1_candidate < num2_candidate {
            num1_idx = std::cmp::min(num1_idx + 1, num1_divisors.len());
        } else if num1_candidate > num2_candidate {
            num2_idx = std::cmp::min(num2_idx + 1, num2_divisors.len());
        }

    }
    return common_divisors;
}

fn get_divisors_of_num(target_num: usize) -> Vec<u32> {
    let mut divisors: Vec<u32> = Vec::new();
    for i in 1..=target_num {
        if target_num % i == 0 {
            divisors.push(i as u32);
        }
    }

    return divisors;
}

fn main() {
    assert_eq!(gcd_of_strings_v1("ABCABC".to_string(), "ABC".to_string()), "ABC");
    assert_eq!(gcd_of_strings_v1("ABABAB".to_string(), "ABAB".to_string()), "AB");
    assert_eq!(gcd_of_strings_v1("LEET".to_string(), "CODE".to_string()), "");
    assert_eq!(gcd_of_strings_v1("ABABABAB".to_string(), "ABAB".to_string()), "ABAB");
}
```

Flow는 다음과 같습니다.

1. `get_common_divisors` 함수(여기선 method)를 통해, 각 String의 길이(length)에 대한 ‘공약수 array’를 구합니다.

   ‘공약수(common divisor)’를 구하는 이유는,
    : 우리가 찾아낼 String ‘x’는 str1과 str2에서 모두 반복되고 있습니다. 때문에, **문자열(String) x의 길이(length)는 str1과 str2모두에서 반복이 가능한 길이**여야 합니다. 이 문자열 x의 길이로서 가능성 있는것이 ‘공약수(common divisor)’입니다.
    : 이 ‘공약수(common divisor)’를 기준으로 String의 slice를 만들어, str1과 str2에서 반복이 발생하는 최대 값을 찾으면, 문자열 ‘x’를 구할 수 있습니다.

2. 이 ‘공약수 array’를 큰 값부터 탐색합니다.(array를 reverse탐색합니다.)

    ```rust
    for &divisor in common_divisors.iter().rev() {
          ...
      }
    ```

3. 이 공약수(common divisor)를 기준으로 str1과 str2를 slice하고, 실제로 각각의 str1과 str2에서 반복이 발생하는지 확인합니다.  
   여기선, String slice를 원문 String의 길이에 맞게 반복하고, 일치하는지 확인합니다.

    ```rust
    fn gcd_of_strings_v1(str1: String, str2: String) -> String {
        let common_divisors = get_common_divisors(str1.len(), str2.len());
        let mut gcd: usize = 0;
        for &divisor  in common_divisors.iter().rev() {
            ...
    
            let str1_repeat = String::from(str1_part).repeat(str1.len() / (divisor as usize));
            if str1_repeat != str1 {
                continue;
            }
    
            let str2_repeat = String::from(str2_part).repeat(str2.len() / (divisor as usize));
            if str2_repeat != str2 {
                continue;
            }
            
            ...
        }
        ...
    }
    ```

4. 위의 모든 조건을 만족하는 divisor를 찾아내면, ‘**Greatest Common Divisor’**를 찾아낸 겁니다.

   > 공약수(common divisor)중 큰것 부터 탐색했기 때문에, 가장 먼저 발견되는 값이 ‘최대공약수’입니다.
   {: .prompt-tip }

### 기타 함수 설명

#### Function get_common_divisors()

두개의 정수를 받아서, $O(n)$의 비용으로 ‘common divisor’ list를 반환합니다.

```rust
fn get_common_divisors(num_1: usize, num_2: usize) -> Vec<u32> {
    let num1_divisors = get_divisors_of_num(num_1);
    let num2_divisors = get_divisors_of_num(num_2);

    let mut num1_idx: usize = 0;
    let mut num2_idx: usize = 0;

    let mut common_divisors: Vec<u32> = Vec::new();

    while (num1_idx < num1_divisors.len() && num2_idx < num2_divisors.len()) {
        let num1_candidate = num1_divisors[num1_idx];
        let num2_candidate = num2_divisors[num2_idx];

        if num1_candidate == num2_candidate {
            common_divisors.push(num1_candidate);
            num1_idx = std::cmp::min(num1_idx + 1, num1_divisors.len());
            num2_idx = std::cmp::min(num2_idx + 1, num2_divisors.len());
            continue;
        }

		    // num1_divisors와 num2_divisors는 정렬되어 있으므로, 아래와 같이 작은쪽의 index만 증가시킵니다.
        if num1_candidate < num2_candidate {
            num1_idx = std::cmp::min(num1_idx + 1, num1_divisors.len());
        } else if num1_candidate > num2_candidate {
            num2_idx = std::cmp::min(num2_idx + 1, num2_divisors.len());
        }

    }
    return common_divisors;
}

```

## References

[Greatest Common Divisor of Strings - LeetCode](https://leetcode.com/problems/greatest-common-divisor-of-strings/?envType=study-plan-v2&envId=leetcode-75)
