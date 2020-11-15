### poll
　　移除优先队列中的最小值，调用 [siftDown]() 调整优先队列的结构。

```java
    @SuppressWarnings("unchecked")
    public E poll() {
        if (size == 0)
            return null;
        int s = --size;
        modCount++;
        // 优先队列的最小值
        E result = (E) queue[0];
        E x = (E) queue[s];
        queue[s] = null;
        if (s != 0)
            siftDown(0, x);
        return result;
    }
```

### remove
　　移除优先队列中的指定值。

```java
    public boolean remove(Object o) {
        // 获取该值在数组中的索引
        int i = indexOf(o);
        // 没找到，则返回 false
        if (i == -1)
            return false;
        else {
            // 
            removeAt(i);
            return true;
        }
    }
```

### indexOf
　　返回数组中与 o 相等值的索引。

```java
    private int indexOf(Object o) {
        if (o != null) {
            for (int i = 0; i < size; i++)
                if (o.equals(queue[i]))
                    return i;
        }
        return -1;
    }
```

### removeAt

```java
    @SuppressWarnings("unchecked")
    private E removeAt(int i) {
        // assert i >= 0 && i < size;
        modCount++;
        int s = --size;
        // 要移除的是数组中的最后一个元素，则直接移除，标记为 null
        if (s == i) // removed last element
            queue[i] = null;
        else {
            // 获取该数组的最后一个值
            E moved = (E) queue[s];
            // 将数组的最后一个值置为空
            queue[s] = null;
            // 将数组的最后一个值插入到要删除值的位置，并调整优先队列的结构
            siftDown(i, moved);
            if (queue[i] == moved) {
                siftUp(i, moved);
                if (queue[i] != moved)
                    return moved;
            }
        }
        return null;
    }
```

