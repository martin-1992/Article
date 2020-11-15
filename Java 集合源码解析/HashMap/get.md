
### get
　　先对 key 进行哈希，在调用 getNode 从数组中获取该节点的值。

```java
    public V get(Object key) {
        Node<K,V> e;
        return (e = getNode(hash(key), key)) == null ? null : e.value;
    }
```

### containsKey
　　HashMap 中是否包含该 key，同样是获取该 key 的哈希值，然后遍历链表或红黑树找到该 key，找不到则为 false。

```java
    public boolean containsKey(Object key) {
        return getNode(hash(key), key) != null;
    }
```

### getNode
　　实现 Map.get 的相关方法，HashMap 为追求性能，会使用位运算，速度更快。

- 先判断节点数组是否不为空，使用 (n - 1) & hash 找到该 key 在数组中对应的索引位置；
- 遍历该索引位置的链表，先检查第一个节点，判断是否为要找到节点，如果这里没有重写 equals，可能导致相等的 key，却判断为不相等，导致出错；
- 如果不是第一个节点，则会判断下个节点，如果下个节点为 TreeNode，则调用红黑树的方法来判断（当链表长度大于 8，会转为红黑树），否则调用链表的方法来判断，继续遍历，直到没找到返回 null。

```java
    transient Node<K,V>[] table;

    final Node<K,V> getNode(int hash, Object key) {
        Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
        // tab 为包含节点对象的数组，
        if ((tab = table) != null && (n = tab.length) > 0 &&
           // (n - 1) & hash 相当于 hash % tab.length，即求余数，因为 tab 为 2 的次方，所以高位为 1，其余为 0，
           // 比如 1000，然后减一为 0111，在与 hash 相与 &，则高位为 0，就是余数，之所以用位运算，是速度更
           // 快。余数即为数组的索引，接下来就是遍历该位置的链表，找到与该 key 相等的节点
            (first = tab[(n - 1) & hash]) != null) {
            // 检查该索引位置的第一个节点，判断是否为要找到节点，如果这里没有重写 equals，
            // 可能导致相等的 key，却判断为不相等，导致出错
            if (first.hash == hash && // always check first node
                ((k = first.key) == key || (key != null && key.equals(k))))
                return first;
            if ((e = first.next) != null) {
                // 下个节点是否为红黑树，调用红黑树判断
                if (first instanceof TreeNode)
                    return ((TreeNode<K,V>)first).getTreeNode(hash, key);
                do {
                    // 调用链表获取
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        return e;
                } while ((e = e.next) != null);
            }
        }
        return null;
    }
```
