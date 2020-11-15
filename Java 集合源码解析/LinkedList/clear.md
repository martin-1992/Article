### clear
　　清空链表，遍历每个节点，将节点的 item、next、prev 设为空，将 frist 和 last 设为空，容量为 0。

```java
    public void clear() {
        for (Node<E> x = first; x != null; ) {
            Node<E> next = x.next;
            x.item = null;
            x.next = null;
            x.prev = null;
            x = next;
        }
        first = last = null;
        size = 0;
        modCount++;
    }
```