
### Iterator
　　迭代器，用于遍历列表中的元素。

- hasNext()，通过比较当前索引位置与容量是否相等，来判断是否还可以继续遍历；
- next()，先检查列表是否有修改，接着检查索引是否有效，最后返回该索引位置的元素；
- remove()，调用 remove() 方法来移除；
- checkForComodification()，原理类似 CAS，保留当前的修改次数 expectedModCount，如果列表在迭代期间经过修改（比如多线程的添加、删除），则 expectedModCount != modCount，则抛出异常。

```java
    public Iterator<E> iterator() {
        return new Itr();
    }

    private class Itr implements Iterator<E> {
        // 下一个元素的索引位置
        int cursor;     
        // 上一次返回的元素的索引下标
        int lastRet = -1; 
        // 原理类似 CAS，保留当前的修改次数 expectedModCount，如果列表在迭代期间经
        // 过修改（比如添加、删除），则 expectedModCount != modCount，则抛出异常
        int expectedModCount = modCount;

        Itr() {}
        
        // 下个元素的索引位置不等于列表大小，即还可以继续遍历，返回 true
        public boolean hasNext() {
            return cursor != size;
        }
        
        @SuppressWarnings("unchecked")
        public E next() {
            // 检查列表是否有修改
            checkForComodification();
            // 检查索引是否有效，即是否超过容量大小和列表大小
            int i = cursor;
            if (i >= size)
                throw new NoSuchElementException();
            Object[] elementData = ArrayList.this.elementData;
            if (i >= elementData.length)
                throw new ConcurrentModificationException();
            cursor = i + 1;
            return (E) elementData[lastRet = i];
        }

        public void remove() {
            // 连续两次调用 remove 方法会抛异常，因为第一次调用后 lastRet 会被设为 -1
            if (lastRet < 0)
                throw new IllegalStateException();
            checkForComodification();

            try {
                ArrayList.this.remove(lastRet);
                cursor = lastRet;
                // 防止连续两次调用此方法
                lastRet = -1;
                // 重新设置 expectedModCount，因为 remove 方法会让 modCount+1
                expectedModCount = modCount;
            } catch (IndexOutOfBoundsException ex) {
                throw new ConcurrentModificationException();
            }
        }

        @Override
        @SuppressWarnings("unchecked")
        public void forEachRemaining(Consumer<? super E> consumer) {
            Objects.requireNonNull(consumer);
            final int size = ArrayList.this.size;
            int i = cursor;
            if (i >= size) {
                return;
            }
            final Object[] elementData = ArrayList.this.elementData;
            if (i >= elementData.length) {
                throw new ConcurrentModificationException();
            }
           
            while (i != size && modCount == expectedModCount) {
                consumer.accept((E) elementData[i++]);
            }
            // update once at end of iteration to reduce heap write traffic
            cursor = i;
            lastRet = i - 1;
            checkForComodification();
        }
        
        // 原理类似 CAS，保留当前的修改次数 expectedModCount，如果列表在迭代期间经
        // 过修改（比如添加、删除），则 expectedModCount != modCount，则抛出异常
        final void checkForComodification() {
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
        }
    }
```
