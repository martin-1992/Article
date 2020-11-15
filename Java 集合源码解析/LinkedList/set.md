### set
　　遍历，根据索引位置，覆盖旧值，时间复杂度为 O(n)。
　　
- 检查输入索引是否合法；
- 根据索引位置，判断是从前往后遍历，还是从后往前遍历；
- 找到要覆盖的索引位置，覆盖旧值，返回旧值。


```java
    public E set(int index, E element) {
        checkElementIndex(index);
        Node<E> x = node(index);
        E oldVal = x.item;
        x.item = element;
        return oldVal;
    }
```

### checkElementIndex
　　检查输入索引是否合法。

```java
    private void checkElementIndex(int index) {
        if (!isElementIndex(index))
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }
    
    private boolean isElementIndex(int index) {
        return index >= 0 && index < size;
    }
```

### node
　　根据索引位置，判断是从前往后遍历，还是从后往前遍历。

```java
    Node<E> node(int index) {
        // assert isElementIndex(index);
        
        // size >> 1，即除以 2，从前往后遍历
        if (index < (size >> 1)) {
            // 遍历次数为 index
            Node<E> x = first;
            for (int i = 0; i < index; i++)
                x = x.next;
            return x;
        } else {
            // 从后往前遍历
            Node<E> x = last;
            for (int i = size - 1; i > index; i--)
                x = x.prev;
            return x;
        }
    }
```

