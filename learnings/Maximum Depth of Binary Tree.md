# Mutable default arguments

## Mutable types:
These types are changed after they are created.
Eg: lists, dictionaries, sets and custom Objects

## Immutable types:
These types cannot be changed after they are created.
Eg: integers, strings, floats and tuples

> Default values are created and evaluated only once, during the function definition.

If a mutable type is used as default argument for a function, it's value will be persisted between function calls which might cause bugs.

The program should look like this.

### Base Condition
Return level when node becomes None.

### Recursive condition
Return the max level from the left and right sub-tree at any node.

```Python
def maxDepth(self, root, level):
	if (root is None):
		return level
	return max(self.maxDepth(root.left, level + 1), self.maxDepth(root.right, level + 1))

def result(self, root):
	return self.maxDepth(root, 0)
```

[[Drawing 2024-04-20 19.57.39.excalidraw]]