---
title: LeetCode 75-45 1466. Reorder Routes to Make All Paths Lead to the City Zero
author: KanghoonYi
name: KanghoonYi(pour)
date: 2024-11-13 15:31:00 +0900
categories: [Programming, python, leetcode75]
tags: [programming, python, leetcode]
pin: false
math: true
---

## 문제 요약
도시 사이의 connection 정보가 input으로 주어집니다.  
n개의 도시에 대한 connection(edge정보)이 주어지는데, 이는 방향성(directional)이 있습니다.  
이중 수도인 capital(0번 도시)로 향하지 않는 방향성을 capital로 향하도록 수정하려고 합니다.  
최종 return값은 수정해야 하는 edge의 갯수입니다.

![leetcode-45-example-img-1](/assets/img/for-post/leetcode75-45/img.png)
_Example 1_

## 문제 풀이

### 첫번째 시도

#### Complexity
**시간복잡도(time complexity)** 는 $O(n)$입니다. 여기서 n은 connection의 길이, 즉 city의 갯수입니다. DFS의 복잡도는 edge와 node의 갯수와 관련이 있지만, 이 문제에서는 모두 city의 갯수와 일치하기 때문에, $O(n)$으로 표기하였습니다.   
**공간복잡도(space complexity)** 는 $O(n)$입니다. connections의 길이에 따라 별도의 `Set[(int, int)]`과 `Dict[int, list[int]]`를 생성합니다.  

> 복잡도를 계산하기 매우 어렵게 느껴서, 부정확합니다.
{: .prompt-info }

1. `edge_set`: 먼저 connections를 추후에 비교연산을 효율적으로 수행하기 위해, `Set[(출발 도시, 도착 도시)]` 형태로 변형시킵니다.
2. `neighbor_nodes`: 추후 connections을 iteration하며, 이웃한 도시를 저장할때 사용합니다.
3. `visit_history`: 이미 방문한 도시를 기록하여, 중복연상을 방지합니다.
4. 이제 0번 도시를 기준으로 Tree라고 생각합니다. Tree는 root에서 leaf로 뻗어나가면서 연산이 진행되기 때문에, 비교할때 반대방향으로 하는것만 주의합니다. dfs로 `neighbor_nodes`를 순회하며,
> 문제의 요구사항이 모든 edge가 0번도시(Tree의 Root node)를 향해야 한다고 했기 때문에, edge방향성 비교를 할때 반대로 하도록 주의해야합니다.
{: .prompt-info }
5. DFS를 하며 edge가 기대(expect, 0번도시를 향하지 않는 경우)와 다른 경우를 찾습니다.
6. 찾은 count를 return합니다.


```python
from typing import List
class Solution:
    def minReorder(self, n: int, connections: List[List[int]]) -> int:
        edge_set: set[(int, int)] = set((connection[0], connection[1]) for connection in connections)
        neighbor_nodes: dict[int, list[int]] = { city: [] for city in range(n) }
        visit_history = set()
        count_changes = 0

        for connection in connections:
            neighbor_nodes[connection[0]].append(connection[1])
            neighbor_nodes[connection[1]].append(connection[0])

        def dfs(neighbor_nodes: dict[int, list[int]], index: int) -> None:
            nonlocal edge_set, count_changes, visit_history
            visit_history.add(index)
            neighbors = neighbor_nodes[index]
            for neighbor_index in neighbors:
                if neighbor_index in visit_history:
                    continue
                if (neighbor_index, index) not in edge_set:
                    count_changes += 1
                dfs(neighbor_nodes, neighbor_index)

            return

        dfs(neighbor_nodes, 0)

        return count_changes

solution = Solution()

assert solution.minReorder(6, [[0,1],[1,3],[2,3],[4,0],[4,5]]) == 3
assert solution.minReorder(5, [[1,0],[1,2],[3,2],[3,4]]) == 2
```

#### 결과
![leetcode-45-submission-1](/assets/img/for-post/leetcode75-45/python_submission_1.png)
_Leetcode 제출 결과_


## 후기
혼자서 풀때는 풀지 못했고, Leetcode Study모임에서 풀어오신분에게 영감을 받아 풀 수 있었습니다.  
확실히 Tree와 Graph로 넘어오면서 한층 더 난이도가 높게 느껴집니다.

## References

[1466. Reorder Routes to Make All Paths Lead to the City Zero - LeetCode](https://leetcode.com/problems/reorder-routes-to-make-all-paths-lead-to-the-city-zero?envType=study-plan-v2&envId=leetcode-75)
