### put

- spread，获取 key 的 hash 值；
- 根据 key 的 hash 值，计算出索引下标。使用 tabAt，为 volatile，保证获取是最新的；
- 找到该索引下标，进行插入；
    1. 判断为链表。如果为空，则将其包装成新节点，使用 CAS 来插入。使用循环，不插入成功不会退出；
    2. 判断为链表。不为空，则使用节点锁，synchronized 锁住该节点。找到旧值，则进行替代。没有找到旧值，则将其包装成新节点，添加到末尾；
    3. 判断为红黑树，则使用节点，synchronized 锁住该节点，进行插入。

```java
    public V put(K key, V value) {
        return putVal(key, value, false);
    }

    /** Implementation for put and putIfAbsent */
    final V putVal(K key, V value, boolean onlyIfAbsent) {
        // 不允许插入 key 为 null 的键，而 HashMap 允许
        if (key == null || value == null) throw new NullPointerException();
        // 计算 hash 值
        int hash = spread(key.hashCode());
        // 修改次数
        int binCount = 0;
        // 使用到 CAS 插入新节点，存在失败，所以使用循环，不断重试，直到插入成功
        for (Node<K,V>[] tab = table;;) {
            // 局部变量，属于虚拟机栈，线程安全
            Node<K,V> f; int n, i, fh;
            // 列表为空，进行初始化
            if (tab == null || (n = tab.length) == 0)
                tab = initTable();
            else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
                // 该 key 为空，使用 CAS 来插入新节点
                if (casTabAt(tab, i, null,
                             new Node<K,V>(hash, key, value, null)))
                    break;                   // no lock when adding to empty bin
            }
            // 正在扩容，帮助扩容
            else if ((fh = f.hash) == MOVED)
                tab = helpTransfer(tab, f);
            else {
                // 存在旧值，则使用细粒度锁，只锁住当前节点，因为只替换当前节点的值
                V oldVal = null;
                synchronized (f) {
                    if (tabAt(tab, i) == f) {
                        if (fh >= 0) {
                            binCount = 1;
                            // 遍历链表节点
                            for (Node<K,V> e = f;; ++binCount) {
                                K ek;
                                // 该节点存在，则更新值
                                if (e.hash == hash &&
                                    ((ek = e.key) == key ||
                                     (ek != null && key.equals(ek)))) {
                                    oldVal = e.val;
                                    if (!onlyIfAbsent)
                                        e.val = value;
                                    break;
                                }
                                // 该节点不存在，则添加到链表末尾
                                Node<K,V> pred = e;
                                if ((e = e.next) == null) {
                                    pred.next = new Node<K,V>(hash, key,
                                                              value, null);
                                    break;
                                }
                            }
                        }
                        else if (f instanceof TreeBin) {
                            // 红黑树，节点添加
                            Node<K,V> p;
                            binCount = 2;
                            if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                           value)) != null) {
                                oldVal = p.val;
                                if (!onlyIfAbsent)
                                    p.val = value;
                            }
                        }
                    }
                }
                if (binCount != 0) {
                    // 同 HashMap 一样，链表长度大于等于 8，转为红黑树
                    if (binCount >= TREEIFY_THRESHOLD)
                        treeifyBin(tab, i);
                    if (oldVal != null)
                        return oldVal;
                    break;
                }
            }
        }
        // size + 1
        addCount(1L, binCount);
        return null;
    }
```

### casTabAt
　　使用 CAS 来保存值。

```java
    static final <K,V> boolean casTabAt(Node<K,V>[] tab, int i,
                                        Node<K,V> c, Node<K,V> v) {
        return U.compareAndSwapObject(tab, ((long)i << ASHIFT) + ABASE, c, v);
    }
```