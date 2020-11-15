### add
　　add 方法调用 offer 方法。

```java
    public boolean add(E e) {
        return offer(e);
    }
```

### offer
　　将值插入到优先队列中。

- 参数校验，不能插入 null；
- 判断是否需要扩容，调用 [grow](https://github.com/martin-1992/Java-Collection-Source-Code/blob/master/PriorityQueue/grow.md)，通过将旧数组的值复制到新数组，当旧数组容量小于 64，则两倍扩容 + 2，否则扩容 1.5 倍；
- 如果数组为空，则直接插入；
- 数组不为空，调用 [siftUp](https://github.com/martin-1992/Java-Collection-Source-Code/blob/master/PriorityQueue/siftUp.md)，比较父节点和要插入的值大小，如果要插入的值 < 父节点，则继续往上查找父节点，直到找到比插入值还小的父节点，插入到该父节点下的子节点。

![avatar](photo_2.png)

```java
    transient int modCount = 0;
    // 优先队列中存在值的数量
    private int size = 0;
    // 数值形式的优先队列
    transient Object[] queue;

    public boolean offer(E e) {
        // 参数校验
        if (e == null)
            throw new NullPointerException();
        // 修改次数加一，迭代器在迭代时会通过 CAS 来校验 modCount 是否
        // 发生改动，从而判断迭代器是否发生变化
        modCount++;
        int i = size;
        // 扩容
        if (i >= queue.length)
            grow(i + 1);
        size = i + 1;
        // 数组为空，则直接添加
        if (i == 0)
            queue[0] = e;
        else
            // 数组不为空，比较父节点和要插入的值大小，如果要插入的值 < 父节点，则继续往上查找父节点，
            // 直到找到比插入值还小的父节点，插入到该父节点下的子节点
            siftUp(i, e);
        return true;
    }
```
