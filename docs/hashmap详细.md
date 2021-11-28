# HashMap详细
---

[原文地址](https://blog.csdn.net/m0_46494737/article/details/114499465)

## 一、HashMap基础

### 1.1 数组优缺点

**优点**

* 按照索引查询元素速度很快
* 能存储大量数据
* 按照索引遍历数据方便

**缺点**

* 根据内容查询元素速度慢
* 数组的大小一旦给定就不能改变
* 数据只能存储一种类型的数据
* 增加、删除元素效率慢

### 1.2 链表的优缺点

**优点**

* 插入、删除速度快
* 内容利用率高，不会浪费内存
* 大小不固定，扩展性灵活

**缺点**

* 不支持随机查找，必须从第一个开始遍历，查找效率低
* 链表中存储元素需要更多的内存，因为边聊中每个结点都包含一个指针，需要额外的内存空间

### 1.3 散列表

也叫哈希表（Hash table），根据关键码值（key value）而直接进行访问的数据结构。即通过键值映射到表中一个位置来访问记录，以加快查找速度，这个映射函数叫`散列函数`，存放记录的数组叫做`散列表`

**特点**

* 访问速度很快：由于散列表有散列函数，可以将指定的key都映射到一个地址上，所以根据key来查询value很快
* 需要额外空间：散列表实际上是存不满的，且当散列表中元素使用率越来越高时，性能会下降，所以需要通过扩容来解决这个问题。另外由于存在哈希冲突，所以需要通过如链地址发来处理
* 无序：散列表是通过hash函数直接找到存储地址的
* 哈希冲突：

### 1.4 什么是哈希？特点是什么？

**核心理论**：Hash也称散列、哈希。基本原理就是把任意长度的输入，通过Hash算法变成固定长度的输出，这样的规则就是对应的Hash算法，而原始数据映射后的二进制串就是哈希值

**特点**

* 从Hash值不可反向推导出原始数据
* 输入数据的微笑变化会得到完全不同的hash值，相同的数据会得到相同的值
* 哈希算法的执行效率要高效，唱的文本也能快速计算出哈希值
* hash算法的冲突概率要小



## 二、HashMap原理讲解

### 2.1 HashMap的继承体系



### 2.2 Node数据结构分析

Node类是HashMap的一个静态内部类

```java
static class Node<K,V> implements Map.Entry<K,V> {
    final int hash;
    final K key;
    V value;
    Node<K,V> next;

    Node(int hash, K key, V value, Node<K,V> next) {
        this.hash = hash;
        this.key = key;
        this.value = value;
        this.next = next;
    }

    public final K getKey()        { return key; }
    public final V getValue()      { return value; }
    public final String toString() { return key + "=" + value; }

    public final int hashCode() {
        return Objects.hashCode(key) ^ Objects.hashCode(value);
    }

    public final V setValue(V newValue) {
        V oldValue = value;
        value = newValue;
        return oldValue;
    }
    // 判断两个node是否相等，若key和value都相等，返回true
    public final boolean equals(Object o) {
        if (o == this)
            return true;
        if (o instanceof Map.Entry) {
            Map.Entry<?,?> e = (Map.Entry<?,?>)o;
            if (Objects.equals(key, e.getKey()) &&
                Objects.equals(value, e.getValue()))
                return true;
        }
        return false;
    }
}
```

### 2.3 底层存储结构介绍



### 2.4 put数据原理分析


### 2.5 HashMap在JDK 8中为什么引入红黑树

`在链表长度大于8且桶数量大于64时变为红黑树，以加快检索速度`

红黑树的插入、删除、查找各种操作性能都比较稳定



### 2.6 HashMap扩容原理

使用一个新的数组代替原有的数组，对原数组的所有数据进行重新计算插入新数组，之后指向新数组。如果扩容器数组以及达到最大了，那么将直接将阈值设置成最大整形并返回。扩容是为了提高查询效率



### 2.7 HashMap构造方法源码分析

```java
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4;//16
//默认table大小为16
static final int MAXIMUM_CAPACITY = 1 << 30;//1073741824
//table最大长度：1 073 741 824
/*
左移的运算规则：按二进制形式把所有的数字向左移动对应的位数，高位移出（舍弃），低位的空位补零。

计算1<<30,首先把1转为二进制数字 0000 0000 0000 0000 0000 0000 0000 0001

然后将上面的二进制数字向左移动30位后面补0得到 0010 0000 0000 0000 0000 0000 0000 0000
二进制再转化为十进制得到1 073 741 824
*/
static final float DEFAULT_LOAD_FACTOR = 0.75f;
//默认负载因子大小：0.75
static final int TREEIFY_THRESHOLD = 8;
//树化阈值
static final int UNTREEIFY_THRESHOLD = 6;
//树降级成为链表的阈值
static final int MIN_TREEIFY_CAPACITY = 64;
//树化的另一个参数：数组长度达到64时（桶的数量）

transient int size;//当前哈希表种元素个数

transient int modCount;//当前哈希表结构修改次数

int threshold;//扩容阈值，当你的哈希表中的元素超过阈值时，就会触发扩容

final float loadFactor;//负载因子  threshold = capacity * loadFactor
//在jdk1.8版本中HashMap有四个构造方法，根据参数内容我们可以发现实质上就是一个套娃

	public HashMap(int initialCapacity, float loadFactor) {
        //做了一些逻辑校验，capacity必须大于0&&最大值不可以超过MAXIMUM_CAPACITY
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal initial capacity: " +
                                               initialCapacity);
        if (initialCapacity > MAXIMUM_CAPACITY)
            initialCapacity = MAXIMUM_CAPACITY;
        //loadFactor（负载因子）必须大于零&&必须是个数不能是NaN
        if (loadFactor <= 0 || Float.isNaN(loadFactor))
            throw new IllegalArgumentException("Illegal load factor: " +
                                               loadFactor);
        this.loadFactor = loadFactor;//初始化填充因子
        this.threshold = tableSizeFor(initialCapacity);
        /*分析tableSizeFor方法的源码可以知道（有兴趣的小伙伴可以自己试着分析下，里面主要运用到了移位运算符），该方法的返回值必须是一个大于等于当前参数的一个数字并且这个数字一定是2的次方数(这样的数有助于提高hash函数的执行效率)，如传入7会返回8(2的三次方)，传入9会返回16(2的4次方)。*/
    
    }
	public HashMap(int initialCapacity) {
        this(initialCapacity, DEFAULT_LOAD_FACTOR);
    }
	public HashMap() {
        this.loadFactor = DEFAULT_LOAD_FACTOR; // 所有其他字段均为默认值
    }
	public HashMap(Map<? extends K, ? extends V> m) {
        this.loadFactor = DEFAULT_LOAD_FACTOR;
        putMapEntries(m, false);
    }
    //putMapEntries(Map<? extends K, ? extends V> m, boolean evict)函数将m的所有元素存入本HashMap实例中。

```

### 2.8 HashMap put方法源码分析

```java
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}
static final int hash(Object key) {
    /*扰动函数：让key的hash值得高16为也彩玉路由运算。作用：减少哈希冲突的概率*/
    // 简单来说：让高位参与运算后，再 & length-1让index更加均匀散列，这样能减少冲突
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; //引用当前hashMap的散列表
        Node<K,V> p; //表示当前散列表的元素
        int n, i;//n表示散列表数组的长度，i表示路由寻址 结果
        if ((tab = table) == null || (n = tab.length) == 0)//延迟初始化逻辑，第一次调用putVal时会初始化hashMap对象中的最耗费内存的散列表
            n = (tab = resize()).length;
        if ((p = tab[i = (n - 1) & hash]) == null)//最简单的一种情况：寻址找到的桶位刚好是null，这个时候，直接将当前的node扔进去就行了。
            tab[i] = newNode(hash, key, value, null);
        else {
            //e:不为null的话，找到一个与当前要插入的key-value一致的key的元素，k：表示临时的一个key
            Node<K,V> e; K k;
            //表示桶位中的该元素，与你当前插入的元素的key完全一致，后续需要替换操作
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            else if (p instanceof TreeNode)//红黑树呗
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {//链表的情况，而且链表的头元素与我们要插入的key不一致。（尾插法）
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);//直接插
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);//前面有8个元素，要树化了~
                        break;
                    }
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    //条件成立的话，说明找到了相同key的node元素，需要进行替换操作
                    p = e;
                }
            }
            if (e != null) {
         // e！=null，条件成立说明。找到了一个与你插入元素key完全一致的数据，需要进行替换。
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;//表示散列表结构被修改的次数，替换node元素的value不计数
        if (++size > threshold)//假如容量不够，就扩容
            resize();
        afterNodeInsertion(evict);
        return null;
    }
```

**流程图**


### 2.9 HashMap resize扩容方法源码分析

扩容目的：解决哈希冲突导致链化影响查询

```java
final Node<K,V>[] resize() {
        Node<K,V>[] oldTab = table;
    //oldTab：引用扩容前的哈希表
        int oldCap = (oldTab == null) ? 0 : oldTab.length;
         //oldCap表示扩容之前table数组的长度
        int oldThr = threshold;//表示扩容之前的扩容阈值，触发本此扩容的阈值
        int newCap, newThr = 0;//newCap表示扩容之后table数组的小大
        //newThr：扩容之后，下次再次触发扩容的条件
    
        //条件如果成立说明 hashmap中的散列表已经初始化过了，是一次正常的扩容
        if (oldCap > 0) {
            if (oldCap >= MAXIMUM_CAPACITY) {
                //比最大的长度还大，无法扩容
                threshold = Integer.MAX_VALUE;
                return oldTab;
            }
            //newCap = oldCap << 1  左移一位实现数值翻倍，并且赋值给newCap，newCap小于数组最大值限制且扩容之前的阈值>=16，这种情况下，则下一次扩容的阈值等于当前阈值翻倍
            else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                     oldCap >= DEFAULT_INITIAL_CAPACITY)
                newThr = oldThr << 1; //阈值跟着翻倍
        }
    //oldCap==0，说明hashmap中的散列表是null
        else if (oldThr > 0) 
            newCap = oldThr;
    //oldCap==0
        else {               
            newCap = DEFAULT_INITIAL_CAPACITY;
            newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
        }
    //newThr为零时，通过newCap和loadFactor计算出一个newThr
        if (newThr == 0) {
            float ft = (float)newCap * loadFactor;
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                      (int)ft : Integer.MAX_VALUE);
        }
    
        threshold = newThr;//下一次触发的阈值
        //创建出一个更大更长的数组
        @SuppressWarnings({"rawtypes","unchecked"})
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
        table = newTab;
    //说明，hashmap本此扩容之前，table不为null
        if (oldTab != null) {
            for (int j = 0; j < oldCap; ++j) {
                Node<K,V> e;//当前node节点
                if ((e = oldTab[j]) != null) {
                    //说明当前桶位中有数据，但是数据具体是单个数据，链表，红黑树，还不知道。
                    oldTab[j] = null;//置空，方便JVM GC回收内存
                    if (e.next == null)//第一种情况：当前桶内只有一个元素，从未发生过碰撞，这种情况直接计算出当前元素应该存放在新数组中的位置，然后扔进去就可以了
                        newTab[e.hash & (newCap - 1)] = e;
                    else if (e instanceof TreeNode)//当前节点已经树化
                        ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                    else { // 链表情况
                        
 //低位链表：存放在扩容之后的数组的下标位置与当前数组的下标位置一致
 //高位链表：存放在扩容之后的数组的下标位置为当前数组下标位置+扩容之前的数组长度
 //举个例子：当前容量为16 则扩容后对应的容量为32，而原本15号位置（最后一个，因为从0开始）的链表中的数据在新哈希表中可能位于15(低位链表),可能位于31(15+16高位链表)。具体用到的还是位运算符，有兴趣的读者可以自己研究一下。
                        Node<K,V> loHead = null, loTail = null;
                        Node<K,V> hiHead = null, hiTail = null;
                        Node<K,V> next;
                        do {
                            next = e.next;
//oldCap高位一定是1，这里依据hash高位值来判断调用哪个语句，假如是1，就调用else  是0 就调用if（这里需要想一下，有点不好理解）
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
                        //
                        if (loTail != null) {
                            loTail.next = null;//设置为null是因为低位的下一个是高位，需要设置位null
                            newTab[j] = loHead;
                        }
                        if (hiTail != null) {
                            hiTail.next = null;
                            newTab[j + oldCap] = hiHead;
                        }
                    }
                }
            }
        }
        return newTab;
    }
```



### 2.10 HashMa get方法

```java
public V get(Object key) {
        Node<K,V> e;
        return (e = getNode(hash(key), key)) == null ? null : e.value;
    }
final Node<K,V> getNode(int hash, Object key) {
    
        Node<K,V>[] tab; //tab：引用当前hashMap的散列表
        Node<K,V> first, e;//first：桶位中的头元素 e是一个临时node元素
        int n; K k;//n：table数组长度
    
        if ((tab = table) != null && (n = tab.length) > 0 &&
            (first = tab[(n - 1) & hash]) != null) {//有数据
            if (first.hash == hash && // 第一种情况：定位出来的桶位元素 即为我们需要get的数据
                ((k = first.key) == key || (key != null && key.equals(k))))
                return first;
            //说明当前桶位不止一个元素，可能是链表也可能是红黑树
            if ((e = first.next) != null) {
                if (first instanceof TreeNode)//情况二：升级成了红黑树
                    return ((TreeNode<K,V>)first).getTreeNode(hash, key);
                //情况三：桶位形成链表，一直往后找，找到对应值即可
                do {
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        return e;
                } while ((e = e.next) != null);
            }
        }
        return null;
    }
```



### 2.11 HashMap remove方法

```java
final Node<K,V> removeNode(int hash, Object key, Object value,
                               boolean matchValue, boolean movable) {
     //matchValue：仅在value，key均匹配才删除
        Node<K,V>[] tab; //tab：引用当前hashmap中的散列表
        Node<K,V> p; //p：当前node元素
        int n, index;//n:表示散列表数组长度  index表示寻址结果
     
        if ((tab = table) != null && (n = tab.length) > 0 &&
            (p = tab[index = (n - 1) & hash]) != null) {
            //有数据才删除，没数据就呵呵
            //p是指向某个桶位
            Node<K,V> node = null, e; //node表示查找到的结果 e表示当前node的下一个元素
            K k; V v;
            //第一种情况：当前桶位中的元素 即为 你要删除的元素
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                node = p;
            //第二种情况：当前桶位后面有链表或者红黑树
            else if ((e = p.next) != null) {
                if (p instanceof TreeNode)//是红黑树
                    node = ((TreeNode<K,V>)p).getTreeNode(hash, key);
                else {
                    //链表的情况
                    do {
                        if (e.hash == hash &&
                            ((k = e.key) == key ||
                             (key != null && key.equals(k)))) {
                            node = e;
                            break;
                        }
                        p = e;
                    } while ((e = e.next) != null);
                }
            }
            //删除相关逻辑
            //判断node不为空的话，说明按照key查找到需要删除的数据了
            if (node != null && (!matchValue || (v = node.value) == value ||
                                 (value != null && value.equals(v)))) {
                if (node instanceof TreeNode)//第一种情况：树的删除逻辑
                    ((TreeNode<K,V>)node).removeTreeNode(this, tab, movable);
                else if (node == p)//第二种情况：当前桶位元素即为删除元素
                    tab[index] = node.next;
                else//第三种情况：链表中的删除，不懂的小伙伴自己复习下链表的删除
                    p.next = node.next;
                ++modCount;
                --size;
                afterNodeRemoval(node);
                return node;
            }
        }
        return null;
    }
```

