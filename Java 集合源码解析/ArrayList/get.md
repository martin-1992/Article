### get
　　返回该列表对应的索引位置的元素，时间复杂度为 O(1)。

- 调用 rangeCheck 方法，判断输入的索引是否合法，即是否越界；
- 调用 elementData[index]，返回索引位置对应的元素值，并进行类型转换。

```java
    public E get(int index) {
        rangeCheck(index);

        return elementData(index);
    }
```

### set
　　在列表的对应索引位置，新值覆盖旧值，返回旧值，时间复杂度为 O(1)。
  
- 检查输入的索引是否越界；
- 获取该索引对应的旧元素的值；
- 对该索引对应位置的值进行覆盖，新元素覆盖旧元素；
- 赋值成功，返回旧元素的值。

```java
    public E set(int index, E element) {
        rangeCheck(index);

        E oldValue = elementData(index);
        elementData[index] = element;
        return oldValue;
    }
```

### rangeCheck
　　如果输入的索引大于列表大小，则抛出异常。

```java
    private void rangeCheck(int index) {
        if (index >= size)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }
```

### elementData
　　返回该列表对应的索引位置的元素，时间复杂度为 O(1)。

```java
    E elementData(int index) {
        return (E) elementData[index];
    }
```

### size
　　返回数组大小，，时间复杂度为 O(1)。

```java
    // 返回数组大小
    public int size() {
        return size;
    }
```

### isEmpty
　　根据数组大小是否为 0 来判断数组是否为空，时间复杂度为 O(1)。

```java
    public boolean isEmpty() {
        return size == 0;
    }
```
