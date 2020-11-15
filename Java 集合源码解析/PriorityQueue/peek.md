### peek
　　返回优先队列的最小值，即根节点值，数组中的第一个。

```java
    @SuppressWarnings("unchecked")
    public E peek() {
        return (size == 0) ? null : (E) queue[0];
    }
```