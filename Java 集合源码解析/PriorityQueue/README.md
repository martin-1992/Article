### PriorityQueue
　　PriorityQueue 的 peek() 和 element 操作是常数时间，add()、offer()、无参数的 remove() 以及 poll() 方法的时间复杂度都是 log(N)。

### 构造函数
　　优先队列是使用数组实现的完全二叉树，根据完全二叉树的性质，不会有空的节点，所以可用数组存储。小顶堆指的是该二叉树的根节点值最小，如下图。

![avatar](photo_1.png)

　　通过下面公式，可计算出某个节点的父节点、左子节点、右子节点的下标，节点编号代表数组的索引位置。

- leftNo = parentNo * 2 + 1
- rightNo = parentNo * 2 + 2
- parentNo = (nodeNo - 1) / 2

```java
    public PriorityQueue() {
        this(DEFAULT_INITIAL_CAPACITY, null);
    }

    public PriorityQueue(int initialCapacity) {
        this(initialCapacity, null);
    }

    public PriorityQueue(int initialCapacity,
                         Comparator<? super E> comparator) {
        // 参数校验
        if (initialCapacity < 1)
            throw new IllegalArgumentException();
        this.queue = new Object[initialCapacity];
        this.comparator = comparator;
    }
```

