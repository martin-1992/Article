### 数据结构
　　同 HashMap 一样，也是使用数组 + 链表 + 红黑树组成。

- 当链表长度大于 8 时，链表会转为红黑树；
- 默认大小为 16，负载因子为 0.75；
- 扩容为 2 两倍扩容，保证数组大小为 2 的倍数；
- (h = key.hashCode()) ^ (h >>> 16) & HASH_BITS，哈希同 HashMap 类似，都会右移 16，再异或一次；
- 不运行 key 和 value 为 null，而 HashMap 允许。

```java
    /**
     * 最大容量
     */
    private static final int MAXIMUM_CAPACITY = 1 << 30;

    /**
     * 默认大小
     */
    private static final int DEFAULT_CAPACITY = 16;

    /**
     * 负载因子
     */
    private static final float LOAD_FACTOR = 0.75f;

    /**
     * 大于等于该值，则转为红黑树
     */
    static final int TREEIFY_THRESHOLD = 8;

    /*
     * Encodings for Node hash fields. See above for explanation.
     */
    static final int MOVED     = -1; // hash for forwarding nodes
    static final int TREEBIN   = -2; // hash for roots of trees
    static final int RESERVED  = -3; // hash for transient reservations
    static final int HASH_BITS = 0x7fffffff; // usable bits of normal node hash
```

### [put]()

- spread，获取 key 的 hash 值；
- 根据 key 的 hash 值，计算出索引下标。使用 tabAt，为 volatile，保证获取是最新的；
- 找到该索引下标，进行插入；
    1. 判断为链表。如果为空，则将其包装成新节点，使用 CAS 来插入。使用循环，不插入成功不会退出；
    2. 判断为链表。不为空，则使用节点锁，synchronized 锁住该节点。找到旧值，则进行替代。没有找到旧值，则将其包装成新节点，添加到末尾；
    3. 判断为红黑树，则使用节点，synchronized 锁住该节点，进行插入。

### [get]()
　　获取为读操作，不需要加锁。注意，在 [Node]() 中 val 和 next 指针都使用 volatile 修饰，保证获取的是最新的值。

- 通过 hash 来获取，找到，则直接返回；
- 找到节点，则遍历链表，查找并返回。

### 线程安全

- get 操作，节点值使用 volatile 修饰，保证可见性，获取的是最新值；
- put 操作。tabAt 使用 volatile，获取最新的节点值。使用 CAS 插入新节点或 synchronized 锁住节点，进行节点值更新和添加尾节点操作。

### 和 HashTable 区别
　　HashTable 效率低，因为是在方法上使用 synchronized（粗粒度锁），相当于锁住整个数组。

