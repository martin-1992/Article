### pop
　　弹出（移除）数组最后一个数。

- 调用 [peek()](https://github.com/martin-1992/Java-Collection-Source-Code/blob/master/Stack/peek.md)，获取数组最后一个值；
- 调用 Vector 类的 removeElementAt 方法，使用复制方法，移动覆盖要删除的值，将最后一位置为空。

```java
    public synchronized E pop() {
        E obj;
        int len = size();
        // 获取最后一个数
        obj = peek();
        // 移除
        removeElementAt(len - 1);

        return obj;
    }
```

### Vector#removeElementAt
　　使用复制方法，移动覆盖要删除的值，将最后一位置为空。使用 synchronized，是线程安全。

```java
    public synchronized void removeElementAt(int index) {
        // 修改次数加一
        modCount++;
        // 检查是否越界
        if (index >= elementCount) {
            throw new ArrayIndexOutOfBoundsException(index + " >= " +
                                                     elementCount);
        }
        else if (index < 0) {
            throw new ArrayIndexOutOfBoundsException(index);
        }
        int j = elementCount - index - 1;
        if (j > 0) {
            // 删除元素，即将元素往前移，覆盖掉要删除的元素
            System.arraycopy(elementData, index + 1, elementData, index, j);
        }
        // 容量减一
        elementCount--;
        // 将最后一位置为空，删除元素空出来的
        elementData[elementCount] = null; /* to let gc do its work */
    }
```



