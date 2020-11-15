### siftUp
　　siftUpUsingComparator 和 siftUpComparable 区别在于 siftUpUsingComparator 使用自定义的比较器 comparator 来判断两个值的大小。

```java
    private void siftUp(int k, E x) {
        if (comparator != null)
            siftUpUsingComparator(k, x);
        else
            siftUpComparable(k, x);
    }
```

### siftUpComparable
　　通过比较要插入的值和父节点的值，如果要插入的值 < 父节点，则继续往上查找父节点，直到找到比插入值还小的父节点，插入到该父节点下的子节点。

```java
    @SuppressWarnings("unchecked")
    private void siftUpComparable(int k, E x) {
        Comparable<? super E> key = (Comparable<? super E>) x;
        while (k > 0) {
            // 获取父节点的值
            int parent = (k - 1) >>> 1;
            Object e = queue[parent];
            // 父节点的值小于要插入的值，则打破循环，插入到该父节点下的子节点，
            // 否则子节点和父节点进行交换，直到 x >= queue[parent]
            if (key.compareTo((E) e) >= 0)
                break;
            // 父节点下移，插入到子节点的位置
            queue[k] = e;
            // 设置父节点为新的子节点，继续往上查找父节点
            k = parent;
        }
        // 父节点比要插入的值小，将要插入的值放到该父节点下的子节点
        queue[k] = key;
    }
```


### siftUpUsingComparator
　　代码同 siftUpComparable 基本一致，只是通过自定义的比较器 comparator 来判断两个值的大小。

```java
    @SuppressWarnings("unchecked")
    private void siftUpUsingComparator(int k, E x) {
        while (k > 0) {
            int parent = (k - 1) >>> 1;
            Object e = queue[parent];
            if (comparator.compare(x, (E) e) >= 0)
                break;
            queue[k] = e;
            k = parent;
        }
        queue[k] = x;
    }
```

