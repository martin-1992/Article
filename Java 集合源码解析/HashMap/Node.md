
### Node
　　新建节点。

```java
    Node<K,V> newNode(int hash, K key, V value, Node<K,V> next) {
        return new Node<>(hash, key, value, next);
    }
```


### Node
　　链表节点。

```java
    static class Node<K,V> implements Map.Entry<K,V> {
        // 哈希值
        final int hash;
        final K key;
        V value;
        // 指向下个节点的指针
        Node<K,V> next;
        
        // 节点构造函数
        Node(int hash, K key, V value, Node<K,V> next) {
            this.hash = hash;
            this.key = key;
            this.value = value;
            this.next = next;
        }

        public final K getKey()        { return key; }
        public final V getValue()      { return value; }
        public final String toString() { return key + "=" + value; }
        
        // 使用 key 的 hashCode 和 value 的 hashCode 进行异或，获得该节点的 hashCode
        public final int hashCode() {
            return Objects.hashCode(key) ^ Objects.hashCode(value);
        }
        
        public final V setValue(V newValue) {
            V oldValue = value;
            value = newValue;
            return oldValue;
        }
        
        // equals 和 hashCode 两个重写其中一个，另外一个也需要重写，不重写可能会出现两个对象
        // 相同，但 hashCode 不同
        public final boolean equals(Object o) {
            if (o == this)
                return true;
            if (o instanceof Map.Entry) {
                Map.Entry<?,?> e = (Map.Entry<?,?>)o;
                if (Objects.equals(key, e.getKey()) &&
                    Objects.equals(value, e.getValue()))
                    return true;
            }
            return false;
        }
    }
```


### Objects#hashCode
　　对象的 hashCode 方法。

```java
    public static int hashCode(Object o) {
        return o != null ? o.hashCode() : 0;
    }
```

