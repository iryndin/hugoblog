+++
Tags = ["data structures","binary search tree"]
date = "2018-06-17T02:55:26Z"
title = "Binary search tree"
+++

Binary search tree (BST) is a data structure that allows fast element lookup, addition or removal of items.

<!--more-->

It is a binary tree that has following properties:

* the left subtree of a node contains only nodes with keys lesser than the node’s key
* the right subtree of a node contains only nodes with keys greater than the node’s key
* the left and right subtree each must also be a binary search tree. There must be no duplicate nodes.

<img src="/img/binary_search_tree.png" style="width:100%" />

## Node class

Let BST node be described with following class: 

```java
public class Node {
  public int key;
  public int val;
  public Node left;
  public Node right;

  public Node(int key, int val) {
    this.key = key;
    this.val = val;
    this.left = this.right = null;
  }
}
```

## Search 

Searching for a key in a BST can be done either recursively or iteratively. 
Complexity is, of course, `O(log(n))` on average. 
But for worst case (when BST degrades to linked list) comlexity can degrade to `O(n)`.

### Recursive search

```java
Node search(Node root, int key) {
  if (root == null || root.key == key) {
  	return root;
  }
  if (key < root.key) return search(root.left, key);
  else return search(root.right, key);
}
```

### Iterative search

```java
Node search(Node root, int key) {
  Node current = root;
  while (current != null) {
    if (current.key == key) return current;
    if (key < current.key) current = current.left;
    else current = current.right;
  }
  return current;
}
```

## Insert new key

Inserting new key is similar to search. We always start from root, and then traverse down to leaf nodes. 
So, that new values are always inserted as leaf nodes. 
This means that BST structure depends on the order in which new keys are inserted. 

### Recursive insert

```java
Node insert(Node root, int key, int val) {
  if (root == null) {
    // when tree is empty, create new root
    return new Node(key, val);
  } else if (key == root.key) {
    root.val = val;
  } else if (key < root.key) {
    root.left = insert(root.left, key, val);
  } else {
    root.right = insert(root.right, key, val);
  }
  return root;
}
```

### Iterative insert

When implementing iterative insert we should also track the parent of the current node.
We need this parent to link one of its children (left or right) to a newly created leaf node. 

```java
Node insert(Node root, int key, int val) {
  Node parent = null;
  Node current = root;
  while (true) {
    if (current == null) {
      current = new Node(key, val);
      if (parent != null) {
        if (key < parent.key) parent.left = current;
        else parent.right = current;
      }
      break;
  	}
    if (key == current.key) {
      current.val = val;
      break;
    }
    parent = current;
    if (key < current.key) {
      current = current.left;
    } else {
      current = current.right;
    }
  }
  return current;
}

```

## Delete key

When removing node from BST, it is mandatory to maintain in-order sequence of nodes. 
Let's consider following cases when deleting a node:

* it is leaf node, i.e. it does not have children. In this case simply remove the node
* node has only one child. In this case simply remove this node, and put a child in its place.
* node has left and right children

So far let's write code that process deletion of a given key from BST for first two cases 
(i.e. when node either does not have children or has only one child):

```java
// remove key and return new root of the BST
Node delete(Node root, int key) {
  if (root == null) {
    return null;
  }
  if (key < root.key) {
    root.left = delete(root.left, key);
  } else if (key > root.key) {
    root.right = delete(root.right, key);
  } else {
    // here root.key == key
    if (root.right == null) {
      return root.left;
    } else if (root.left == null) {
      return root.right;
    } else {
      // Complex case: node has 2 children
      // Discuss it further. 
      deleteNodeWith2Children(root, key);
    }
  }
  return root;
}
```

### Delete using in-order successors approach

Let's consider case when node has 2 non-null children: `left` and `right`. 
We find in-order successor (node with the next smallest key) of the current node, 
and put its value in-place of the current node. Then we simply delete this successor node. 
So, the code is as following: 

```java
void deleteNodeWith2Children(Node root, int key) {
  assert root.key == key;
  assert root.left != null;
  assert root.right != null;

  Node successor = findSuccessor(root.right);
  root.key = successor.key;
  root.val = successor.val;
  root.right = delete(root.right, successor.key);
}

// Find most minimal node, 
// so go left as far as possible
Node findSuccessor(Node root) {
  Node cur = root;
  while (cur.left != null) {
    cur = cur.left; 
  }
  return cur;
}
```

