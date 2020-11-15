### poll
　　弹出头节点。

```java
    public E poll() {
        final Node<E> f = first;
        return (f == null) ? null : unlinkFirst(f);
    }
```

### unlinkFirst

- 将旧头节点的 item、next 设为空；
- 判断该队列是否只有一个节点，是则将尾节点 last 也设为空。不是，则断开指向旧头节点的连接；
- 容量减一，修改次数加一。


```java
    private E unlinkFirst(Node<E> f) {
        // assert f == first && f != null;
        final E element = f.item;
        final Node<E> next = f.next;
        // 设为空，用于 GC 回收
        f.item = null;
        f.next = null; 
        // 设置下个节点为新的头节点
        first = next;
        // 如果下个节点为空，则设置尾节点为空
        if (next == null)
            last = null;
        else
            // 断开指向旧头节点的连接
            next.prev = null;
        size--;
        modCount++;
        return element;
    }
```

### peek
　　返回头节点的值。

```java
    public E peek() {
        final Node<E> f = first;
        return (f == null) ? null : f.item;
    }
```