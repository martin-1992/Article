### Node
　　和 HashMap 几乎一样，构造节点。val 和 next 指针加了 volatile，包装其可见性，用于多线程情况下。

```java
    static class Node<K,V> implements Map.Entry<K,V> {
        // 哈希值
        final int hash;
        final K key;
        volatile V val;
        // 指向下个节点的指针
        volatile Node<K,V> next;

        Node(int hash, K key, V val, Node<K,V> next) {
            this.hash = hash;
            this.key = key;
            this.val = val;
            this.next = next;
        }

        public final K getKey()       { return key; }
        public final V getValue()     { return val; }

        /**
         * 使用 key 的 hashCode 和 value 的 hashCode 进行异或，获得该节点的 hashCode
         */
        public final int hashCode()   { return key.hashCode() ^ val.hashCode(); }
        public final String toString(){ return key + "=" + val; }
        public final V setValue(V value) {
            throw new UnsupportedOperationException();
        }

        /**
         * equals 和 hashCode 两个重写其中一个，另外一个也需要重写，不重写可能会出现两个对象
         * 相同，但 hashCode 不同
         */ 
        public final boolean equals(Object o) {
            Object k, v, u; Map.Entry<?,?> e;
            return ((o instanceof Map.Entry) &&
                    (k = (e = (Map.Entry<?,?>)o).getKey()) != null &&
                    (v = e.getValue()) != null &&
                    (k == key || k.equals(key)) &&
                    (v == (u = val) || v.equals(u)));
        }

        /**
         * Virtualized support for map.get(); overridden in subclasses.
         * 遍历查找
         */
        Node<K,V> find(int h, Object k) {
            Node<K,V> e = this;
            if (k != null) {
                do {
                    K ek;
                    if (e.hash == h &&
                        ((ek = e.key) == k || (ek != null && k.equals(ek))))
                        return e;
                } while ((e = e.next) != null);
            }
            return null;
        }
    }
```





