### get
　　获取为读操作，不需要加锁。注意，在 [Node]() 中 val 和 next 指针都使用 volatile 修饰，保证获取的是最新的值。

- 通过 hash 来获取，找到，则直接返回；
- 找到节点，则遍历链表，查找并返回。

```java
    public V get(Object key) {
        Node<K,V>[] tab; Node<K,V> e, p; int n, eh; K ek;
        // 获取 hash 值
        int h = spread(key.hashCode());
        if ((tab = table) != null && (n = tab.length) > 0 &&
            (e = tabAt(tab, (n - 1) & h)) != null) {
            if ((eh = e.hash) == h) {
                // 哈希直接找到值，则返回
                if ((ek = e.key) == key || (ek != null && key.equals(ek)))
                    return e.val;
            }
            else if (eh < 0)
                return (p = e.find(h, key)) != null ? p.val : null;
            while ((e = e.next) != null) {
                // 遍历链表，查找节点值，并返回
                if (e.hash == h &&
                    ((ek = e.key) == key || (ek != null && key.equals(ek))))
                    return e.val;
            }
        }
        return null;
    }
```

### spread
　　计算哈希值，同 HashMap 一样，会将高位右移 16 位，再异或一遍。

```java
    static final int HASH_BITS = 0x7fffffff;

    static final int spread(int h) {
        return (h ^ (h >>> 16)) & HASH_BITS;
    }
```

### containsKey
　　是否包含该 key。

```java
public boolean containsKey(Object key) {
        return get(key) != null;
    }
```