### add
　　添加值到队列末尾。

```java
    public boolean add(E e) {
        linkLast(e);
        return true;
    }
```

### offer
　　offfer 和 add 没区别。

```java
    public boolean offer(E e) {
        return add(e);
    }
```

#### linkLast
　　将值包装成节点，添加到队列末尾。

- 将要添加的值，保证成节点 [Node](https://github.com/martin-1992/Java-Collection-Source-Code/tree/master/LinkedList)，并设置新节点的 prev 指针指向 last 节点；
- 设置新的 last 为新节点；
- 判断，如果旧 last 节点为空，表名为空队列，first 节点也指向新节点。否则 last 节点的 next 指针指向新节点；
- 容量加一，修改次数加一。

```java
    void linkLast(E e) {
        final Node<E> l = last;
        // 将要添加的值，保证成节点 Node，并设置新节点的 prev 指针指向 last 节点
        final Node<E> newNode = new Node<>(l, e, null);
        // 设置新的 last 为新节点
        last = newNode;
        // 旧 last 节点为空，表名为空队列，first 节点也指向新节点
        if (l == null)
            first = newNode;
        else
            // last 节点的 next 指针指向新节点
            l.next = newNode;
        size++;
        modCount++;
    }
```

### add
　　添加值到指定位置。

- 检查输入的索引是否合法；
- 如果索引等于容量，即将值添加到末尾；
- 根据索引位置，遍历找到要插入节点的位置；
- 将值包装成节点，插入到 index 前面。如果插入的位置为头节点，则设置插入的节点为新的头节点。

```java
    public void add(int index, E element) {
        checkPositionIndex(index);
        // 将值添加到末尾
        if (index == size)
            linkLast(element);
        else
            linkBefore(element, node(index));
    }
```

#### checkPositionIndex
　　检查输入的索引是否合法。

```java
    private void checkPositionIndex(int index) {
        if (!isPositionIndex(index))
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }
    
    private boolean isPositionIndex(int index) {
        return index >= 0 && index <= size;
    }
```

### node
　　根据索引位置，判断是从前往后遍历，还是从后往前遍历，返回该节点。

```java
    Node<E> node(int index) {
        // assert isElementIndex(index);
        
        // size >> 1，即除以 2，从前往后遍历
        if (index < (size >> 1)) {
            // 遍历次数为 index
            Node<E> x = first;
            for (int i = 0; i < index; i++)
                x = x.next;
            return x;
        } else {
            // 从后往前遍历
            Node<E> x = last;
            for (int i = size - 1; i > index; i--)
                x = x.prev;
            return x;
        }
    }
```

#### linkBefore
　　将值 e 包装成节点，插入到 succ 前面，要判断插入的位置是否为头节点，是则设置当前插入的节点为新的头节点。

```java
    void linkBefore(E e, Node<E> succ) {
        // assert succ != null;
        final Node<E> pred = succ.prev;
        final Node<E> newNode = new Node<>(pred, e, succ);
        succ.prev = newNode;
        // 要插入的节点位置为头节点，则设置新的头节点为要插入的节点，这里不判断尾节点，
        // 是因为在代码 add 中以判断是否插入尾节点
        if (pred == null)
            first = newNode;
        else
            pred.next = newNode;
        size++;
        modCount++;
    }
```
