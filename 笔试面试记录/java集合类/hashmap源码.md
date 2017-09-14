### HashMap源码

> 本文大部分内容参考自：
>
> http://blog.csdn.net/fjse51/article/details/53811465

#### 一、HashMap概述

本文所讲的源码版本是JDK1.8.0_25。在JDK1.6中，HashMap采用位桶+链表实现，即使用链表处理冲突，同一hash值的链表都存储在一个链表里。但是当位于一个桶中的元素较多，即hash值相等的元素较多时，通过key值依次查找的效率较低。而JDK1.8中，HashMap采用位桶+链表+红黑树实现，当链表长度超过阈值（8）时，将链表转换为红黑树，这样大大减少了查找时间。

#### 二、HashMap的数据结构

HashMap是由数组和链表结合实现的，其实是一个Entry数组

```
//Node是单向链表，它实现了Map.Entry接口  
static class Node<K,V> implements Map.Entry<K,V> {  
    final int hash;  
    final K key;  
    V value;  
    Node<K,V> next;  
    //构造函数Hash值 键 值 下一个节点  
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
    //判断两个node是否相等,若key和value都相等，返回true。可以与自身比较为true  
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

```

```
//红黑树  
static final class TreeNode<K,V> extends LinkedHashMap.Entry<K,V> {  
    TreeNode<K,V> parent;  // 父节点  
    TreeNode<K,V> left;   //左子树  
    TreeNode<K,V> right;//右子树  
    TreeNode<K,V> prev;    // needed to unlink next upon deletion  
    boolean red;    //颜色属性  
    TreeNode(int hash, K key, V val, Node<K,V> next) {  
        super(hash, key, val, next);  
    }  
  
    //返回当前节点的根节点  
    final TreeNode<K,V> root() {  
        for (TreeNode<K,V> r = this, p;;) {  
            if ((p = r.parent) == null)  
                return r;  
            r = p;  
        }  
    }  
```

```
transient Node<K,V>[] table;//存储（位桶）的数组  
```

有了以上3个数据结构，只要有一点数据结构基础的人，都可以大致联想到HashMap的实现了。首先有一个每个元素都是链表（可能表述不准确）的数组，当添加一个元素（key-value）时，就首先计算元素key的hash值，以此确定插入数组中的位置，但是可能存在同一hash值的元素已经被放在数组同一位置了，这时就添加到同一hash值的元素的后面，他们在数组的同一位置，但是形成了链表，所以说数组存放的是链表。而当链表长度太长时，链表就转换为红黑树，这样大大提高了查找的效率。

#### 三、HashMap源码分析

##### 3.1、关键属性

```java
public class HashMap<K,V> extends AbstractMap<K,V> implements Map<K,V>, Cloneable, Serializable {  
    private static final long serialVersionUID = 362498820763181265L;  
    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16  
    static final int MAXIMUM_CAPACITY = 1 << 30;//最大容量  
    static final float DEFAULT_LOAD_FACTOR = 0.75f;//填充比  
    //当add一个元素到某个位桶，其链表长度达到8时将链表转换为红黑树  
    static final int TREEIFY_THRESHOLD = 8;  
    static final int UNTREEIFY_THRESHOLD = 6;  
    static final int MIN_TREEIFY_CAPACITY = 64;  
    transient Node<K,V>[] table;//存储元素的数组  
    transient Set<Map.Entry<K,V>> entrySet;  
    transient int size;//存放元素的个数  
    transient int modCount;//被修改的次数fast-fail机制  
    int threshold;//临界值 当实际大小(容量*填充比)超过临界值时，会进行扩容
    final float loadFactor;//填充比（......后面略）  
```

##### 3.2、构造方法

- 构造方法有4种，主要涉及到的参数有，指定初始容量，指定填充比和用来初始化的Map


- 第一种，初始容量和填充比。
- 第二种，初始容量
- 第三种，无参数
- 第四种，传入一个Map

#####3.3、扩容机制

