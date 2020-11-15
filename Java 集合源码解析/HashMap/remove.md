
### remove
　　从 HashMap 中移除某个 key 及对应的值。

```java
    public V remove(Object key) {
        Node<K,V> e;
        return (e = removeNode(hash(key), key, null, false, true)) == null ?
            null : e.value;
    }
```

### removeNode

- 从 Map 中找到该 Key 对应的节点，没找到返回 null；
- 找到，则首先判断是否为首节点，不是则遍历红黑树或遍历链表；
- 找到该节点后，则进行删除。先判断该节点是否属于红黑树，然后再判断是否为首节点，最后用链表撒删除方法。

```java
    final Node<K,V> removeNode(int hash, Object key, Object value,
                               boolean matchValue, boolean movable) {
        Node<K,V>[] tab; Node<K,V> p; int n, index;
        // 在 Map 中找到该 key 对应的节点
        if ((tab = table) != null && (n = tab.length) > 0 &&
            (p = tab[index = (n - 1) & hash]) != null) {
            Node<K,V> node = null, e; K k; V v;
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                // 首节点
                node = p;
            else if ((e = p.next) != null) {
                if (p instanceof TreeNode)
                    // 遍历红黑树
                    node = ((TreeNode<K,V>)p).getTreeNode(hash, key);
                else {
                    do {
                        // 遍历链表
                        if (e.hash == hash &&
                            ((k = e.key) == key ||
                             (key != null && key.equals(k)))) {
                            node = e;
                            break;
                        }
                        p = e;
                    } while ((e = e.next) != null);
                }
            }
            // 删除该 key
            if (node != null && (!matchValue || (v = node.value) == value ||
                                 (value != null && value.equals(v)))) {
                if (node instanceof TreeNode)
                    ((TreeNode<K,V>)node).removeTreeNode(this, tab, movable);
                else if (node == p)
                    // 要删除的节点为链表的首节点，则直接指向下个节点
                    tab[index] = node.next;
                else
                    // 链表是指向要要删除节点的下个节点
                    p.next = node.next;
                ++modCount;
                --size;
                afterNodeRemoval(node);
                return node;
            }
        }
        // 不在 Map 中，返回空
        return null;
    }
```
