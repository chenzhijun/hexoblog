---
title: JDK-HashMap解析
date: 2018-02-06 09:49:08
tags:
    - Java
    - JDK
categories: Java
---

# JDK-HashMap解析

## jdk 类介绍

HashMap 是一个继承Map接口并且是使用哈希表的实现。提供了所有的map操作，并且允许null建和null值。除了非同步方式和允许null键值，它大致上可以等同于`Hashtable`。影响HashMap表现的有两个原因：初始容量和加载因子。

capacity：容量指的是hash表中的总容量。初始容量在第一次创建hashmap的时候就已经创建好了。
load factor: 加载因子，是测量capacity在hash表达到了多满的时候可以自动增长。

当键值对达到了加载因子的量并且超过了当前的capacity，hash表就会重新进行hash（意思是内部数据结构将重建），目的是为了将hash表能够达到差不多两倍的容量。

通常的规则是，默认的加载因子为0.75，它可以在时间和空间上做一个非常好的平衡。如果设置自定义capacity，map里面可能存在多少键值队就需要作为一个考虑因素，以达到最小的rehash操作。如果初始capacity在设置capacity之后还能存下更多的键值对，那么就不会出现rehash操作。

如果事先能大致确认map的容量是多少，那么给map定一个初始化大小是非常可行的，会比让它自己增长更有效率。在hash表中如果出现太多的相同的key拥有相同的hashCode()会降低效率，如果需要改善这种关系，如果键是可以比较的，类可以使用key的比较顺序来进行改善。

HashMap的实现是非线程安全的。在多线程的环境下，必须在外部上进行同步控制。如果外部没有控制，那么需要将map进行包装，也就是使用Collections.synchronizedMap(map)方法，最好是在一开始创建map的时候就进行包装。
map的iterators方法是fail-fast的。在iterator创建之后，如果map的结构被修改了，除了iterator的remove方法，iterator会抛出ConcurrentModificationException异常。所以在多线程环境下进行修改，iterator会快速的返回失败。不过也得注意在非同步环境下，迭代器的fail-fast不能得到保证。迭代器抛出这个异常一个非常好的功能，但是不应该依赖它来写程序，如果出现异常那么就应该说明程序有bug。

jdk的源码里面，介绍的非常详细。真的很详细。

## 分块介绍

之前说过map是非线程安全的，这节就单纯的聊一下非线程安全下的map。

一个类我们从哪里来看它了？其实我也不知道，那就干脆从我们使用一个对象开始。我们是先创建对象，给对象设置值，然后使用对象的方法。

### 创建map

通常在创建map的时候我们会使用对象的构造方法，如果有对象有属性，可能还会有带参数的构造方法。所以在创建map之前我们先了解下map有哪些属性？

通过idea我截取了一个图，在jdk8中是

![2018-02-06-14-54-11](/images/qiniu/2018-02-06-14-54-11.png)

在jdk7中：
![2018-02-06-15-00-27](/images/qiniu/2018-02-06-15-00-27.png)

可以看到，map对象里面有一些属性，根据这些属性的定义我们可以猜到大致的意思。

`DEFAULT_INITIAL_CAPCITY = 1<<4`：默认容量大小，1左移4位，16。记住默认的大小必须是2的幂。
`MAXIMUM_CAPACITY = 1<<30`：最大的容量
`DEFAULT_LOAD_FACTOR`:加载因子，默认0.75
`Entry<?,?>[] EMPTY_TABLE`： 空表
`Entry<K,V>[] table`：长度为2的幂方，并且必要的时候可以改变大小，默认为空表。注意到这个字段是`transient`
`size`: 实际map中键值对的数量
`threshold`：下一次表重构的极限值
`modCount`：hash表结构修改一次就增加一次，包括增加减少键值对，内部数据结构重构。主要用来在迭代器中fail-fast

大致的参数就这些了。

然后我们看到构造方法，如下图：

![2018-02-06-15-22-02](/images/qiniu/2018-02-06-15-22-02.png)

```java
    public HashMap(int initialCapacity) {
        this(initialCapacity, DEFAULT_LOAD_FACTOR);
    }

    /**
     * Constructs an empty <tt>HashMap</tt> with the default initial capacity
     * (16) and the default load factor (0.75).
     */
    public HashMap() {
        this(DEFAULT_INITIAL_CAPACITY, DEFAULT_LOAD_FACTOR);
    }
```

如果你看构造函数带有Map的构造方法：

```java
    public HashMap(Map<? extends K, ? extends V> m) {
        this(Math.max((int) (m.size() / DEFAULT_LOAD_FACTOR) + 1,
                      DEFAULT_INITIAL_CAPACITY), DEFAULT_LOAD_FACTOR);
        inflateTable(threshold);

        putAllForCreate(m);
    }
```
你会发现，同样的它也在里面调用带两个参数的构造方法，所以我认为，这个带两个参数的构造方法可以算作是基本方法，其实想想这算不算代码复用？如果我们可以在平常的开发中开发出了基础方法，那么是否也可以像这样进行包装使用？

