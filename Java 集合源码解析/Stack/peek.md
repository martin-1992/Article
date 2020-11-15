### peek
　　使用索引下标，获取数组最后一个数。

```java
    public synchronized E peek() {
        int len = size();
        // 空栈异常
        if (len == 0)
            throw new EmptyStackException();
        return elementAt(len - 1);
    }
```

### Vector#elementAt
　　栈是一个数组，所以使用下标索引获取最后一位值，即可。

```java
    public synchronized E elementAt(int index) {
        if (index >= elementCount) {
            throw new ArrayIndexOutOfBoundsException(index + " >= " + elementCount);
        }

        return elementData(index);
    }
```

### Vector#elementData
　　索引下标获取数组对应的值。

```java
    protected Object[] elementData;

    @SuppressWarnings("unchecked")
    E elementData(int index) {
        return (E) elementData[index];
    }
```
