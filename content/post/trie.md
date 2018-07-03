+++
date = "2018-06-28T03:20:39Z"
title = "Trie or Prefix Tree"
Tags = ["data structures","trie", "prefix tree"]
+++

Let's consider another popular data structure: Trie or Prefix tree. 

<!--more-->

Trie (or prefix tree) is an ordered tree data structure, mainly used for operations with strings (but now always). If we consider strings, then for trie following points are true:

* Nodes can have multiple children 
* Key is not stored in the node, rather it is defined by node position in the tree
* Each node stores single character
* Prefix is defined as a sequence of characters by traversing trie from root down to particular node. Hence, all children of a node have the common prefix. 

Codewise, node of a trie can be defines as this (please note that there is no `key` in the node!): 

```java
class TrieNode<T> {
  T value;
  Map<Character, TrieNode> children;
}
```

Or another common representation is to have array (with size equal to alphabet length) where non-null array members are children of this node. 

```java
class TrieNode {
  T value;
  TrieNode[] children = new TrieNode[26];

  /**
   * Get child for character {@code c}
   **/
  public TrieNode getChild(char c) {
    int idx = (int)'a' - (int)c;
    return children[idx];
  }
}
```

Example of storing dictionary (`sam, sand, sir, sick, sunday`) in trie:

<img src="/img/trie.png" style="width:50%" />

## Usage of trie

Trie can be used for following tasks:

* dictionary representation
* autocomplete feature in text editors
* searching for common prefix
* it is used in some algorithms: Aho-Corasick, Lempel-Ziv-Welch compression algorithm

## Operations on the trie

Let's deal with first implementation of `TrieNode`, 
i.e. where chidren are put into map: `Map<Character, TrieNode`. 

### Look for a key

```java
<T> T find(String key, TrieNode<T> root) {
  TrieNode<T> cur = root;
  for (char c : key.toCharArray()) {
    TrieNode<T> n = cur.children == null ? null : cur.children.get(c);
    if (n != null) {
      cur = n;
    } else {
      return null;
    }
  }
  return cur.value;
}
```

### Insert key

```java
// Return previous value associated with key, 
// or null if no previous value was associated
<T> T insert(String key, T newValue, TrieNode<T> root) {
  TrieNode<T> cur = root;
  for (int i=0; i<key.length(); i++) {
    char c = key.charAt(i);
    TrieNode<T> n = cur.children == null ? null : cur.children.get(c);
    if (n != null) {
      cur = n;
    } else {
      if (cur.children == null) {
        cur.children = new HashMap<>();
      }
      TrieNode<T> newNode = new TrieNode<>();
      cur.children.put(c, newNode);
      cur = newNode;
    }
  }
  T oldValue = cur.value;
  cur.value = value;
  return oldValue;
}
```

### Remove key

```java
<T> T delete(String key, TrieNode<T> root) {
  TrieNode<T> cur = root;
  for (char c : key.toCharArray()) {
    TrieNode<T> n = cur.children == null ? null : cur.children.get(c);
    if (n != null) {
      cur = n;
    } else {
      return null;
    }
  }  
  T oldValue = cur.value;
  cur.value = null;
  return oldValue;
}
```

## Links

* [Leetcode - Problems for tries](https://leetcode.com/tag/trie/)
* [GeekForGeeks - Trie articles](https://www.geeksforgeeks.org/tag/trie/)
* [GeekForGeeks - Trie practice](https://practice.geeksforgeeks.org/topics/Trie)
* [Basecs - Trying to understand tries](https://medium.com/basecs/trying-to-understand-tries-3ec6bede0014)
* [Hackerearth - Trie](https://www.hackerearth.com/practice/data-structures/advanced-data-structures/trie-keyword-tree/tutorial/)
* [Baeldung - The Trie Data Structure in Java](http://www.baeldung.com/trie-java)


