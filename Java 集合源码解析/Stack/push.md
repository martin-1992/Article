### push
　　调用 Vector 的 addElement 方法。

```java
    public E push(E item) {
        addElement(item);

        return item;
    }
```

### Vector#addElement
　　将值添加到数组末尾。

- 判断是否需要自动扩容，需要则创建两倍于旧数组容量的新数组，同时将旧数组的值一一复制到新数组；
- 将值添加到数组末尾。

```java
    protected transient int modCount = 0;

    protected Object[] elementData;
    
    // 栈当前容量
    protected int elementCount;

    public synchronized void addElement(E obj) {
        modCount++;
        // 自动扩容
        ensureCapacityHelper(elementCount + 1);
        // 添加到数组末尾
        elementData[elementCount++] = obj;
    }
```

### ensureCapacityHelper
　　调用 grow 方法进行扩容。

```java
    private void ensureCapacityHelper(int minCapacity) {
        // overflow-conscious code
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }
```

### grow
　　扩容，栈的新容量是旧容量的两倍，判断新容量值是否溢出，最大为 Integer.MAX_VALUE，最后将旧数组的值复制到新数组。

```java
    private void grow(int minCapacity) {
        // 旧容量
        int oldCapacity = elementData.length;
        // 如果 capacityIncrement 大于 0，则新容量为 oldCapacity + capacityIncrement，
        // 否则新容量为旧容量的两倍，而在栈里，capacityIncrement 初始为 0，所以栈的新容
        // 量是旧容量的两倍
        int newCapacity = oldCapacity + ((capacityIncrement > 0) ?
                                         capacityIncrement : oldCapacity);
        // 新容量小于 minCapacity，则设置 minCapacity 为新容量 
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        // 新容量溢出，则校正新容量最大值为 Integer.MAX_VALUE;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // 将旧数组的值复制到新数组
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
```

### hugeCapacity
　　校正容量，当新容量大于 Integer.MAX_VALUE - 8 时，返回 Integer.MAX_VALUE。

```java
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

    private static int hugeCapacity(int minCapacity) {
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError();
        return (minCapacity > MAX_ARRAY_SIZE) ?
            Integer.MAX_VALUE :
            MAX_ARRAY_SIZE;
    }
```