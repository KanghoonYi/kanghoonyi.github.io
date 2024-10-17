---
title: Hash Table, Hash Map, Hash Set
author: KanghoonYi(pour)
name: KanghoonYi(pour)
date: 2024-10-17 16:13:00 +0900
categories: [Programming, Computer Basics]
tags: [Computer Science, programming, Data Structure, algorithm]
pin: false
math: true
---

## Overview

Hash를 이용한 자료구조는 Time Complexity를 줄이는데 아주 중요한 역할을 합니다.  
여기서는, Hash를 이용한 Data Structure를 소개합니다.  
또한, Hash의 Time Complexity는 $O(1)$(insert, delete, search)로 알려져 있는데, 어떻게 이게 가능한지 알아보려 합니다.  

> 단순하게 생각하면, key의 hash값이 일치하는지 확인하는것도 $O(n)$이 될것 같은데, $O(1)$로 소개되는, 관련된 내용도 다룹니다.
{: .prompt-info }

## Hash와 관련된 이유
Hash를 이용한 Data structure은 table, map, set등 다양합니다.  
이들의 성능은 모두 ‘**hash function**’과 깊게 연관되어 있습니다.  
> A huge amount of the performance of a hash table depends on the quality of your hash functions.
>

어떤 ‘hash function’을 쓰느냐에 따라, 세부적인 성능이 달라 지지만, 그 내부에 있는 idea는 같습니다.

### Key값을 Hash처리하는 이유.

Key값으로 보통 String데이터가 주어지는데, 이를 그대로 사용하지 않고, hash function을 사용하는 이유가 뭘까요?  
이는 **높은 성능을 내기위한 목적과, 일관된 성능을 제공하기 위함**입니다.  

**높은 성능?**
: 만약, key로 주어진 String을 그대로 이용한다면, String을 구성하는 각 character를 모두 비교해야 합니다.
: 이는 100% 일치하는지 확인하는 방법이긴 하지만, 문자열 길이에 비례하는 $O(n)$의 Time Complexity를 갖게 됩니다.
: Hash Data structure에 사용되는 Hash function들은 String을 받으면, Integer값을 return합니다. 이 값은, HashMap이 갖고 있는 **데이터의 위치를 나타내는 index**가 됩니다.
: 이는 key값을 비교할때, ‘Integer 비교(comparison)’가 가능하게 하며, 더 나은 ‘비교 성능($O(1)$)’과 더 적은 메모리를 사용하게 합니다.

