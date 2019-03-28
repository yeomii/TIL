# 117. Populating Next Right Pointers in Each Node II

[문제 링크](https://leetcode.com/problems/populating-next-right-pointers-in-each-node-ii/)

---

## 문제 설명

* 이진트리에서 각 노드에 next 라는 포인터가 해당 노드 기준으로 오른쪽 형제 노드중 가장 가까이 있는 노드를 가리키도록 하자

## 문제 풀이

* preorder 로 순회할 때, 조상 노드들이 next 포인터가 알맞게 채워져 있다고 가정하면
* 방문중인 노드의 next 포인터는 
    * 부모 노드의 오른쪽 자식노드 (방문중인 노드 제외) 
    * 혹은 오른쪽 형제 노드의 자식 노드들 중 가장 왼쪽에 위치한 노드이다.

```python3
class Solution:
    def findNextLeftMostChild(self, root: 'Node'):
        if root is None:
            return None
        while root.next is not None:
            if root.next.left is not None:
                return root.next.left
            elif root.next.right is not None:
                return root.next.right
            else:
                root = root.next
        return None
            
    # solution
    def connect(self, root: 'Node') -> 'Node':
        if root is None:
            return None
        
        # 자식노드가 둘 다 있을 경우 왼쪽 자식 노드의 next 노드는 오른쪽 자식노드
        if root.left is not None and root.right is not None:
            root.left.next = root.right

        # 자식노드 중 가장 오른쪽에 있는 노드의 next 를 결정
        rightMost = root.right if root.right is not None else root.left
        if rightMost is not None:
            rightMost.next = self.findNextLeftMostChild(root)
        
        # preorder 순회
        self.connect(root.right)
        self.connect(root.left)
        
        return root
```
