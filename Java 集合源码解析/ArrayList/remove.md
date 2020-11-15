
### remove(int index)
　　移除指定索引的值，时间复杂度为 O(n)。

- 检查索引是否越界；
- 将要移除索引后的值往前移一格，即末尾的值与倒数第二的值相同，将末尾的值设为 null，使得 GC 回收；
- 返回已移除索引的值。

```java
public E remove(int index) {
    rangeCheck(index);

    modCount++;
    // 获取要移除索引的值
    E oldValue = elementData(index);
    
    int numMoved = size - index - 1;
    // 将索引后的值往前移一格
    if (numMoved > 0)
        System.arraycopy(elementData, index+1, elementData, index,
                         numMoved);
    // 设为 null，让 GC 回收
    elementData[--size] = null; 

    return oldValue;
}
```

### remove(Object o) 
　　删除第一个出现的元素值，包括 null，删除操作同样是往前移一格，并将最后一格设为 null，让 GC 回收，时间复杂度为 O(n)。

```java
public boolean remove(Object o) {
    if (o == null) {
        // 找到第一个 null，并进行删除
        for (int index = 0; index < size; index++)
            if (elementData[index] == null) {
                fastRemove(index);
                return true;
            }
    } else {
        for (int index = 0; index < size; index++)
            if (o.equals(elementData[index])) {
                fastRemove(index);
                return true;
            }
    }
    return false;
}

private void fastRemove(int index) {
    modCount++;
    int numMoved = size - index - 1;
    if (numMoved > 0)
        System.arraycopy(elementData, index+1, elementData, index,
                         numMoved);
    elementData[--size] = null; // clear to let GC do its work
}
```

### removeRange(int fromIndex, int toIndex) 
　　移除指定索引范围的值，同样是移动数组里的值进行覆盖。

```java
protected void removeRange(int fromIndex, int toIndex) {
    modCount++;
    // 将 toIndex 后的元素值往前移，移到 fromIndex 后面
    int numMoved = size - toIndex;
    System.arraycopy(elementData, toIndex, elementData, fromIndex,
                     numMoved);

    // 比如 size 为 9，toIndex=5，fromIndex=2，则往前移动了 3 格，即将数组最后 3 格设为 null，让 GC 回收
    int newSize = size - (toIndex-fromIndex);
    for (int i = newSize; i < size; i++) {
        elementData[i] = null;
    }
    size = newSize;
}
```


### removeAll
　　批量移除，如果 Collection 为 ArrayList，时间复杂度为 O(n^2)。
  
- 判断对象是否为空，为空抛出异常；
- 遍历，判断列表中的元素是否在集合对象中，不在则移动 r 指针并赋值 elementData[w++] = elementData[r]，在则不移动。相当于将不在集合对象中的值往前移，最后 w 指针后面的值置为 null，删除即可；
- finally 为保证当出现异常时，r 指针没遍历完数组时，会将后面的值都往前移到列表中，保证数据完整性。

```java
    public boolean removeAll(Collection<?> c) {
        Objects.requireNonNull(c);
        return batchRemove(c, false);
    }
    
    private boolean batchRemove(Collection<?> c, boolean complement) {
        // 实例列表对象
        final Object[] elementData = this.elementData;
        int r = 0, w = 0;
        boolean modified = false;
        try {
            for (; r < size; r++)
                // 遍历，如果该集合包含要移除的元素，则 r 指针不移动，不包含时才移动 r 指针，
                // 这样可覆盖掉删除的值
                if (c.contains(elementData[r]) == complement)
                    elementData[w++] = elementData[r];
        } finally {
            // finally 主要是防止 c.contains 抛异常时，即 r 指针中断没有遍历完整个列表，
            // 这时会将 r 指针后面没有遍历的值复制到列表中，保证列表完整性
            主要是防止c.contains有异常抛出时能保证elementData数据的完整性
            if (r != size) {
                System.arraycopy(elementData, r,
                                 elementData, w,
                                 size - r);
                w += size - r;
            }
            if (w != size) {
                // 将 w 指针后面的值置为空，即删除这些元素，没被删除的元素已经前移
                for (int i = w; i < size; i++)
                    elementData[i] = null;
                modCount += size - w;
                size = w;
                modified = true;
            }
        }
        return modified;
    }
```

### retainAll
　　与 removeAll 相反，两个集合的交集保留，即将包含的对象往前移。

```java
public boolean retainAll(Collection<?> c) {
        Objects.requireNonNull(c);
        return batchRemove(c, true);
    }
```