> 대표적인 Hash함수로 ‘[siphash](https://github.com/veorq/SipHash/blob/master/siphash.c)’가 있습니다. 현재 rust와 cpp에서 hashmap을 구성할때 사용하고 있습니다.
{: .prompt-info }

**일관된 성능?**
: Hash함수는 데이터 분배에 영향을 미칩니다. 다량의 데이터가 Hash table에 들어가야 한다면, hash결과에 따라, 데이터가 **균일하게** bucket(key값에 대응하는 데이터들이 보관되는 곳)에 배치되도록 합니다.
: 이는 검색, 삽입, 삭제 속도를 일정하게 유지(key에 따른 성능뿐만 아니라 전체적인 성능도)해주는 역할을 합니다.
  
Hash를 이용한 자료구조에서, Hash함수는 다음을 **반드시** 만족해야 합니다.
```rust
k1 == k2 -> hash(k1) == hash(k2)
```

## Hash Collision 문제

Hash는 input값을 encoding하기 때문에, 서로 다른 값인데, 같은 hash값을 가질 수 있습니다.

> Hash의 결과값은 범위가 한정되어 있는 반면에, 입력값은 훨씬 다양하기 때문에 발생하게 됩니다.
>

이를 수식으로 표현하면 다음과 같습니다.

```rust
k1 != k2 -> hash(k1) == hash(k2)
```

Collision이 발생하는 경우,  
: ‘Chaining’이라는 방법으로 중복된 hash값 끼리 묶어서 저장(같은 bucket에 저장).
: ‘Open Addressing’이라는 방법으로 빈 bucket에 데이터를 저장.
이런 방법이 사용됩니다.

이 Collision의 발생을 줄이려면,
: 값의 범위가 더 큰 hash function을 사용하거나,
: 값의 분포가 더 균등하게 발생하는 hash function을 사용허간,
: Hashmap의 사이즈를 결정하는 요소들을 수정
합니다.

## Hash를 이용한 Data Structure

### HashTable 혹은 HashMap

![Java hashmap from [https://oshyshkov.com/2021/07/07/java-hashmap-implementation/](https://oshyshkov.com/2021/07/07/java-hashmap-implementation/)](/assets/img/for-post/Hash%20Table%2C%20Hash%20Map%2C%20Hash%20Set/image.png)
_Java hashmap from [https://oshyshkov.com/2021/07/07/java-hashmap-implementation/](https://oshyshkov.com/2021/07/07/java-hashmap-implementation/)_

HashMap은 주어진 key값을 Hash function을 통해 integer key로 변환합니다.  
이 integer key를 기반으로 탐색, 삽입을 수행합니다.  

> HashTable과 HashMap은 구현 세부사항에 약간의 차이는 있지만, Data Structure관점에선 동일하게 취급합니다.
{: .prompt-info }
 
> Java에서는 [HashTable](https://docs.oracle.com/javase/8/docs/api/index.html?java/util/Hashtable.html)과 [HashMap](https://docs.oracle.com/javase/8/docs/api/?java/util/HashMap.html)이 별도로 있습니다.  
> HashTable은 Synchronous(동기)로서 일관된 데이터를 보장하지만 성능은 떨어집니다(병렬 처리 어려움).  
> HashMap은 Ansynchronous(비동기)로서 더 나은 성능을 발휘하지만, 데이터 일관성은 떨어집니다. null값을 저장할 수 있습니다.
{: .prompt-info }

> Rust에서 HashMap은 SIMD lookup을 지원합니다.  
> A [hash map](https://doc.rust-lang.org/std/collections/index.html#use-a-hashmap-when) implemented with quadratic probing and SIMD lookup.
{: .prompt-info }

### HashSet

데이터의 중복을 제거하는데 많이 사용하는 hashSet입니다.  
hash값을 기준으로, unique한 데이터를 저장합니다.  
Java와 rust에선, Hashmap을 기반으로 구현되어 있습니다(value가 공백인 형식, key값만 이용.).  

> A hash set implemented as a HashMap where the value is ().
- from [rust doc](https://doc.rust-lang.org/std/collections/struct.HashSet.html)
>

## 다양한 Hash Function과 Pros & Cons

Hash를 사용한 Data Structure에서, Hash함수를 바꿈으로서 성능을 개선할 수 있습니다.

**SipHash 1-3**
: ‘medium sized key’에서 좋은 성능을 내며, ‘HashDos’공격에 대한 대비가 되어 있는 function입니다.
: 기본적으로 host에서 생성된 random seed를 사용하여 보안성을 갖춥니다.
: 암호화를 갖춘 hash function들 중에서는 빠른 특징이 있습니다.(murmur보다는 느립니다)

> While its performance is very competitive for medium sized keys, other hashing algorithms will outperform it for small keys such as integers as well as large keys such as long strings, though those algorithms will typically not protect against attacks such as HashDoS.
>

**MurmurHash**
: 낮은 ‘collision’을 자랑하고, 데이터 분포를 고르게 만들 수 있습니다(다량의 데이터에 대한 hashmap성능을 최적화 할때 좋음).
: key값이 커도, 빠르게 hash처리할 수 있습니다(성능이 좋습니다).
: seed값을 사용하지 않아, 보안에 취약한 모습을 보입니다.(공격자가 의도적으로 collision을 발생시킬 수 있음)

**SHA-256**
: SHA-256은 보안성을 갖춘 해시 함수로, 주로 암호화와 관련된 환경에서 많이 사용됩니다. 보안이 중요한 해시 작업에 적합합니다.
: 계산 방법이 매우 복잡하지만, 충돌가능성이 매우 낮습니다.
: 하지만, 성능이 느려서 HashMap에 사용하기에는 적합하지 않을 수 있습니다.

**MD5**
: 이전에는 많이 사용했지만, 충돌이 발생하는 경우를 쉽게 찾을 수 있어(빈번히 발생할 수 있어) 사용되지 않습니다.

## Hash관련 취약점

### [HashDoS Attack](https://en.wikipedia.org/wiki/Collision_attack#Hash_flooding)

공격자가 의도적으로 Hash Collision을 발생시키는 공격입니다.  
의도적으로 Hash Collision을 발생시켜, 특정 Bucket에 데이터가 몰리게하고, 이 데이터를 선형탐색(linear probe)하도록 하여 컴퓨터 자원을 차지하게 만듭니다.  
이에 대응하여(해결하기 위하여), [SipHash](https://en.wikipedia.org/wiki/SipHash)가 등장했습니다.  

## Conclusion

Hash Data Structure은 Hash Function과 깊게 관련되어 있으며, 그 목적과 용도에 맞게 내부 hash function을 변경하여 사용할 수 있습니다.  
$O(1)$이라는 Time Complexity가 가능한것도, Hash Function이 내놓는 결과가 Integer이기 때문에, 가능한 결과입니다.  
다만, Collision이 많이 발생하는 Hashmap인 경우 성능이 감소할 수 있지만, 데이터 분포가 잘 되는 Hash function을 사용한다면 문제를 최소화 할 수 있습니다.

## References

HashMap의 구현체중 하나인 SwissTable에 대한 소개 영상
: [CppCon 2017: Matt Kulukundis “Designing a Fast, Efficient, Cache-friendly Hash Table, Step by Step”](https://www.youtube.com/watch?v=ncHmEUmJZf4)

Rust의 hashmap
: [HashMap in std::collections - Rust](https://doc.rust-lang.org/std/collections/struct.HashMap.html)

HashTable은 프로그래머의 기본기라고 소개하는 영상
: [Hash Table은 프로그래머의 기본기](https://youtu.be/S7vni1hdsZE?si=Wk46PzLhLcUcNHf1)

Java hashmap Implementation
: [Java HashMap implementation](https://oshyshkov.com/2021/07/07/java-hashmap-implementation/)

Basics of Hash Tables
: [Basics of Hash Tables  Tutorials & Notes](https://www.hackerearth.com/practice/data-structures/hash-tables/basics-of-hash-tables/tutorial/)

Hash collision을 이용한 HashDos 공격
: [Collision attack](https://en.wikipedia.org/wiki/Collision_attack#Hash_flooding)

Collisions for Hash Functions MD4, MD5, HAVAL-128 and RIPEMD
: [Collisions for Hash Functions MD4, MD5, HAVAL-128 and RIPEMD](https://eprint.iacr.org/2004/199)

SwissTable
: [abseil / Swiss Tables and <code>absl::Hash</code>](https://abseil.io/blog/20180927-swisstables)
