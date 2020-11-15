### siftDown
　　siftDownUsingComparator 和 siftDownComparable 区别在于 siftDownUsingComparator 使用自定义的比较器 comparator 来判断两个值的大小。

```java
    private void siftDown(int k, E x) {
        if (comparator != null)
            siftDownUsingComparator(k, x);
        else
            siftDownComparable(k, x);
    }
```

### siftDownComparable
　　通过比较最后一个值 x 和要删除节点下的子节点值 child，如果 x > child，则继续往下遍历，直到找到比 x 还大的子节点值，插入到该位置。

```java
    private int size = 0;

    @SuppressWarnings("unchecked")
    private void siftDownComparable(int k, E x) {
        Comparable<? super E> key = (Comparable<? super E>)x;
        // 将数组当前容量除以 2
        int half = size >>> 1;
        // 只有 k < half 才需要调整，如果是 k >= half，表示要删除的值在最后一层，无需调整
        while (k < half) {
            // 获取该节点下的左子节点，这里是假设左子节点比右子节点小
            int child = (k << 1) + 1;
            Object c = queue[child];
            // 右子节点
            int right = child + 1;
            // 左子节点和右子节点进行比较，如果右子节点值比较小，则 c 为右子节点的值
            if (right < size &&
                ((Comparable<? super E>) c).compareTo((E) queue[right]) > 0)
                c = queue[child = right];
            // 如果最后一个值小于等于该节点的子节点的值，即最后一个值 x <= queue[child]，
            // 则将最后一个值插入到该节点，否则继续往下遍历，因为小顶堆是小的在上面，所以
            // 需要将大的值往下移
            if (key.compareTo((E) c) <= 0)
                break;
            // 继续遍历，子节点上移，插入到父节点的位置
            queue[k] = c;
            // 设置子节点为新的 k，继续往下遍历
            k = child;
        }
        // 将数组中的最后一个值移动到索引 k
        queue[k] = key;
    }
```

### siftDownUsingComparator
　　代码同 siftDownComparable 基本一致，只是通过自定义的比较器 comparator 来判断两个值的大小。

```java
    @SuppressWarnings("unchecked")
    private void siftDownUsingComparator(int k, E x) {
        int half = size >>> 1;
        while (k < half) {
            int child = (k << 1) + 1;
            Object c = queue[child];
            int right = child + 1;
            if (right < size &&
                comparator.compare((E) c, (E) queue[right]) > 0)
                c = queue[child = right];
            if (comparator.compare(x, (E) c) <= 0)
                break;
            queue[k] = c;
            k = child;
        }
        queue[k] = x;
    }
```