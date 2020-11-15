### remove
　　遍历，找到相等的对象删除，时间复杂度为 O(n)。

```java
    public boolean remove(Object o) {
        if (o == null) {
            for (Node<E> x = first; x != null; x = x.next) {
                if (x.item == null) {
                    unlink(x);
                    return true;
                }
            }
        } else {
            for (Node<E> x = first; x != null; x = x.next) {
                if (o.equals(x.item)) {
                    unlink(x);
                    return true;
                }
            }
        }
        return false;
    }
```

#### unlink
　　有三步，分别设置 prev 为 null、next 为 null 和值 item 为 null，这样才完成删除操作，可被 GC 回收。

- 先判断要删除的节点是否为头节点；
    1. 是则设置新的头节点，即指向要删除的节点下个节点；
    2. 不是则断开与前节点的连接，前节点指向要删除的节点的下个节点，并将要删除节点的 prev 设置为 null；
- 接着判断要删除的节点是否为尾节点；
    1. 是则设置新的尾节点，即指向要删除的节点前个节点；
    2. 不是则断开与后节点的连接，后节点指向要删除的节点的前个节点，并将要删除节点的 next 设置为 null；
- 将要删除的节点的值 item 设置为 null；
- 容量减一，修改次数加一。

```java
    E unlink(Node<E> x) {
        // assert x != null;
        // 要删除节点的值
        final E element = x.item;
        // 要删除节点的前一个节点
        final Node<E> next = x.next;
        // 要删除节点的后一个节点
        final Node<E> prev = x.prev;

        // 前节点为空，表示要删除的节点为头节点，则新的节点 first 指向下个节点
        if (prev == null) {
            first = next;
        } else {
            // 删除节点，即前节点指向当前节点的下个节点
            prev.next = next;
            // 删除节点的 prev 指针指向空，断开指向前节点
            x.prev = null;
        }
        
        // 后节点为空，表示要删除的节点为尾节点，设置新的尾节点为要删除的节点的前个节点
        if (next == null) {
            last = prev;
        } else {
            // 删除节点，即后节点指向当前节点的前个节点
            next.prev = prev;
            // 删除节点的 next 指针指向空，断开指向后节点
            x.next = null;
        }
        
        // 删除节点的值为空，这样可被 GC 回收
        x.item = null;
        size--;
        modCount++;
        return element;
    }
```

### remove

- 检查输入的索引是否合法；
- 调用 node，遍历，根据索引找到要删除的节点；
- 调用 unlink，删除该节点。

```java
    public E remove(int index) {
        checkElementIndex(index);
        return unlink(node(index));
    }
```

#### checkElementIndex
　　检查输入的索引是否合法。

```java
    private void checkElementIndex(int index) {
        if (!isElementIndex(index))
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }
    
    private boolean isElementIndex(int index) {
        return index >= 0 && index < size;
    }
```

#### node
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
