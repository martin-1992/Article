
### clear
　　遍历列表，将列表中的每个元素置为空。注意列表对象不为空，不会被 GC 回收，时间复杂度为 O(n)。

```java
    public void clear() {
        modCount++;

        // clear to let GC do its work
        for (int i = 0; i < size; i++)
            elementData[i] = null;

        size = 0;
    }
```

### trimToSize
　　调整当前列表实例的大小，当列表大小 size 小于列表当前的实际大小，则新建一个列表，将当前列表的值复制到新的列表。通常用于扩容后的列表中有很多值被删除，导致实际容量小于列表大小，可通过该方法，减少列表大小，释放内存，时间复杂度为 O(n)。

```java
    public void trimToSize() {
        modCount++;
        if (size < elementData.length) {
            elementData = (size == 0)
              ? EMPTY_ELEMENTDATA
              : Arrays.copyOf(elementData, size);
        }
    }
```
