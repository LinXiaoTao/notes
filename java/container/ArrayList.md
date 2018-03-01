### 概述

ArrayList 是一个可调整大小，非线程安全的数组，内部实现为数组。

`size`、`isEmpty`、`get`、`set`、`iterator` 和 `listIterator` 操作为 constant time，`add` 操作为 amortized constant time，比如，添加 n 个元素需要 O(n)。除此之外的其他操作大致都为 linear time。

> constant factor 比 LinkedList 实现要低

所有的 ArrayList 实例都有一个容量（capacity）。容量是用于保存这个列表元素的数组的大小，它至少和列表的大小一样。当添加元素到 ArrayList 中，容量将自动增长，但这会考虑到操作的时间复杂度。

在添加大量元素之前使用 `ensureCapacity` 设置合适的容量，将减少容量自动增长的次数。

ArrayList 操作不是同步的。如果有多个线程并发访问 ArrayList 实例，至少有一个线程修改到列表结构，那么必须从外部进行同步。一般这种外部同步是通过对实例的同步进行的，如果没有这样的实例存在，也可以使用 `Collections.synchronizedList()` 进行包装，最好在创建的时候进行包装：

``` java
List list = Collections.synchronizedList(new ArrayList());
```

> 修改列表结构：指添加或者删除、修改内部数组大小等操作，设置某个元素的值不属于这个行为。

可以通过 `iterator()` 和 `listIterator` 返回 fail-fast 的迭代器，在并发修改时，可以快速而干净的失败。

