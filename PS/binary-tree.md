# Binary Tree

* [leet code - introduction](https://leetcode.com/explore/learn/card/data-structure-tree/)

## Intro

`tree` 는 계층적인 트리 구조를 나타내기 위해 자주 사용되는 자료구조이다.

`Binary Tree` 는 트리 구조의 한 형태로, 각 노드 당 자식 노드를 최대 2개 까지만 가질 수 있다.


## Traversing Tree (트리 순회하기)
- Pre-order
  - root 먼저 방문 > 왼쪽 subtree 방문 > 오른쪽 subtree 방문
- In-order
  - 왼쪽 subtree 방문 > root 방문 > 오른쪽 subtree 방문
  - binary search tree 에서 in-order 로 순회하면 sorting 된 결과를 얻을 수 있다
- Post-order
  - 왼쪽 subtree 방문 > 오른쪽 subtree 방문 > root 방문
  - 노드를 삭제하는 경우 post-order 를 사용
- Recursive or Iterative
  - 위 세 가지 순서로 순회하는 것을 recursive 또는 iterative 로 구현할 수 있다
- Level-order
  - 같은 레벨에 있는 노드들을 전부 방문한 후 그 다음 레벨에 있는 노드들을 순서대로 방문하는 방법
  - Breadth-First Search 로 쉽게 구현 가능하다