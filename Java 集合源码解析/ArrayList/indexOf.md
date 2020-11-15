
### indexOf
　　通过遍历，获得该元素在数组中的索引位置（返回第一个与该元素相等的），时间复杂度为 O(n)。

- 如果该元素为空，返回数组中第一个为空的索引；
- 不为空，则遍历数组中的元素，返回第一个相等的元素的索引位置；
- 最后没找到，则返回 -1。

```java
    public int indexOf(Object o) {
        if (o == null) {
            for (int i = 0; i < size; i++)
                if (elementData[i]==null)
                    return i;
        } else {
            for (int i = 0; i < size; i++)
                if (o.equals(elementData[i]))
                    return i;
        }
        return -1;
    }
```

### contains
　　数组中是否存在该元素，通过遍历数组中的元素，判断是否有相等的，时间复杂度为 O(n)。

```java
    public boolean contains(Object o) {
        // 找到索引，则一定大于等于 0，即为 true，找不到为 -1，返回 false
        return indexOf(o) >= 0;
    }
```

### lastIndexOf
　　与 indexOf 的区别是，indexOf 是从前往后遍历，获取第一个与该元素相等的索引位置。而 lastIndexOf 是从后往前遍历，获取最后一个与该元素相等的索引位置，时间复杂度为 O(n)。

```java
    public int lastIndexOf(Object o) {
        if (o == null) {
            for (int i = size-1; i >= 0; i--)
                if (elementData[i]==null)
                    return i;
        } else {
            for (int i = size-1; i >= 0; i--)
                if (o.equals(elementData[i]))
                    return i;
        }
        return -1;
    }
```