> fail-fast：在迭代器创建成功后，如果列表的结构被除了使用迭代器本身以外的操作修改了，那么迭代器将抛出 [`ConcurrentModificationException`](https://docs.oracle.com/javase/8/docs/api/java/util/ConcurrentModificationException.html)。这个行为不能保证，因为一般来说，在存在非同步并发修改的情况下不可能做出硬性保证，只能尽最大努力。

### 源码分析

#### 构造方法

``` java
// 构造方法可以指定内部存储数组的大小
// 使用 initialCapacity = 0 和 使用默认构造方法 生成的空数组不一样，前一个是 EMPTY_ELEMENTDATA，后一个是 DEFAULTCAPACITY_EMPTY_ELEMENTDATA

public ArrayList(int initialCapacity) {
        if (initialCapacity > 0) {
            this.elementData = new Object[initialCapacity];
        } else if (initialCapacity == 0) {
            this.elementData = EMPTY_ELEMENTDATA;
        } else {
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        }
    }

    public ArrayList() {
        // 使用这个构造函数，当前的 minCapacity 是 DEFAULT_CAPACITY = 10
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }
```

#### capacity

``` java
// 确保合适的最小容量
public void ensureCapacity(int minCapacity) {
    	// 如果当前实例是通过默认构造方法创建，并且没有元素，使用 DEFAULT_CAPACITY，否则为 0
        int minExpand = (elementData != DEFAULTCAPACITY_EMPTY_ELEMENTDATA)
            // any size if not default element table
            ? 0
            // larger than default for default empty table. It's already
            // supposed to be at default size.
            : DEFAULT_CAPACITY;

        if (minCapacity > minExpand) {
            // 是否需要调整容量
            ensureExplicitCapacity(minCapacity);
        }
    }

// 计算合适的容量
private static int calculateCapacity(Object[] elementData, int minCapacity) {
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            // 如果当前是使用默认构造方法，并且为空元素的实例，使用 DEFAULT_CAPACITY 和 minCapacity 的较大值
            return Math.max(DEFAULT_CAPACITY, minCapacity);
        }
        return minCapacity;
    }


private void ensureExplicitCapacity(int minCapacity) {
    	// 设置修改列表结构，记录 modCount
        modCount++;

        // overflow-conscious code
        if (minCapacity - elementData.length > 0)
            // 需要增长
            grow(minCapacity);
    }

// 增长容量确保能够符合最小容量
private void grow(int minCapacity) {
        // 需要注意数据溢出
        int oldCapacity = elementData.length;
    	// 尝试增加一半
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        if (newCapacity - minCapacity < 0)
            // 如果小于最小容量，使用最小容量
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            // 超过最大容量，修正值
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
    }

private static int hugeCapacity(int minCapacity) {
        if (minCapacity < 0) // overflow
            // 出现负数，猜测是值溢出
            throw new OutOfMemoryError();
    	// 使用 MAX_ARRAY_SIZE 或者 Integer.MAX_VALUE
        return (minCapacity > MAX_ARRAY_SIZE) ?
            Integer.MAX_VALUE :
            MAX_ARRAY_SIZE;
    }
```

#### contains

``` java
// 返回列表中是否至少包含一个指定元素，包括 null
public boolean contains(Object o) {
        return indexOf(o) >= 0;
}

public int indexOf(Object o) {
    	// null 或者 not null 两种情况
        if (o == null) {
            for (int i = 0; i < size; i++)
                if (elementData[i]==null)
                    return i;
        } else {
            for (int i = 0; i < size; i++)
                if (o.equals(elementData[i]))
                    return i;
        }
        return -1;
    }
  
public int lastIndexOf(Object o) {
    	// 将数组倒着遍历
        if (o == null) {
            for (int i = size-1; i >= 0; i--)
                if (elementData[i]==null)
                    return i;
        } else {
            for (int i = size-1; i >= 0; i--)
                if (o.equals(elementData[i]))
                    return i;
        }
        return -1;
    }
```

#### get,set,add,remove

``` java
// get,set 操作类似
public E get(int index) {
    	// 下标范围检查
        rangeCheck(index);

        return elementData(index);
    }

E elementData(int index) {
    	// 返回数组元素
        return (E) elementData[index];
    }

public boolean add(E e) {
       // 确保容量
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        elementData[size++] = e;
        return true;
    }

public void add(int index, E element) {
        rangeCheckForAdd(index);

        ensureCapacityInternal(size + 1);  // Increments modCount!!
    	// 将待添加的位置的元素和之后的元素向后移动一位
        System.arraycopy(elementData, index, elementData, index + 1,
                         size - index);
    	// 添加到指定位置
        elementData[index] = element;
        size++;
    }

public E remove(int index) {
        rangeCheck(index);

        modCount++;
        E oldValue = elementData(index);

        int numMoved = size - index - 1;
        if (numMoved > 0)
            // 将待删除下表之后的元素向前移动一位
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
    	// 将最后的一位设置为 null
        elementData[--size] = null; // clear to let GC do its work

        return oldValue;
    }

// 删除第一个指定元素
public boolean remove(Object o) {
        if (o == null) {
            for (int index = 0; index < size; index++)
                if (elementData[index] == null) {
                    fastRemove(index);
                    return true;
                }
        } else {
            for (int index = 0; index < size; index++)
                if (o.equals(elementData[index])) {
                    fastRemove(index);
                    return true;
                }
        }
        return false;
    }

// 快速删除，不检查边界同时不返回删除元素
private void fastRemove(int index) {
        modCount++;
        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // clear to let GC do its work
    }
```

#### removeAll,retainAll

``` java
// 删除指定的所有元素
public boolean removeAll(Collection<?> c) {
        Objects.requireNonNull(c);
        return batchRemove(c, false);
    }

// 只保留指定的元素
public boolean retainAll(Collection<?> c) {
        Objects.requireNonNull(c);
        return batchRemove(c, true);
    }

 private boolean batchRemove(Collection<?> c, boolean complement) {
        final Object[] elementData = this.elementData;
        int r = 0, w = 0;
        boolean modified = false;
        try {
            for (; r < size; r++)
                if (c.contains(elementData[r]) == complement)
                    // 保留
                    elementData[w++] = elementData[r];
        } finally {
            // Preserve behavioral compatibility with AbstractCollection,
            // even if c.contains() throws.
            if (r != size) {
                // 如果发生了问题，将剩下的所有元素都保留
                System.arraycopy(elementData, r,
                                 elementData, w,
                                 size - r);
                w += size - r;
            }
            if (w != size) {
                // 有元素不需要
                // clear to let GC do its work
                for (int i = w; i < size; i++)
                    // 清除不需要的元素，GC
                    elementData[i] = null;
                modCount += size - w;
                size = w;
                modified = true;
            }
        }
        return modified;
    }
```

#### iterator,listIterator

``` java
public Iterator<E> iterator() {
        return new Itr();
    }

// AbstractList.Itr 的优化版本
private class Itr implements Iterator<E> {
        int cursor;       // 下一个返回的元素下标
        int lastRet = -1; // 最后返回元素下标
        int expectedModCount = modCount; // modCount 用于检查并发修改

        Itr() {}

        public boolean hasNext() {
            return cursor != size;
        }

        // 移动到下一个元素
        @SuppressWarnings("unchecked")
        public E next() {
            // 检查并发修改
            checkForComodification();
            int i = cursor;
            if (i >= size)
               // 超过边界
                throw new NoSuchElementException();
            Object[] elementData = ArrayList.this.elementData;
            if (i >= elementData.length)
                // 可能是由并发修改导致的
                throw new ConcurrentModificationException();
            cursor = i + 1;
            // 使用外部类 ArrayList 的内部存储数组，保存 lastRet
            return (E) elementData[lastRet = i];
        }

        public void remove() {
            if (lastRet < 0)
                // 当前没有元素被返回 next()
                throw new IllegalStateException();
            // 检查并发修改
            checkForComodification();

            try {
                ArrayList.this.remove(lastRet);
                cursor = lastRet;
                lastRet = -1;
                expectedModCount = modCount;
            } catch (IndexOutOfBoundsException ex) {
                // 可能是由并发修改导致的
                throw new ConcurrentModificationException();
            }
        }

        @Override
        @SuppressWarnings("unchecked")
    	// 遍历剩余元素
        public void forEachRemaining(Consumer<? super E> consumer) {
            Objects.requireNonNull(consumer);
            final int size = ArrayList.this.size;
            int i = cursor;
            if (i >= size) {
                // 没有剩余元素
                return;
            }
            final Object[] elementData = ArrayList.this.elementData;
            if (i >= elementData.length) {
                // 可能是由并发修改导致的
                throw new ConcurrentModificationException();
            }
            while (i != size && modCount == expectedModCount) {
                // 遍历剩余元素
                consumer.accept((E) elementData[i++]);
            }
            // update once at end of iteration to reduce heap write traffic
            cursor = i;
            lastRet = i - 1;
            checkForComodification();
        }

        final void checkForComodification() {
            // modCount 不相同，抛出并发修改异常
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
        }
    }


private class ListItr extends Itr implements ListIterator<E> {
        ListItr(int index) {
            super();
            cursor = index;
        }

        public boolean hasPrevious() {
            return cursor != 0;
        }

        public int nextIndex() {
            return cursor;
        }

        public int previousIndex() {
            return cursor - 1;
        }

        @SuppressWarnings("unchecked")
    	// 移动到上一个元素
        public E previous() {
            checkForComodification();
            int i = cursor - 1;
            if (i < 0)
                throw new NoSuchElementException();
            Object[] elementData = ArrayList.this.elementData;
            if (i >= elementData.length)
                throw new ConcurrentModificationException();
            cursor = i;
            return (E) elementData[lastRet = i];
        }

    	// 同 ArrayList.set 相比多了 并发修改 检查
        public void set(E e) {
            if (lastRet < 0)
                throw new IllegalStateException();
            checkForComodification();

            try {
                ArrayList.this.set(lastRet, e);
            } catch (IndexOutOfBoundsException ex) {
                throw new ConcurrentModificationException();
            }
        }

    	// 同 ArrayList.add 相比多了 并发修改 检查
        public void add(E e) {
            checkForComodification();

            try {
                int i = cursor;
                ArrayList.this.add(i, e);
                cursor = i + 1;
                lastRet = -1;
                expectedModCount = modCount;
            } catch (IndexOutOfBoundsException ex) {
                throw new ConcurrentModificationException();
            }
        }
    }
```

#### other

``` java
// 返回一个 SubList，使用新的大小进行范围检查，其他操作委托给 ParentList，共用一个内部存储数组
public List<E> subList(int fromIndex, int toIndex) {
        subListRangeCheck(fromIndex, toIndex, size);
        return new SubList(this, 0, fromIndex, toIndex);
    }

	// 遍历，有 并发修改 检查
	@Override
    public void forEach(Consumer<? super E> action) {
        Objects.requireNonNull(action);
        final int expectedModCount = modCount;
        @SuppressWarnings("unchecked")
        final E[] elementData = (E[]) this.elementData;
        final int size = this.size;
        for (int i=0; modCount == expectedModCount && i < size; i++) {
            action.accept(elementData[i]);
        }
        if (modCount != expectedModCount) {
            throw new ConcurrentModificationException();
        }
    }
	
	// 删除符合条件的元素
	@Override
    public boolean removeIf(Predicate<? super E> filter) {
        Objects.requireNonNull(filter);
        // figure out which elements are to be removed
        // any exception thrown from the filter predicate at this stage
        // will leave the collection unmodified
        int removeCount = 0;
        final BitSet removeSet = new BitSet(size);
        final int expectedModCount = modCount;
        final int size = this.size;
        for (int i=0; modCount == expectedModCount && i < size; i++) {
            @SuppressWarnings("unchecked")
            final E element = (E) elementData[i];
            if (filter.test(element)) {
                // 需要删除，设置下标为 true
                removeSet.set(i);
                removeCount++;
            }
        }
        if (modCount != expectedModCount) {
            throw new ConcurrentModificationException();
        }

        // shift surviving elements left over the spaces left by removed elements
        final boolean anyToRemove = removeCount > 0;
        if (anyToRemove) {
            final int newSize = size - removeCount;
            for (int i=0, j=0; (i < size) && (j < newSize); i++, j++) {
                // 返回从 i 开始之前或者之后，第一个为 false 的下标
                i = removeSet.nextClearBit(i);
                elementData[j] = elementData[i];
            }
            for (int k=newSize; k < size; k++) {
                // 清除
                elementData[k] = null;  // Let gc do its work
            }
            this.size = newSize;
            if (modCount != expectedModCount) {
                throw new ConcurrentModificationException();
            }
            modCount++;
        }

        return anyToRemove;
    }


	// 替换符合条件的元素
	@Override
    @SuppressWarnings("unchecked")
    public void replaceAll(UnaryOperator<E> operator) {
        Objects.requireNonNull(operator);
        final int expectedModCount = modCount;
        final int size = this.size;
        for (int i=0; modCount == expectedModCount && i < size; i++) {
            elementData[i] = operator.apply((E) elementData[i]);
        }
        if (modCount != expectedModCount) {
            throw new ConcurrentModificationException();
        }
        modCount++;
    }

	// 排序
	@Override
    @SuppressWarnings("unchecked")
    public void sort(Comparator<? super E> c) {
        final int expectedModCount = modCount;
        Arrays.sort((E[]) elementData, 0, size, c);
        if (modCount != expectedModCount) {
            throw new ConcurrentModificationException();
        }
        modCount++;
    }
```

#### ArrayListSpliterator







