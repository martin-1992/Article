### grow
　　扩容方法，通过将旧数组的值复制到新数组，当旧数组容量小于 64，则两倍扩容 + 2，否则扩容 1.5 倍。

```java
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8

    private void grow(int minCapacity) {
        // 旧数组的容量
        int oldCapacity = queue.length;
        // Double size if small; else grow by 50%
        // 当旧数组容量小于 64，则两倍扩容 + 2，否则扩容 1.5 倍
        int newCapacity = oldCapacity + ((oldCapacity < 64) ?
                                         (oldCapacity + 2) :
                                         (oldCapacity >> 1));
        // 溢出判断，设置 newCapacity 为最大容量 Integer.MAX_VALUE
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // 将旧数组的值复制到新数组
        queue = Arrays.copyOf(queue, newCapacity);
    }
```

### hugeCapacity
　　当 minCapacity 大于 MAX_ARRAY_SIZE，返回最大容量 Integer.MAX_VALUE，否则返回 Integer.MAX_VALUE - 8。

```java
    private static int hugeCapacity(int minCapacity) {
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError();
        return (minCapacity > MAX_ARRAY_SIZE) ?
            Integer.MAX_VALUE :
            MAX_ARRAY_SIZE;
    }
```