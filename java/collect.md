# 1、ArrayList怎么进行扩容的？
```java
private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
}
```
> 总结：从代码中可以看出，进行扩容并不一定就是扩大到原来的1.5倍，是有相应的判断逻辑
- 默认扩容为原来长度的1.5倍
- 如果1.5倍的大小小于需要的容量，则使用需要的容量 及传入的minCapacity
- 如果扩容后大小大于最大值，则使用最大值 及 Integer.MAX_VALUE - 8 的值
# 2、ArrayList能遍历调用remove方法吗？
```java
# 普通remove方法，通过下标去删除元素，只要未数据越界不会抛异常
public E remove(int index) {
        rangeCheck(index);

        modCount++;
        E oldValue = elementData(index);

        int numMoved = size - index - 1;
        if (numMoved > 0)
        System.arraycopy(elementData, index+1, elementData, index,
        numMoved);
        elementData[--size] = null; // clear to let GC do its work

        return oldValue;
        }
# 内部类Itr实现的remove，实现了fail-fast机制，会判断是否有被修改，
private class Itr implements Iterator<E> {
    public void remove() {
        if (lastRet < 0)
            throw new IllegalStateException();
        checkForComodification();

        try {
            ArrayList.this.remove(lastRet);
            cursor = lastRet;
            lastRet = -1;
            expectedModCount = modCount;
        } catch (IndexOutOfBoundsException ex) {
            throw new ConcurrentModificationException();
        }
    }
    
    # 第一次读取数expectedModCount 和 modCount不一致,抛出异常,modCount每次修改都会+1
    final void checkForComodification() {
        if (modCount != expectedModCount)
            throw new ConcurrentModificationException();
    }
}
```
> 总结：可以使用普通的下标for循环进行删除数据；使用for-each则会抛出异常;因为for-each底层使用就是Iterator
> 同时需要注意使用普通for循环遍历删除数据，会将删除下标之后的数据向前移动一位；需要注意数据的变动和size的变化
