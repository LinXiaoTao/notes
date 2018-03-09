### 参考

[Java 8系列之重新认识HashMap](https://tech.meituan.com/java-hashmap.html)

[API](https://docs.oracle.com/javase/8/docs/api/java/util/HashMap.html)



### 概述

HashMap 允许 null values 和 null key，但是不保证 map 中的顺序，尤其是，不能保证随着时间推移，顺序一直保持不变。

假设散列函数在桶之间正确分散元素，get 和 put 操作是 constant-time。迭代集合需要的时间与实例的 capacity（桶的数量）加上其大小（健值映射数量）成正比。如果迭代性能很重要的话，那么 initial capacity 不能设置太高（或者设置 load factor 太小）。

HashMap 实例有两个参数会影响性能：initial capacity 和 load factor。capacity 是散列表中桶的数量，initial capacity 是散列表创建时桶的数量。而 load factor 是衡量散列表使用程度，需要自动增长 capacity。当散列表中的键值对数量大于 load factor 和 capacity 的乘积时，散列表将以大约两倍扩容（rehashed）。

默认情况下，load factor 等于 0.75 是在时间和空间中一个平衡。太高的值将减少空间开销，但会增加查找成本。设置 initial capacity 时应考虑键值对的数量和 load factor，以便减少扩容操作的次数。如果 initial capacity 大于最大键值对数量处于 load factor，则不会发生扩容。

出现太多相同 `hashCode()` 一样的键将会导致性能下降。如果键是 `Comparable`，HashMap 将给键排序，以降低前面的影响。

HashMap 是非线程安全的。如果多个线程并发访问，并且至少有一个线程修改了结构，它必须在外部进行同步。通常的做法是，使用 HashMap 的实例来进行同步操作，如果没有这样的实例存在，可以使用 `Collections.synchronizedmap()` 进行包装。这个操作最好是在创建的时候进行：

``` java
Map m = Collections.synchronizedMap(new HashMap());
```

> 结构修改：指添加或者删除一个或多个映射，只是改变实例已经存在的键相关联的值不属于结构修改。

HashMap 中所有集合视图方法返回的迭代器都是 fail-fast



### 源码分析

#### 存储结构

从结构实现来讲，HashMap 是数组 + 红黑树（JDK 1.8 增加了红黑树）实现的，如下图所示：

![结构](https://tech.meituan.com/img/java-hashmap/hashMap%E5%86%85%E5%AD%98%E7%BB%93%E6%9E%84%E5%9B%BE.png)

HashMap 内部使用哈希桶数组作为数据底层存储

``` java
/**
 * 第一次使用时进行初始化，并在必要时候会扩容。当分配完，长度总是 2的N次幂
 * 在一些操作中允许长度为 0，以允许当前不需要的引导机制
**/
transient Node<K,V>[] table;                                  
```

使用哈希表来存储，为了解决冲突，HashMap 采用了链地址法。简单来说，就是数组加链表的结合。

> 哈希表为解决冲突，可以采用开放地址法和链地址等来解决问题

HashMap 通过设计良好的 hash 算法来减少 hash 碰撞，使用扩容只在必要时增大哈希桶数组大小，来使得占用空间更少。

几个字段的含义：

``` java
	/**
     * map 中包含键值对的数量
     */
    transient int size;

    /**
     * HashMap 结构被修改的次数
     * 结构修改表示改变了 HashMap 中键值对的数量或者以其他方式修改它的内部结构（比如，扩容）
     * 这个字段用在 HashMap fail-fast 的 Collection-views 做迭代器
     */
    transient int modCount;

    /**
     * 下一次需要扩容的值（capacity * load factor）
     * 
     * @serial
     */
    // (The javadoc description is true upon serialization.
    // Additionally, if the table array has not been allocated, this
    // field holds the initial array capacity, or zero signifying
    // DEFAULT_INITIAL_CAPACITY.)
    int threshold;

    /**
     * The load factor for the hash table.
     * 默认值为 0.75
     * @serial
     */
    final float loadFactor;
```

HashMap 中，哈希桶数组的长度 length 大小必须为 2的N次幂，主要是为了在取模和扩容时做优化，同时减少冲突，在定位哈希桶索引位置时，也加入高位参与运算的过程。

在 JDK 1.8 时，如果链表长度太多（默认为超过 8）时，链表就转换为红黑树，进一步提高性能。

#### 确定哈希桶数组索引位置

第一步是先获取 hash 值：

``` java
static final int hash(Object key) {                              
    int h;
    // h ^ (h >>> 16) 高位参与运算
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}                                                                
```

第二步确认 hash 位于哈希桶数组的索引：

``` java
(length - 1) & hash
```

> 这种计算结果跟取模运算类似，但在长度规定为 2的N次幂 的数组上，这样的计算效率更高。
>
> 因为任意 2的N次幂 的数减一之后的二进制表示都是高位全为 0，低位全为 1。那么和任意的值进行 & 运算，结果都是落在减一的值之间。

![计算过程](https://tech.meituan.com/img/java-hashmap/hashMap%E5%93%88%E5%B8%8C%E7%AE%97%E6%B3%95%E4%BE%8B%E5%9B%BE.png)

#### 分析 put 方法

``` java
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,                    
               boolean evict) {                                                   
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    // n = tab.length
    if ((tab = table) == null || (n = tab.length) == 0)
        // 如果哈希桶数组还没初始化，初始化它
        n = (tab = resize()).length;
    // p = tab[index]
    if ((p = tab[i = (n - 1) & hash]) == null)
        // hash 在哈希桶数组中对应位置的节点还没初始化，初始化指定索引的节点
        tab[i] = newNode(hash, key, value, null);                                 
    else {                                                                        
        Node<K,V> e; K k;
        // k = p.key
        if (p.hash == hash &&                                                     
            ((k = p.key) == key || (key != null && key.equals(k))))
            // hash 相等并且 key 相等
            e = p;
        // 当前 key 不在头部
        else if (p instanceof TreeNode)
            // 当前数组节点对应为红黑树
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);       
        else {
            // 默认为链表
            for (int binCount = 0; ; ++binCount) {                                
                if ((e = p.next) == null) {
                    // 已经遍历到尾部，仍然没有不存在 key，新建一个链表节点
                    p.next = newNode(hash, key, value, null);
                    // 如果节点的节点大于等于 8,转化为红黑树
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st          
                        treeifyBin(tab, hash);                                    
                    break;                                                        
                }                                                                 
                if (e.hash == hash &&                                             
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    // 找到链表中的 key
                    break;                                                        
                p = e;                                                            
            }                                                                     
        }                                                                         
        if (e != null) { // existing mapping for key                              
            V oldValue = e.value;                                                 
            if (!onlyIfAbsent || oldValue == null)
                // 保存值
                e.value = value;                                                  
            afterNodeAccess(e);
            // 不涉及结构的修改，直接返回
            return oldValue;                                                      
        }                                                                         
    }
    // 涉及结构修改
    // 自增 modCount
    ++modCount;
    // 检查是否需要扩容
    if (++size > threshold)                                                       
        resize();                                                                 
    afterNodeInsertion(evict);                                                    
    return null;                                                                  
}                                                                                 
```

#### 扩容机制

默认扩容增量是扩为原来的 2 倍，那么在 `length - 1` 实际上就是在高位上多了 1，所以新的索引位置要么是在原位置，要么是移动 2次幂。

``` java
final Node<K,V>[] resize() {                                                  
    Node<K,V>[] oldTab = table;                                               
    int oldCap = (oldTab == null) ? 0 : oldTab.length;                        
    int oldThr = threshold;                                                   
    int newCap, newThr = 0;                                                   
    if (oldCap > 0) {                                                         
        if (oldCap >= MAXIMUM_CAPACITY) {
            // MAXMUM_CAPACITY 限制
            threshold = Integer.MAX_VALUE;                                    
            return oldTab;                                                    
        }                                                                     
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&                 
                 oldCap >= DEFAULT_INITIAL_CAPACITY)                          
            newThr = oldThr << 1; // 双倍                        
    }                                                                         
    else if (oldThr > 0) // 使用 threshold 作为 capacity          
        newCap = oldThr;                                                      
    else {               // 使用默认值   
        newCap = DEFAULT_INITIAL_CAPACITY; //16                                    
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY); // 16 * 0.75       
    }                                                                         
    if (newThr == 0) {
        // 计算 threashold
        float ft = (float)newCap * loadFactor;                                
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ? 
                  (int)ft : Integer.MAX_VALUE);                               
    }                                                                         
    threshold = newThr;                                                       
    @SuppressWarnings({"rawtypes","unchecked"})                               
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    // 使用新的哈希桶数组
    table = newTab;                                                           
    if (oldTab != null) {                                                     
        for (int j = 0; j < oldCap; ++j) {
            // 遍历
            Node<K,V> e;                                                      
            if ((e = oldTab[j]) != null) {                                    
                oldTab[j] = null;                                             
                if (e.next == null)
                    // 如果当前节点只有头部
                    newTab[e.hash & (newCap - 1)] = e;                        
                else if (e instanceof TreeNode)
                    // 如果当前节点为红黑树
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);        
                else { // 保留顺序
                    // 链表处理
                    Node<K,V> loHead = null, loTail = null;                   
                    Node<K,V> hiHead = null, hiTail = null;                   
                    Node<K,V> next;                                           
                    do {                                                      
                        next = e.next;
                        // 取最高位是否为 1，当大于 0 时，表示在扩容后，索引会发生变化，等于 oldIndex + oldCap
                        if ((e.hash & oldCap) == 0) {                         
                            if (loTail == null)                               
                                loHead = e;                                   
                            else                                              
                                loTail.next = e;                              
                            loTail = e;                                       
                        }                                                     
                        else {                                                
                            if (hiTail == null)                               
                                hiHead = e;                                   
                            else                                              
                                hiTail.next = e;                              
                            hiTail = e;                                       
                        }                                                     
                    } while ((e = next) != null);                             
                    if (loTail != null) {                                     
                        loTail.next = null;                                   
                        newTab[j] = loHead;                                   
                    }                                                         
                    if (hiTail != null) {                                     
                        hiTail.next = null;
                        // j + oldCap 不会被占用，所以放心存储进去
                        newTab[j + oldCap] = hiHead;                          
                    }                                                         
                }                                                             
            }                                                                 
        }                                                                     
    }                                                                         
    return newTab;                                                            
}                                                                             
```



### 小结

1. 扩容是个特别消耗性能的操作，应该给个合适的 initialCapacity，避免频繁扩容。
2. load tractor 可以修改，但 0.75 是个非常合理的值，一般情况下，不修改。
3. HashMap 非线程安全，不能并发修改。
4. JDK 1.8 引入的红黑树大大提升了性能。 

