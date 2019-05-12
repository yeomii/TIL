# Diagonal Traverse

[문제 링크](https://leetcode.com/problems/diagonal-traverse/)

---

## 문제 설명
2차원 행렬이 주어질 때 이미지처럼 대각선 순서로 순회하기

## 문제 풀이
* 문제에서 나온 우상향 대각선의 경우 선마다 `row index + column index` 가 동일하다.
* `row index + column index` 의 최대 합은 `m-1 + n-1` 이기 때문에 이 합이 작은 순서대로 반복문을 돌리면 된다.
* 특정 대각선에서 row index 는 [0, m), column index 는 [0, n) 사이에 존재하므로 이 정보를 가지고 아래 코드의 `i` 의 범위를 정해줄 수 있다.

```python3
def findDiagonalOrder(self, matrix: List[List[int]]) -> List[int]:
        
    m = len(matrix)
    if m == 0:
        return []
    
    n = len(matrix[0])
    if n == 0:
        return []
    
    res = []
    
    for s in range(m + n - 1):
        if s % 2 == 0:
            for i in range(max(0, s - m + 1), min(n, s + 1)):
                res.append(matrix[s-i][i])
        else:
            for i in range(max(0, s - n + 1), min(m, s + 1)):
                res.append(matrix[i][s-i])
    return res
```

* Time Complexity : O(n)
* Space Complexity : O(1)