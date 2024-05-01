The approach is to calculate the height of the left subtree and the right subtree at each node, and calculate the diameter by adding the heights.

Every time diameter is calculated, compare it with the previous value and store the max value as the diameter.

finally, return the diameter value.

## Solving it using recursion

**Base condition** return 0 at the leaf nodes, since the height there is 0
**Recursive condition**: at every node, calculate height of left and right subtree, and return max height (left or right) + 1.

**Additional logic**: remember to update the diameter value to keep storing the maximum value.

The solution:
```python
class Solution:
	def diameterOfBinaryTree(self, root):
		self.diameter = 0
		self.depth(root)
		return self.diameter

	def depth(self, node):
		if node is None:
			return 0
		left_height = self.depth(node.left)
		right_height = self.depth(node.right)
		self.diameter = max(self.diameter, left_height + right_height)
		return max(left_height, right_height) + 1
```

This approach is using an attribute to keep track of the overall maximum value of the diameter, this can be replaced with a mutable data type like a list or dictionary.

Approach using mutable data types:
```python
class Solution:
	def diameterOfBinaryTree(self, root):
		self.depth(root, maxDiameter=[0])
		return maxDiameter[0]
		
	def depth(self, node, maxDiameter):
		if node is None:
			return 0
		left_height = self.depth(node.left, maxDiameter)
		right_height = self.depth(node.right, maxDiameter)
		maxDiameter[0] = max(maxDiameter[0], left_height + right_height)
		return max(left_height, right_height) + 1
```

[[Drawing 2024-04-21 14.54.05.excalidraw]]