### Delete using in-order predecessor approach

Here approach is the same as with in-order successor. 
We find in-order predecessor of current node, and put its key and value in place of current node. 
Then we simply delete the predecessor node. 

```java
void deleteNodeWith2Children(Node root, int key) {
  assert root.key == key;
  assert root.left != null;
  assert root.right != null;

  Node predecessor = findPredecessor(root.left);
  root.key = predecessor.key;
  root.val = predecessor.val;
  root.left = delete(root.left, predecessor.key);
}

// Find node with max key,
// so go right as far as possible
Node findPredecessor(Node root) {
  Node cur = root;
  while (cur.right != null) {
    cur = cur.right; 
  }
  return cur;
}
```

### Notes on using either in-order successor or predecessor

Consistently using the in-order successor or the in-order predecessor for every instance of the two-child case can lead to an unbalanced tree, so some implementations select one or the other at different times.

## Tree traversals

Next depth-first-search traversals exist: 

* in-order traversal (Left, Root, Right)
* pre-order traversal (Root, Left, Right)
* post-order traversal (Left, Right, Root)

### In-order traversal

In-order traversal: we visit nodes in the following order: Left -> Root -> Right. 

```java
void traverseInOrder(Node root) {
  if (root != null) {
    traverseInOrder(root.left);
    System.out.println(root.key);
    traverseInOrder(root.right);
  }
}
```

### Pre-order traversal

```java
void traversePreOrder(Node root) {
  if (root != null) {
    System.out.println(root.key);
    traverseInOrder(root.left);    
    traverseInOrder(root.right);
  }
}
```

### Post-order traversal

```java
void traversePreOrder(Node root) {
  if (root != null) {    
    traverseInOrder(root.left);    
    traverseInOrder(root.right);
    System.out.println(root.key);
  }
}
```

## Verify that Binary Search Tree is correct

Here is a small task: verify that given BST is correct.  Lower given an example with incorrect BST: 

```
     20
    /  \
  10    30
       /  \
      5    40
```

Node with key `5` should be greater than `20`, hence it is not a valid BST. 

So we should track not only value of parent, but minimum and maximum values as well. 
We should check that node key is between max and min value.   
If we go down left branch, we should update `maxKey`, 
because left branch in BST should contain nodes with smaller keys.  
If we go right branch, we should update `minKey`, because right branch contains 

```java
boolean isCorrect(Node root) {
  if (root == null) return true;
  return isCorrect(root, Integer.MIN_VALUE, INTEGER.MAX_VALUE);
}

boolean isCorrect(Node root, int minKey, int maxKey) {
  if (root == null) return true;
  if (root.key <= minKey || root.key >= maxKey) return false;
  return 
    isCorrect(root.left, minKey, root.key) &&
    isCorrect(root.right, root.key, maxKey);
}
```

## Find max and min elements in BST

Minimal element in the BST is the leftmost element:

```java
Node findMin(Node root) {
  if (root == null) return null;
  Node cur = root;
  while (cur.left != null) {
  	cur = cur.left;
  }
  return cur;
}
```

The largest lement in the BST in rightmost element.

```java
Node findMax(Node root) {
  if (root == null) return null;
  Node cur = root;
  while (cur.right != null) {
  	cur = cur.right;
  }
  return cur;
}
```

## Links

* [Wikipedia article on Binary Search Tree](https://en.wikipedia.org/wiki/Binary_search_tree)
* [GeeksForGeeks - Binary Search Tree | Set 1 (Search and Insertion)](https://www.geeksforgeeks.org/binary-search-tree-set-1-search-and-insertion/)
* [GeeksForGeeks - Tree Traversals (Inorder, Preorder and Postorder)](https://www.geeksforgeeks.org/tree-traversals-inorder-preorder-and-postorder/)
* [GeeksForGeeks - All articles on BST](https://www.geeksforgeeks.org/binary-search-tree-data-structure/)
* [GeeksForGeeks - All articles tagged with BST tag](https://www.geeksforgeeks.org/category/binary-search-tree/)
* [Leetcode - Problems for BST](https://leetcode.com/tag/binary-search-tree/)
* [Leetcode - Problems for Tree](https://leetcode.com/tag/tree/)
