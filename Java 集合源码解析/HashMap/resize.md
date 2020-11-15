
### resize
　　两倍扩容，遍历旧数组的每个节点对其进行重新哈希，填充的索引位置有两种：与原先索引位置一致，新索引位置即原先索引位置加上旧容量的大小。

- 参数校验，旧容量大于最大容量，没法扩容了，返回旧数组。而如果旧容量乘以 2 是小于最大容量，则新容量是旧容量的两倍；
- 当旧容量是小于等于 0 时，而旧阈值是大于 0，则直接将新容量设置为旧阈值；
- 旧容量小于等于 0，旧阈值也是，则设置新容量为默认值；
- 旧阈值为 0，表示负载因子 DEFAULT_LOAD_FACTOR 为 0，它是可修改的，使用不可修改的默认负载因子 loadFactor 来获得阈值，并判断是否溢出；
- 旧数组不为空，遍历旧数组的每个节点。
    1. 将旧数组的节点保存为 e，同时将旧数组对应的索引位置赋值为 null，用于 GC 回收；
    2. 只有一个节点的情况下，则直接哈希对新数组的容量求余数，获得新数组的索引位置；
    3. 多个节点情况下，需要根据 e.hash & oldCap 来判断高位是否为 1，是则新索引为 [旧索引位置 + 旧容量 oldCap]，不是则还是和旧索引位置一致。比如 00101 & 01111 和 00101 & 011111 是相等的，即使扩容两倍后，相与 & 的索引值不变。而 10101 & 01111 和 10101 & 011111 则是不等的，扩容两倍后，新索引的值是旧索引位置 + 旧容量 oldCap。

```java
    final Node<K,V>[] resize() {
        // 旧数组
        Node<K,V>[] oldTab = table;
        // 旧容量
        int oldCap = (oldTab == null) ? 0 : oldTab.length;
        // 旧阈值
        int oldThr = threshold;
        // 参数校验，旧容量大于最大容量，没法扩容了，返回旧数组
        int newCap, newThr = 0;
        if (oldCap > 0) {
            if (oldCap >= MAXIMUM_CAPACITY) {
                threshold = Integer.MAX_VALUE;
                return oldTab;
            }
            // 旧容量乘以 2 是小于最大容量，则新容量是旧容量的两倍
            else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                     oldCap >= DEFAULT_INITIAL_CAPACITY)
                newThr = oldThr << 1; // double threshold
        }
        else if (oldThr > 0) 
            // 当旧容量是小于等于 0 时，而旧阈值是大于 0，则直接将新容量设置为旧阈值
            newCap = oldThr;
        else {               
            // 旧容量小于等于 0，旧阈值也是，则设置新容量为默认值
            newCap = DEFAULT_INITIAL_CAPACITY;
            newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
        }
        if (newThr == 0) {
            // 旧阈值为 0，表示负载因子 DEFAULT_LOAD_FACTOR 为 0，它是可修改的，使用不可修改的默
            // 认负载因子 loadFactor 来获得阈值，并判断是否溢出
            float ft = (float)newCap * loadFactor;
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                      (int)ft : Integer.MAX_VALUE);
        }
        threshold = newThr;
        @SuppressWarnings({"rawtypes","unchecked"})
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
        table = newTab;
        if (oldTab != null) {
             // 旧数组不为空，遍历旧数组的每个节点
            for (int j = 0; j < oldCap; ++j) {
                Node<K,V> e;
                // 将旧数组的节点保存为 e，同时将旧数组对应的索引位置赋值为 null，用于 GC 回收
                if ((e = oldTab[j]) != null) {
                    oldTab[j] = null;
                    if (e.next == null)
                        // 只有一个节点的情况下，则直接哈希对新数组的容量求余数，获得新数组的索引位置
                        newTab[e.hash & (newCap - 1)] = e;
                    else if (e instanceof TreeNode)
                        // 该索引位置为红黑树
                        ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                    else { 
                        // 
                        Node<K,V> loHead = null, loTail = null;
                        Node<K,V> hiHead = null, hiTail = null;
                        Node<K,V> next;
                        do {
                            // 多个节点情况下，遍历该节点的链表
                            next = e.next;
                            // 如果该节点的 hash 值和旧容量 oldCap 是否为 0，来判断该节点的 hash 值高位是否为 1，
                            // 举例 e.hash 为 10101（21），旧的索引为 e.hash & (oldCap - 1)，oldCap = 10000，
                            // oldCap-1=01111，高位被抹去，于是旧的索引为 e.hash & (oldCap - 1)=10101 & 01111。
                            // 这里使用 e.hash & oldCap = 10101 & 10000，高位没有被抹去。如果高位为 1，则哈希到索
                            // 引是 newTab[j + oldCap] = hiHead，即原先索引值加上旧容量值。如果是 e.hash 为 00101，
                            // 高位本身为 0，e.hash & (oldCap-1) 等价于 e.hash & (newCap - 1)，所以 newTab[j] =
                            // loHead，即新的索引值还是跟旧索引值一样
                            if ((e.hash & oldCap) == 0) {
                                if (loTail == null)
                                    loHead = e;
                                else
                                    loTail.next = e;
                                loTail = e;
                            }
                            else {
                                if (hiTail == null)
                                    hiHead = e;
                                else
                                    hiTail.next = e;
                                hiTail = e;
                            }
                        } while ((e = next) != null);
                        if (loTail != null) {
                            loTail.next = null;
                            newTab[j] = loHead;
                        }
                        if (hiTail != null) {
                            hiTail.next = null;
                            newTab[j + oldCap] = hiHead;
                        }
                    }
                }
            }
        }
        return newTab;
    }
```