```
//可用来初始化HashMap大小 或重新调整HashMap大小 变为原来2倍大小  
   final Node<K,V>[] resize() {  
       Node<K,V>[] oldTab = table;  
       int oldCap = (oldTab == null) ? 0 : oldTab.length;  
       int oldThr = threshold;  
       int newCap, newThr = 0;  
       if (oldCap > 0) {  
           if (oldCap >= MAXIMUM_CAPACITY) {//超过1>>30大小,无法扩容只能改变 阈值  
               threshold = Integer.MAX_VALUE;  
               return oldTab;  
           }  
           else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&  
                    oldCap >= DEFAULT_INITIAL_CAPACITY)//新的容量为旧的2倍 最小也是16  
               newThr = oldThr << 1; // 扩容阈值加倍  
       }  
       else if (oldThr > 0)   
           newCap = oldThr;//oldCap=0 ,oldThr>0此时newThr=0   
       else {               //oldCap=0,oldThr=0 相当于使用默认填充比和初始容量 初始化  
           newCap = DEFAULT_INITIAL_CAPACITY;  
           newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);  
       }  
         
       if (newThr == 0) {  
           float ft = (float)newCap * loadFactor;  
           newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?  
                     (int)ft : Integer.MAX_VALUE);  
       }  
       threshold = newThr;  
       @SuppressWarnings({"rawtypes","unchecked"})  
           Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];  
       //数组辅助到新的数组中，分红黑树和链表讨论  
       table = newTab;  
       if (oldTab != null) {  
           for (int j = 0; j < oldCap; ++j) {  
               Node<K,V> e;  
               if ((e = oldTab[j]) != null) {  
                   oldTab[j] = null;  
                   if (e.next == null)  
                       newTab[e.hash & (newCap - 1)] = e;  
                   else if (e instanceof TreeNode)  
                       ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);  
                   else { // preserve order  
                       Node<K,V> loHead = null, loTail = null;  
                       Node<K,V> hiHead = null, hiTail = null;  
                       Node<K,V> next;  
                       do {  
                           next = e.next;  
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
                           newTab[j + oldCap] = hiHead;  
                       }  
                   }  
               }  
           }  
       }  
       return newTab;  
   }  
```



#####3.4、计算index值

```java
//JDK1.8
static final int hash(Object key){
    int h;
  	return (key==null)?0:(h=key.hashCode())^(h>>>16);
}
```

- 计算entry在数组中的位置，首先根据key计算出hash值，然后通过hash(key)&(length-1)计算出index。
- 在计算hash(key)的过程中，利用了“扰动函数”，hash值如果直接用key.hashCode()的低(legth-1)位，碰撞可能会很严重，而且如果散列本身做得不好，可能碰撞就会极其严重。
- 这时候“扰动函数”的价值就体现出来了，扰动函数混合原始哈希码的高位和低位，以此来加大低位的随机性。而且混合后的低位参杂了高位的部分特征，这样高位的信息也被变相保留下来。

##### 3.5、保证哈希表容量一直是大于容量的最小2的次方值

```
//这段代码保证HashMap的容量总是2的n次方  
static final int tableSizeFor(int cap) {  
    int n = cap - 1;  
    n |= n >>> 1;  
    n |= n >>> 2;  
    n |= n >>> 4;  
    n |= n >>> 8;  
    n |= n >>> 16;  
    return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;  
} 
```

为什么要保证容量是2的次方呢？

length为2的整数幂保证了length-1最后一位（当然是二进制表示）为1，从而保证了取索引操作 h&（length-1）的最后一位同时有为0和为1的可能性，保证了散列的均匀性。反过来讲，当length为奇数时，length-1最后一位为0，这样与h按位与的最后一位肯定为0，即索引位置肯定是偶数，这样数组的奇数位置全部没有放置元素，浪费了大量空间。

##### 3.6、put方法

- 1、如果键值对数组tab[]为空或者长度为0，则分配内存resize() 。
- 2、根据键值key计算hash值得到插入的数组索引i，如果tab[i]==null，直接新建节点添加，否则转入3
- 3、判断当前数组中处理hash冲突的方式为链表还是红黑树(check第一个节点类型即可),分别处理。

##### 3.7、get方法

- 1、bucket里的第一个节点，直接命中；

- 2、如果有冲突，则通过key.equals(k)去查找对应的entry 

  若为树，则在树中通过key.equals(k)查找，O(logn)； 

  若为链表，则在链表中通过key.equals(k)查找，O(n)。

##### 3.8、Fail-Fast机制

#### 四、面试问题：HashMap扩容机制。

- 1、介绍HashMap的底层数据结构，Node结构组成的链表散列
- 2、介绍HashMap的关键属性：
  - **默认初始容量**（table长度）、**最大容量**（table长度）、**默认填充比(0.75)**
  - **临界值** = HashMap所能容纳的最大数据量的Node(键值对)个数=填充比*容量
  - 当实际存在的键值对数量size超过临界值时就重新resize(扩容)，扩容后的HashMap容量是之前容量的两倍。
  - 当链表长度到达8时变红黑树
  - modCount字段主要用来记录HashMap内部结构发生变化的次数，主要用于迭代的快速失败。强调一点，内部结构发生变化指的是结构发生变化，例如put新键值对，但是某个key对应的value值被覆盖不属于结构变化。