这三个构造方法其实只是包装了一层而已，当然map构造方法进行了一些其它的操作，所以我们需要再看下里面的基础构造方法:

```java
    public HashMap(int initialCapacity, float loadFactor) {
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal initial capacity: " +
                                               initialCapacity);
        if (initialCapacity > MAXIMUM_CAPACITY)
            initialCapacity = MAXIMUM_CAPACITY;
        if (loadFactor <= 0 || Float.isNaN(loadFactor))
            throw new IllegalArgumentException("Illegal load factor: " +
                                               loadFactor);

        this.loadFactor = loadFactor;
        threshold = initialCapacity;
        init();
    }
```

这里可以看到，我们的map里面，在使用的时候至少需要进行两个属性设置：`initialCapacity`初始容量和`loadFactor`加载因子。

可以看到代码的严谨性：在初始化的时候进行了参数可用性校验，平常我写方法的时候，偶尔会忘记这个，不过最后我会根据sonar扫描一次代码提交。哈哈sonar这点还是可以做到的。

在这个构造方法里面init()是一个空方法。

在map的构造方法里面后面还有两个操作，一个是inflateTable，一个是putAllForCreate。其实可以想到，用map的构造方法是有数据的，不同于其它三个，其它三个都是无数据的。所以我们只需要构建一个map出来就好了。而map的构造方法拥有数据所以在构建map之后我们需要将数据填充到新的map中。一个是扩容的代码：

```java
    private static int roundUpToPowerOf2(int number) {
        // assert number >= 0 : "number must be non-negative";
        return number >= MAXIMUM_CAPACITY
                ? MAXIMUM_CAPACITY
                : (number > 1) ? Integer.highestOneBit((number - 1) << 1) : 1;
    }

    /**
     * Inflates the table.
     */
    private void inflateTable(int toSize) {
        // Find a power of 2 >= toSize
        int capacity = roundUpToPowerOf2(toSize);

        threshold = (int) Math.min(capacity * loadFactor, MAXIMUM_CAPACITY + 1);
        table = new Entry[capacity];
        initHashSeedAsNeeded(capacity);
    }
```

之后的设置初始值就不详细说了。

### 常用方法

#### 常用方法put(key,value)

这个方法是从Map接口中继承过来的，我们大致猜测一下，map如果插入了一个键值对，插入之前我们是不是需要先判断一下容量？是不是还需要有个地方存入这个值？在哪里存这个键值对？：

```java
   public V put(K key, V value) {
       //对表容量进行判断
        if (table == EMPTY_TABLE) {
            inflateTable(threshold);
        }
        //key的空值处理
        if (key == null)
            return putForNullKey(value);
        //对key进行hash计算
        int hash = hash(key);
        //hash表中的位置i
        int i = indexFor(hash, table.length);
        //在i的位置找到值，
        for (Entry<K,V> e = table[i]; e != null; e = e.next) {
            
            Object k;
            //条件1：判断当前key的hash与i位置的hash是否一致
            //条件2：判断key是否是i位置(entry)的key相等的
            //条件3：判断key是否和i位置(entry)的key内容相等
            //条件1&&（条件2||条件3）
            if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
                //说明找到了hash的key，将新值替换旧值
                V oldValue = e.value;
                e.value = value;
                e.recordAccess(this);
                return oldValue;
            }
        }

        //map数据结构修改，+1
        modCount++;
        //没有在i的位置找到值，说明这个是新的key，加入到entry
        addEntry(hash, key, value, i);
        return null;
    }
```

我们再代码中就可以看到之前的问题了，不过代码实现中还做了null值的处理。下面我们一个个来看：

扩容处理：inflateTable(threshold); 这个在上面构造map的时候讲过了。 

空值处理：putForNullKey(value);

```java
    private V putForNullKey(V value) {
        //从table的0位置开始遍历
        for (Entry<K,V> e = table[0]; e != null; e = e.next) {
            //找到null的key，替换值
            if (e.key == null) {
                V oldValue = e.value;
                e.value = value;
                e.recordAccess(this);
                return oldValue;
            }
        }
        modCount++;
        addEntry(0, null, value, 0);
        return null;
    }
```

可以看到，这里的的操作是从0位置开始查找的。也就是从一开始遍历，这种情况下可以想到一个O(n)的时间复杂度。所以，如果是1.7，null键最好不要存在。

在代码中可以看到存键值对的地方是Entry，关于Entry我们现在只需要知道它是一个单链表的数据结构就好了，之后我们专门讲解一下Entry和hash。

