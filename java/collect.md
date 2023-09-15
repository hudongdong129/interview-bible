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
# 普通remove方法，通过下标去删除元素，只要数据未越界不会抛异常
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
# 3、ArrayList remove方法后，会释放空间吗？
```java
public E remove(int index) {
        rangeCheck(index);

        modCount++;
        E oldValue = elementData(index);

        int numMoved = size - index - 1;
        if (numMoved > 0)
        System.arraycopy(elementData, index+1, elementData, index,
        numMoved);
        # 描述的很清楚，只是将对应下标设置为null，方法GC进行回收，但实际回收空间是在GC之后
        elementData[--size] = null; // clear to let GC do its work

        return oldValue;
        }
```
> 总结：remove方法不会立即释放空间，只是将对应的下标数据设置为null，帮助进行GC，真正释放空间是在GC之后
# 4、new ArrayList()和new ArrayList(0)有什么区别？
```java
// 无参构造方法
private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
public ArrayList() {
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}

// 有参构造方法
private static final Object[] EMPTY_ELEMENTDATA = {};

public ArrayList(int initialCapacity) {
        if (initialCapacity > 0) {
        this.elementData = new Object[initialCapacity];
        // 入参为0
        } else if (initialCapacity == 0) {
        this.elementData = EMPTY_ELEMENTDATA;
        } else {
        throw new IllegalArgumentException("Illegal Capacity: "+
        initialCapacity);
        }
}

// 第一次添加元素add()方法，进行数组大小的判断

private void ensureCapacityInternal(int minCapacity) {
        ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
}

private static int calculateCapacity(Object[] elementData, int minCapacity) {
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        return Math.max(DEFAULT_CAPACITY, minCapacity);
        }
        return minCapacity;
}
```
> 总结：无参构造方法和传入0的构造方法，在进行初始化时，都是生成一个空的集合数组
> 不同点：在进行第一次添加元素时，如添加一个新的元素，使用无参的构造方法时，需要取传入大小和默认10取最大值
> 而new ArrayList(0),新的数组容量就是传入的大小 + 1
# 5、ArrayList能不能添加为null的元素?
```java
public boolean add(E e) {
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        elementData[size++] = e;
        return true;
}
```
> 总结：看源码，对传入的值并没有判断是否为null，所以是可以插入为null的元素
# 6、为什么elementData需要被transient修饰？
```java
// transient的作用，被修饰的字段不进行序列化，并且只是针对java的Serializable接口起作用
transient Object[] elementData; // non-private to simplify nested class access

// 内部的writeObject方法
private void writeObject(java.io.ObjectOutputStream s) throws java.io.IOException{
        // 防止序列化期间有修改
        int expectedModCount = modCount;
        // 写出非transient非static属性（会写出size属性）
        s.defaultWriteObject();

        // 写出元素个数
        s.writeInt(size);

        // 依次写出元素
        for (int i=0; i<size; i++) {
            s.writeObject(elementData[i]);
        }

        // 如果有修改，抛出异常
        if (modCount != expectedModCount) {
            throw new ConcurrentModificationException();
        }
}
```
> 总结：因为集合内部元素自己实现了序列化操作，只需根据实际存储数据的size来进行序列化；
> elementData定义为transient的优势，自己根据size序列化真实的元素，而不是根据数组的长度序列化元素，减少了空间占用。
# 7、ArrayList和LinkedList有啥区别？
```java
// ArrayList的底层通过数组进行存储
transient Object[] elementData;
// LinkedList的底层通过Node形成的双向链表进行存储
transient Node<E> first; // 表头
transient Node<E> last; // 表尾
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
> 总结：因为ArrayList底层是数组，随机查找快，插入慢；LinkedList底层通过链表实现随机查找慢，插入快
> 
> · 为什么ArrayList随机查找更快，从时间复杂度来说ArrayList的随机查找复杂度为O(1),LinkedList随机查找为O(n)
> 
> · 从缓存局部性原理来说，数组使用的是连续的内存空间，对CPU缓存更友好。
> 
> 从存储内存的占用角度来说，一般情况下LinkedList占用的内存更多，因为除了自身元素外，还需要存储前后指针数据。
> 但是也有特殊情况，因为ArrayList存在扩容场景，如果一开始存储1w条数据，发生了扩容数组会变为原来的1.5倍，实际占用了1w5个存储空间，可能会更占用内存。
# 8、有没有线程安全的集合容器，它们是如果保证线程安全的？
```java
// 远古集合Vector是线程安全的，通过synchronized关键字来保证线程安全
public class Vector<E>
        extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
    // 通过synchronized关键字保证线程安全
    public synchronized boolean add(E e) {
        modCount++;
        ensureCapacityHelper(elementCount + 1);
        elementData[elementCount++] = e;
        return true;
    }
}
// CopyOnWriteArrayList集合类，通过ReentrantLock实现加锁，并且是在元数据的基础上copy一个新的数组上操作，修改期间不会改变原来的数据
public boolean add(E e) {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        Object[] elements = getArray();
        int len = elements.length;
        Object[] newElements = Arrays.copyOf(elements, len + 1);
        newElements[len] = e;
        setArray(newElements);
        return true;
    } finally {
        lock.unlock();
    }
}
```
> 总结：Vector JDK1.0使用的安全集合类，通过synchronized关键字来实现集合安全
> 
> CopyOnWriteArrayList 是JDK1.5之后引入的，在J.U.C并发包下面的一个安全集合，通过ReentrantLock和写时复制来实现线程安全