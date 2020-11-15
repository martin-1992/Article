### LinkedList 介绍
　　通过构造函数 Node，可知为双向链表。

### 构造函数

```java
public class LinkedList<E>
    extends AbstractSequentialList<E>
    implements List<E>, Deque<E>, Cloneable, java.io.Serializable {
    
    // 容量
    transient int size = 0;
    // 头节点
    transient Node<E> first;
    // 尾节点
    transient Node<E> last;
    
    // 空的构造函数
    public LinkedList() { }
    
    public int size() {
        return size;
    }
}
```


### Node
　　静态内部类 Node，包含三个属性，值 item、指向下个节点的指针 next、指向前个节点的指针 prev。

```java
    private static class Node<E> {
        E item;
        Node<E> next;
        Node<E> prev;

        Node(Node<E> prev, E element, Node<E> next) {
            this.item = element;
            this.next = next;
            this.prev = prev;
        }
    }
```

### [add](https://github.com/martin-1992/Java-Collection-Source-Code/blob/master/LinkedList/add.md)
　　offer 和 add 没区别，因为 offer 是调用 add 方法。添加时检查输入的索引是否合法、队列是否为空，为空则设置新节点为头结点，否则遍历找到前面一个节点，添加到该节点后面。

### [get](https://github.com/martin-1992/Java-Collection-Source-Code/blob/master/LinkedList/get.md)
　　检查索引合法性，遍历获取节点值。

- 先检查索引是否合法；
- 获取链表的中位值，判断该索引是否小于中位值，小于则从前遍历，大于则从后遍历。

### [set](https://github.com/martin-1992/Java-Collection-Source-Code/blob/master/LinkedList/set.md)
　　检查索引合法性，遍历获取旧节点值，覆盖设置新节点值。

### [clear](https://github.com/martin-1992/Java-Collection-Source-Code/blob/master/LinkedList/clear.md)
　　清空链表，遍历每个节点，将节点的 item、next、prev 设为空，将 frist 和 last 设为空，容量为 0。

### [indexOf、lastIndexOf](https://github.com/martin-1992/Java-Collection-Source-Code/blob/master/LinkedList/indexOf.md)
　　从前往后遍历，从后往前遍历，找到第一个或最后一个与该 Object（为 null、不为 null） 相等的节点。

### [poll、peek](https://github.com/martin-1992/Java-Collection-Source-Code/blob/master/LinkedList/poll.md)
　　返回头节点的值，在弹出头节点时，需判断该链表是否为空。为空，则要设置尾节点为空。

### [remove](https://github.com/martin-1992/Java-Collection-Source-Code/blob/master/LinkedList/remove.md)
　　遍历，找到相等的对象删除。

- 判断输入索引是否合法；
- 判断从后遍历还是从前遍历快；
- 判断删除的节点是否为头节点、尾节点。

### 总结

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
