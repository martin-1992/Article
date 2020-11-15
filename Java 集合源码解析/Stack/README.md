### Stack
　　Stack 的方法调用的是 Vector 类的方法，**使用数组来实现 push、peek 和 pop，** 这些方法使用 synchronized 修饰，属于线程安全的。

### 构造函数
　　空的构造函数。

```java
  public Stack() {
  }
```

### [push](https://github.com/martin-1992/Java-Collection-Source-Code/blob/master/Stack/push.md)
　　将值添加到数组末尾。

- 先确认容量是否足够存放新值；
- 不够，则两倍扩容。判断两倍值是否溢出，溢出则以最大值为准，将旧数组的值复制到新数组中。

### [pop](https://github.com/martin-1992/Java-Collection-Source-Code/blob/master/Stack/pop.md)
　　调用 peek 获取最后一个值，通过数组长度减一作为索引，将数组最后一个值设为 null。

### [peek](https://github.com/martin-1992/Java-Collection-Source-Code/blob/master/Stack/peek.md)
　　获取数组的最后一个值，通过数组长度减一作为索引来获取。注意要判断数组长度为空时，抛出空栈异常。

### 总结
　　Stack 是使用数组实现的，其操作只是添加、删除末尾值。

- push，添加末尾值时。要先确认容量是否足够，不够两倍扩容时，要判断新值是否溢出；
- pop，删除末尾值时。使用数组长度减一作为索引，设该索引为 null。
