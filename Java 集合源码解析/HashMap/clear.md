
### clear
　　清空 Map 中的 key，遍历数组 Map，将每个节点赋值为空。

```java
    public void clear() {
        Node<K,V>[] tab;
        modCount++;
        if ((tab = table) != null && size > 0) {
            size = 0;
            for (int i = 0; i < tab.length; ++i)
                tab[i] = null;
        }
    }
```
