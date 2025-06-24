---
title: Big-O notation and complexity(big-O 표기법과 복잡도) Examples
author: KanghoonYi
name: KanghoonYi(pour)
date: 2024-09-09 21:01:16 +0900
categories: [Books, Cracking The Coding Interview]
tags: [Computer Science, algorithm, 알고리즘, Time Complexity]
pin: false
math: true
---

## Overview

몇가지 예시를 통해 big-O표기법에 대해서 이해합니다.

## 예시

### 여러개의 문자열을 각각 정렬하고 전체 list에 대해서도 정렬하는 경우

총 N개의 문자열이 있다고 하자, 이때, 각각의 문자열을 알파벳 순으로 정렬하고, 전체 문자열 목록을 사전(Dictionary)순에 따라 정리하는 예시입니다.

이 경우,
: 문자열 1개를 정렬하는데, $O(N\log{N})$의 비용이 들기때문에, 전체 문자열 N에 대해서 수행해야 하므로 총$O(N*N\log{N})$.
: 이후 전체 문자열을 사전순으로 정렬해야 하므로, 추가로 $O(N\log{N})$.
: 총 $O(N^2\log{N} + N\log{N})\rightarrow O(N\log{N})$ 이 된다고 생각할 수 있습니다.

하지만 **이건 틀렸습니다**.

- ‘문자열 길이와 문자열 list의 길이는 다르다’는 걸 고려해야 합니다.

이를 수정하면,
: 문자열중 가장 긴 경우의 길이를 ‘s’라고 하고, array의 길이를 ‘a’라고 했을때,
: 1개의 문자열을 정렬하는데, $O(s\log{s})$가 든다. a개의 문자열을 정렬해야 하므로, $O(a*s\log{s})$가 된다.
: 이제, 전체 문자열 array를 정렬하기 위한 비용을 다루는데, 쉽게 $O(a\log{a})$라고만 생각할 수 있다. 하지만, ‘사전순’으로 정렬해야 하기 때문에, 2개의 문자열을 비교하는 비용인 $O(s)$를 고려해야 한다. 이에 따라, 문자열 array를 정렬하는 비용은 $O(a*s\log{a})$이다.

이를 모두 정리하면,  
$$
O(a*s(\log{a}+\log{s}))
$$
이다.

> 2개의 문자열을 비교하는 비용($O(s)$)을 잊지말자.
{: .prompt-tip }

### 재귀호출(recursive call)의 패턴분석

다음은 이진탐색트리(Binary Search Tree, 이하 BST)의 모든 Node의 합을 구하는 코드입니다.  
여기서, 시간복잡도(Time complexity)는 어떻게 될까요?  

```java
int sum(Node node) {
	if (node == null) {
		return 0;
	}
	return sum(node.left) + node.value + sum(node.right);
}
```

BST라는 이유로, $\log{n}$으로 시작하는 무언가를 떠올릴 수 있지만, 이 경우,

$$
O(N)
$$

가 맞습니다.    
  
이를 2가지 방향으로 설명할 수 있습니다.

#### **코드가 실제로 의미하는바가 무엇인가?**
간단한 방법으로 코드의 실제 의미를 생각해보는 방법입니다. 이 코드는 각 노드를 방문한 후, 상수시간($O(1)$)의 작업을 수행합니다. 이에 따라, 각 Node를 방문하는 비용이 지배적이며, 결국 Node의 갯수와 연관이 있습니다.  
즉, Node의 갯수 N과 선형관계인 $O(N)$이 시간복잡도(Time complexity)가 됩니다.

#### **재귀호출의 패턴을 분석하자**
지난 post에서 ‘Recursive call’의 runtime을 $O(\text{분기 갯수}^{깊이})$로 표현했습니다.  
각 Node의  
BST의 경우, 그 의미에 따라서 ‘분기 갯수 = 2’가 되고, $O(2^{깊이})$으로 정리됩니다.  
여기서, BST의 깊이는 $\log_2{N}$이 므로, $O(2^{\log_2{N}})$이 됩니다.  
$\log_2{}$의 의미를 다시 기억하면, 다음과 같습니다. 

$$
2^{P}=Q \rightarrow \log_2{Q}=P
$$

이에 따라, $O(2^{\log_2{N}})$을 정리하면,

$$
P=2^{\log_2{N}} \rightarrow \log_2{P}=\log_2{N} \rightarrow P=N
$$

즉,

$$
O(N)
$$

이 됩니다.

## Reference

**Book**
: [CRACKING the CODING INTERVIEW](https://www.crackingthecodinginterview.com/)