讲完1.7。我们看看1.8中的put方法：

```java
    public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
    }

    final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        //判断是否是空表
        if ((tab = table) == null || (n = tab.length) == 0)
            //空表，重构表
            n = (tab = resize()).length;
        //如果原始表中不存在数据，直接新建一个节点，p为表中key的数据节点
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        else {
            //原始表中存在数据
            Node<K,V> e; K k;
            //找到了key节点hash的值
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            // 如果该节点是一个TreeNode
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
            //如果节点存在，修改节点里面的值
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        //增加修改次数
        ++modCount;
        //size+1之后的大小大于极限值
        if (++size > threshold)
            resize();
        //空方法
        afterNodeInsertion(evict);
        return null;
    }
```

jdk1.8中的put方法，实现好像有些优化。加入了TreeNode，之后这块我补上，整体的流程是和1.7一致的。

#### 常用方法get(key)

该方法在1.7中的实现是：
```java

    public V get(Object key) {
        //判断null，如果是null遍历循环找出值
        if (key == null)
            return getForNullKey();
        //找到存了key的entry
        Entry<K,V> entry = getEntry(key);

        return null == entry ? null : entry.getValue();
    }

    final Entry<K,V> getEntry(Object key) {
        //如果是空表，直接返回null
        if (size == 0) {
            return null;
        }

        // 计算hash值
        int hash = (key == null) ? 0 : hash(key);
        //循环遍历找到值，从hash值开始找
        for (Entry<K,V> e = table[indexFor(hash, table.length)];
             e != null;
             e = e.next) {
            Object k;
            if (e.hash == hash &&
                ((k = e.key) == key || (key != null && key.equals(k))))
                return e;
        }
        return null;
    }

```

1.7中还是很好理解的，这里主要是hash值计算，之后我们重点讲解。

在1.8中，get方法：

```java

    public V get(Object key) {
        Node<K,V> e;
        return (e = getNode(hash(key), key)) == null ? null : e.value;
    }

    final Node<K,V> getNode(int hash, Object key) {
        Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
        //表不为空
        //表的内容长度不为0
        //在(n-1)&hash的位置不为null
        if ((tab = table) != null && (n = tab.length) > 0 &&
            (first = tab[(n - 1) & hash]) != null) {
            // 判断第一个位置?为什么判断第一个位置，是和node的数据结构有关系么？是不是加入的时候都是加入第一个位置？
            if (first.hash == hash && // always check first node
                ((k = first.key) == key || (key != null && key.equals(k))))
                return first;
            if ((e = first.next) != null) {
                if (first instanceof TreeNode)
                    return ((TreeNode<K,V>)first).getTreeNode(hash, key);
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

注意到8中总是计算判断第一个node，为什么？之后了解清楚了再来详细解析

#### 常用方法containsKey(Object),containsValue(Object)

containsKey(Object)的源码中我们可以看到也是调用了getEntry方法：

```java
    public boolean containsKey(Object key) {
        return getEntry(key) != null;
    }
```

containsValue(Object)的源码中我们可以看到，基本类似，也是循环链表，然后获取到值之后比较value：

```java
    public boolean containsValue(Object value) {
        if (value == null)
            return containsNullValue();

        Entry[] tab = table;
        for (int i = 0; i < tab.length ; i++)
            for (Entry e = tab[i] ; e != null ; e = e.next)
                if (value.equals(e.value))
                    return true;
        return false;
    }
```


#### 常用方法keySet(),values(),entrySet()

打开源码看看keySet()，之前我一直就是用它来获取key，然后遍历map：

```java
    public Set<K> keySet() {
        Set<K> ks = keySet;
        if (ks == null) {
            ks = new KeySet();
            keySet = ks;
        }
        return ks;
    }

    final class KeySet extends AbstractSet<K> {
        public final int size()                 { return size; }
        public final void clear()               { HashMap.this.clear(); }
        public final Iterator<K> iterator()     { return new KeyIterator(); }
        public final boolean contains(Object o) { return containsKey(o); }
        public final boolean remove(Object key) {
            return removeNode(hash(key), key, null, false, true) != null;
        }
        public final Spliterator<K> spliterator() {
            return new KeySpliterator<>(HashMap.this, 0, -1, 0, 0);
        }
        public final void forEach(Consumer<? super K> action) {
            Node<K,V>[] tab;
            if (action == null)
                throw new NullPointerException();
            if (size > 0 && (tab = table) != null) {
                int mc = modCount;
                for (int i = 0; i < tab.length; ++i) {
                    for (Node<K,V> e = tab[i]; e != null; e = e.next)
                        action.accept(e.key);
                }
                if (modCount != mc)
                    throw new ConcurrentModificationException();
            }
        }
    }
```

可以看到，如果ks为null，那么

#### size()

#### Entry，hash，hashCode（）




