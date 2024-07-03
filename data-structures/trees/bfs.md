# Implementing BFS in Trees

## Approach:

Use a Que data structure to implement the BFS approach.

Things to take care of:

- Level: This can be done by iterating in the queue from 0 to it's length, so new children are not considered in the same level.

> [!IMPORTANT]
> Length of a que for every iteration contains all the nodes at a single level.
> Unlike DFS, where we use Stack (recursion) to implement it, we use a queue here.

**Algorith** :

```python
class Solution:
  def bfs(self, root):

    # getting the initial configuration
    level_order_traversal = []
    queue = collections.deque()
    queue.append(root) #insert a node from right

    # iterate till the queue is not empty
    while queue:
      items_in_level = [] # an array to store items at every level
      level_length = len(queue)
      for i in range(level_length):
        item = queue.popleft() # take an item out from the left
        if item:
          items_in_level.append(item)
          # insert the left and right children in any order
          q.append(item.left)
          q.append(item.right)

      if items_in_level:
        level_order_traversal.append(items_in_level)
    return level_order_traversal
```
