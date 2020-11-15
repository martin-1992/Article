
### toArray
　　将数组中的元素复制到新数组，并返回，时间复杂度为 O(n)。

- 如果数组大小小于列表容量，则创建一个新列表，将值复制过去，并返回；
- 如果数组大小大于列表容量，则将 a[size] = null，可确认列表的大小 size。注意只有当数组中没有 null 才可使用，不然就没法确认列表大小。

```java
    @SuppressWarnings("unchecked")
    public <T> T[] toArray(T[] a) {
        if (a.length < size)
            // Make a new array of a's runtime type, but my contents:
            return (T[]) Arrays.copyOf(elementData, size, a.getClass());
        System.arraycopy(elementData, 0, a, 0, size);
        if (a.length > size)
            a[size] = null;
        return a;
    }
    
    public Object[] toArray() {
        return Arrays.copyOf(elementData, size);
    }
```

### clone
　　复制一个数组，将数组的元素复制到新数组中，时间复杂度为 O(n)。

```java
    public Object clone() {
        try {
            ArrayList<?> v = (ArrayList<?>) super.clone();
            v.elementData = Arrays.copyOf(elementData, size);
            v.modCount = 0;
            return v;
        } catch (CloneNotSupportedException e) {
            // this shouldn't happen, since we are Cloneable
            throw new InternalError(e);
        }
    }
```
