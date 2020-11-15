
### put
　　将 key 和 value 添加到 HashMap 中。

```java
    public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
    }
```

### putVal
　　将 key 和 value 添加到 HashMap 中。

- 数组为空，则先进行扩容，因为要插入值；
- 没有找到该 key 的哈希值，即在数组中没有该索引 (n - 1) & hash，则将该值包装成节点，插入数组中。插入新节点成功，返回 null；
- 数值中存在该 key，有三种情况：
    1. 该 key 和要插入的 key 是相同的，则覆盖掉旧的 key；
    2. 哈希冲突，插入红黑树；
    3. 哈希冲突，将该值构造成节点，插入链表末尾。如果链表长度 binCount 大于 8，则转为红黑树。

```java
    // 链表长度为 8，转为红黑树
    static final int TREEIFY_THRESHOLD = 8;

    final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        // 数组为空，则进行扩容
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
        // 没有找到该 key 的哈希值，即在数组中没有该索引 (n - 1) & hash，则将该值包装成节点，插入数组中；
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        else {
            Node<K,V> e; K k;
            // 相同的 key，则覆盖掉，下面逻辑 e.value = value 会覆盖掉旧的 value
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            else if (p instanceof TreeNode)
                // 哈希冲突，插入红黑树
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {
                // 哈希冲突，插入链表末尾
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        // 将该值保证成节点，插入链表末尾
                        p.next = newNode(hash, key, value, null);
                        // 如果链表长度 binCount 大于 8，则转为红黑树
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    // 在链表中找到该 key，打破循环，表示链表中存在该 key，返回该节点 e
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    // 继续遍历，当前节点为前节点，然后循环调用 p.next
                    p = e;
                }
            }
            // e 初始化为空，这里不为空，表示该 key 存在 map 中
            // 设置该 key 的新值，返回旧值
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!only45IfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        // 修改次数加一
        ++modCount;
        // 容量超过阈值，则进行扩容
        if (++size > threshold)
            resize();
        // 回调函数
        afterNodeInsertion(evict);
        // 插入新节点成功，返回 null
        return null;
    }
```
