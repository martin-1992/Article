## Java-Collection-Source-Code
 
### [ArrayList](https://github.com/martin-1992/Java-Collection-Source-Code/tree/master/ArrayList)
　　底层为数组 Object[]，默认数组大小为 10，扩容为 1.5 倍。

- 对数组的操作，通过遍历、复制来实现的；
  1. 遍历，查找指定的值、指定的索引；
  2. 复制，扩容、覆盖要删除的值。
- 在进行数组操作（比如添加、查找值时），要先进行参数校验。
  1. 检查输入的索引是否合法，小于 0 或越界；
  2. 检查数组是否能存放新的值，是否要扩容；
  3. 检查扩容后的大小是否为溢出值。

### [LinkedList](https://github.com/martin-1992/Java-Collection-Source-Code/tree/master/LinkedList)
　　为双向链表，构造内置函数 Node。包含三个属性，值 item、指向下个节点的指针 next、指向前个节点的指针 prev，通过指针来串联各个 Node。

- 对链表的操作，通过遍历、操作节点的前后指针实现的；
    1. 遍历，找到指定的值、节点。计算链表的中位数，通过比较索引和中位数的大小，来判断是从前遍历还是从后遍历比较快，性能优化的一点；
    2. 添加节点，将节点的 next 指针指向新节点；
    3. 删除节点，遍历，将节点的 prev、next、val 设为 null，将前面一个节点和后面一个节点连接起来；
    3. 设置节点，遍历，新值覆盖旧值。
- 在对链表操作时（比如添加、删除节点）时，要进行检查。
    1. 检查输入的索引是否越界；
    2. 检查删除、添加的节点是否为头节点、尾节点，如果自己构建，使用哨兵模式，就不需关注这些；
    3. 添加节点时，判断队列是否为空，为空则要设置头节点和尾节点为新节点；
    4. 比较索引的值和链表的中位数，判断是从后遍历还是从前遍历快。
  
### [Stack](https://github.com/martin-1992/Java-Collection-Source-Code/tree/master/Stack)
　　Stack 是使用数组 Object[] 实现的，其操作只是添加、删除末尾值。

- push，添加末尾值时。要先确认容量是否足够，不够两倍扩容时，要判断新值是否溢出；
- pop，删除末尾值时。使用数组长度减一作为索引，设该索引为 null。

### [HashMap](https://github.com/martin-1992/Java-Collection-Source-Code/tree/master/HashMap)
　　HashMap 是由数组 + 链表 + 红黑树组成，数组中每个值为链表节点或红黑树节点，当链表长度大于 8 时，会转为红黑树。HashMap 在哈希时遇到哈希冲突使用链表法，通过链表连接冲突的节点。<br />
　　负载因子为 0.75，默认大小为 16，进行两倍库容，保证为 2 的幂大小。

- HashMap 的容量必须为 2 的次方，这样可使用位运算，根据 (n - 1) & hash(key) 来获得该 key 在数组中的索引位置 index，等价于取模运算；
- 在取 hash 值，进行二次哈希。将哈希值的高位右移 16 位，与原先的哈希值再进行一次异或操作，即为 (h = key.hashCode()) ^ (h >>> 16)；
- 使用位运算 (n - 1) & hash(key) 来计算索引值；
- 两倍扩容时，需要重新哈希。同一个链表节点上的节点值，**通过 e.hash & n 来判断其高位是否为 1，来判断该值是否大于旧容量值 oldCap。** 注意与 e.hash & (n-1) 求的是余数，而 e.hash & n 通过高位来判断出该值是否大于容量值。大于，即高位为 1，则重新哈希后的节点索引值为旧索引位置 + 旧容量 oldCap。因为是两倍扩容，所以这里是加上旧容量 oldCap。

### [PriorityQueue](https://github.com/martin-1992/Java-Collection-Source-Code/tree/master/PriorityQueue)

### [ConcurrentHashMap]()
　　同 HashMap 一样，也是使用数组 + 链表 + 红黑树组成。

- 当链表长度大于 8 时，链表会转为红黑树；
- 默认大小为 16，负载因子为 0.75；
- 扩容为 2 两倍扩容，保证数组大小为 2 的倍数；
- (h = key.hashCode()) ^ (h >>> 16) & HASH_BITS，哈希同 HashMap 类似，都会右移 16，再异或一次；
- 不运行 key 和 value 为 null，而 HashMap 允许。

#### 线程安全

- get 操作，节点值使用 volatile 修饰，保证可见性，获取的是最新值；
- put 操作。tabAt 使用 volatile，获取最新的节点值。使用 CAS 插入新节点或 synchronized 锁住节点，进行节点值更新和添加尾节点操作。


### reference

- [部分图片来自 JCFInternals](https://github.com/CarpenterLee/JCFInternals/blob/master/README.md)

