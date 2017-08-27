### java集合类总结

	>本文内容参考自：http://blog.csdn.net/hguisu/article/details/7644395
	>
	>本文大概按如下思路讲解：
	>
	>全文分Collection和Map两大部分；每部分都包括某个集合类的实现原理+线程安全性+效率+排序等其他一些功能实现。
	>
	>最后总结一下面试中常见的集合类的比较+常见问题。

#### 一、总体结构

##### 1.1父子关系

在[Java](http://lib.csdn.net/base/java)的util包中有两个所有集合的父接口Collection和Map,它们的父子关系：

**+Collection 这个接口extends自 --java.lang.Iterable接口**
 ├+List(接口 代表有序，可重复的集合。列表）
 │├ ArreyList     (Class 数组，随机访问，没有同步，线程不安全)
 │├ Vector        (Class  数组                   同步        线程全)
 │├ LinkedList    (Class  链表   插入删除   没有同步   线程不安全)
 │└ Stack          (Class)
 └+Set（接口 不能含重复的元素。仅接收一次并做内部排序，集）
 │├ HashSet            (Class）
 │├ LinkedHashSet   (Class）
 │└ TreeSet       (Class）

 └+Queue（接口 ）

​    └ Deque(接口)

Queue接口与List、Set同一级别，都是继承了Collection接口。LinkedList实现了Queue接口。**Queue接口窄化了对LinkedList的方法的访问权限**（即在方法中的参数类型如果是Queue时，就完全只能访问Queue接口所定义的方法 了，而不能直接访问 LinkedList的非Queue的方法），以使得只有恰当的方法才可以使用。

双向队列(Deque),是Queue的一个子接口。一般这样用：Deque<String> deque = new LinkedList<String>();

**+Map(接口)**
 ├ +Map(接口 映射集合)
 │ ├ HashMap            (Class 不同步，线程不安全。除了允许使用null 键值之外，与Hashtable大致相同)
 │ ├ Hashtable           (Class 同步   ，线程安全    。不允许使用null 键值)
 │ ├ +SortedMap 接口
 │ │   ├ TreeMap         (Class)
 │ ├ WeakHashMap     (Class)

##### 1.2数组Array

- 效率高，但容量固定且无法动态改变。array还有一个缺点是，无法判断其中实际存有多少元素，length只是告诉我们array的容量。
- Java中有一个Arrays类，专门用来操作array。
  - arrays中拥有一组static函数，
  - equals()：比较两个array是否相等。array拥有相同元素个数，且所有对应元素两两相等。
  - fill()：将值填入array中。
  - sort()：用来对array进行排序。
  - binarySearch()：在排好序的array中寻找元素。
  - System.arraycopy()：array的复制。

#####1.3数组Array和集合的区别

- 数组是大小固定的，并且同一个数组只能存放类型一样的数据（基本类型/引用类型）
- JAVA集合可以[存储](http://www.storworld.com/)和操作数目不固定的一组数据。
- 若程序时不知道究竟需要多少对象，需要在空间不足时自动扩增容量，则需要使用容器类库，array不适用。

#### 二、Map接口：映射

![map](E:\笔记\笔试面试记录\图片\map.jpg)

##### 1.1   Map概括

- Map没有继承Collection接口， Map 提供 key 到 value 的映射，你可以通过“键”查找“值”。一个 Map 中不能包含相同的 key ，每个 key 只能映射一个 value 。 Map 接口提供3种集合的视图，即Map 的内容可以被当作一组 key 集合，一组 value 集合，或者一组 key-value 映射。
- 方法 put(Object key, Object value) 添加一个“值” ( 想要得东西 ) 和与“值”相关联的“键” (key) ( 使用它来查找 ) 。方法get(Objectkey) 返回与给定“键”相关联的“值”。可以用 containsKey() 和 containsValue() 测试 Map 中是否包含某个“键”或“值”。 标准的 Java 类库中包含了几种不同的 Map ： HashMap, TreeMap,LinkedHashMap, WeakHashMap, IdentityHashMap 。它们都有同样的基本接口 Map ，但是行为、效率、排序策略、保存对象的生命周期和判定“键”等价的策略等各不相同。
- Map 同样对每个元素保存一份，但这是基于 " 键"的， Map 也有内置的排序，因而不关心元素添加的顺序。如果添加元素的顺序对你很重要，应该使用 LinkedHashSet 或者 LinkedHashMap.
- 执行效率是 Map 的一个大问题。看看 get() 要做哪些事，就会明白为什么在 ArrayList 中搜索“键”是相当慢的。而这正是 HashMap 提高速度的地方。 HashMap 使用了特殊的值，称为“散列码” (hash code) ，来取代对键的缓慢搜索。“散列码”是“相对唯一”用以代表对象的int值，它是通过将该对象的某些信息进行转换而生成的（在下面总结二：需要的注意的地方有更进一步探讨）。所有 Java 对象都能产生散列码，因为 hashCode() 是定义在基类 Object 中的方法 。 HashMap 就是使用对象的 hashCode() 进行快速查询的。此方法能够显著提高性能。

##### 1.2 Hashtable

- Hashtable继承Map接口，实现一个key-value映射的哈希表。任何非空（non-null）的对象都可作为key或者value。Hashtable是同步的。添加数据使用 put(key, value) ，取出数据使用get(key) ，这两个基本操作的时间开销为常数。
- Hashtable 通过初始化容量 (initial capacity) 和负载因子 (load factor) 两个参数调整性能。通常缺省的 load factor0.75 较好地实现了时间和空间的均衡。增大 load factor 可以节省空间但相应的查找时间将增大，这会影响像get和 put 这样的操作。
- 由于作为 key 的对象将通过计算其散列函数来确定与之对应的 value 的位置，因此任何作为 key 的对象都必须实现 hashCode 方法和 equals 方法。 hashCode 方法和 equals 方法继承自根类 Object ，如果你用自定义的类当作 key 的话，要相当小心，按照散列函数的定义：
  - 如果两个对象相同，即 obj1.equals(obj2)=true ，则它们的 hashCode 必须相同
  - 但如果两个对象不同，则它们的 hashCode 不一定不同
  - 如果两个不同对象的 hashCode 相同，这种现象称为冲突，冲突会导致操作哈希表的时间开销增大，所以尽量定义好的 hashCode() 方法，能加快哈希表的操作。
  - 如果相同的对象有不同的 hashCode ，对哈希表的操作会出现意想不到的结果（期待的 get 方法返回null），要避免这种问题，只需要牢记一条：要同时复写 equals 方法和 hashCode 方法，而不要只写其中一个。

##### 1.3HashMap类

- HashMap和Hashtable类似，也是基于hash散列表的实现。不同之处在于 HashMap是**非同步**的，并且**允许null，即null value和null key。**但是将HashMap视为Collection时 （values()方法可返回Collection），其迭代子操作时间开销和HashMap的容量成比例。因此，如果迭代操作的性能相当重要的话，不要 将HashMap的初始化容量设得过高，或者load factor过低。
- LinkedHashMap 类：类似于 HashMap ，但是迭代遍历它时，取得“键值对”的顺序是其插入次序，或者是最近最少使用 (LRU) 的次序。只比 HashMap 慢一点。而在迭代访问时反而更快，因为它使用链表维护内部次序。

##### 1.4 WeakHashMap类

- WeakHashMap是一种改进的HashMap，它是为解决特殊问题设计的，它对key实行“弱引用”，如果一个key不再被外部所引用，那么该key可以被GC回收。

##### 1.5 TreeMap类

- 基于红黑树数据结构的实现。查看“键”或“键值对”时，它们会被排序 ( 次序由 Comparabel 或 Comparator 决定 ) 。 TreeMap 的特点在于，你得到的结果是经过排序的。 TreeMap 是唯一的带有 subMap() 方法的 Map ，它可以返回一个子树。

##### 1.6 IdentifyHashMap 类

- 使用 == 代替 equals() 对“键”作比较的 hash map 。专为解决特殊问题而设计。

##### 1.7 ConcurrentMap和ConcurrentHashMap 

详情见另一个文章。

##### 1.8 Hashtable和HashMap比较

-  Hashtable和HashMap都实现了Map接口，但是Hashtable的实现是基于Dictionary抽象类。
-  HashTable 是线程安全的，不能存储 null 值
-  HashMap 不是线程安全的，可以存储 null 值

#### 三、Collections类和Collection接口

![collection](C:\Users\72703\Desktop\collection.jpg)

-  Collections是针对集合类的一个帮助类，他提供一系列静态方法实现对各种集合的搜索、排序、线程安全化等操作。
-  Collection是最基本的集合接口，一个Collection代表一组Object，即Collection的元素（Elements）。一些 Collection允许相同的元素而另一些不行。一些能排序而另一些不行。Java SDK不提供直接继承自Collection的 类，Java SDK提供的类都是继承自Collection的“子接口”如List和Set。
-  所有实现 Collection 接口的类都必须提供两个标准的构造函数：无参数的构造函数用于创建一个空的 Collection ，有一个 Collection 参数的构造函数用于创建一个新的 Collection ，这个新的 Collection 与传入的 Collection 有相同的元素。后一个构造函数允许用户复制一个 Collection 。

#### 四、List接口，有序可重复的集合

实际上有两种List: 一种是基本的ArrayList,其优点在于随机访问元素，另一种是更强大的LinkedList,它并不是为快速随机访问设计的，而是具有一套更通用的方法。 

List : 次序是List最重要的特点：它保证维护元素特定的顺序。List为Collection添加了许多方法，使得能够向List中间插入与移除元素（这只推荐LinkedList使用。）一个List可以生成ListIterator,使用它可以从两个方向遍历List,也可以从List中间插入和移除元素。 

#####4.1 ArrayList类

1) ArrayList实现了可变大小的数组。它允许所有元素，包括null。ArrayList没有同步。
2) size，isEmpty，get，set方法运行时间为常数。但是add方法开销为分摊的常数，添加n个元素需要O(n)的时间。其他的方法运行时间为线性。
3) 每个ArrayList实例都有一个容量（Capacity），即用于存储元素的数组的大小。这个容量可随着不断添加新元素而自动增加，但是增长[算法](http://lib.csdn.net/base/datastructure) 并没有定义。当需要插入大量元素时，在插入前可以调用ensureCapacity方法来增加ArrayList的容量以提高插入效率。
4) 和LinkedList一样，ArrayList也是非同步的（unsynchronized）。

5) 由数组实现的List。允许对元素进行快速随机访问，但是向List中间插入与移除元素的速度很慢。ListIterator只应该用来由后向前遍历ArrayList,而不是用来插入和移除元素。因为那比LinkedList开销要大很多。

6) ArrayList类如何动态扩容？JDK1.6中是使用无参构造函数时，初始容量是10，满了之后需要添加下一个元素时才会扩容成1.5倍。

#####4.2 Vector类

　　Vector非常类似ArrayList，但是Vector是同步的。由Vector创建的Iterator，虽然和ArrayList创建的Iterator是同一接口，但是，因为Vector是同步的，当一个Iterator被创建而且正在被使用，另一个线程改变了Vector的状态（例如，添加或删除了一些元素），这时调用Iterator的方法时将抛出ConcurrentModificationException，因此必须捕获该异常。

#####4.3 LinkedList类

　　LinkedList实现了List接口，允许null元素。此外LinkedList提供额外的get，remove，insert方法在 LinkedList的首部或尾部。如下列方法：addFirst(), addLast(), getFirst(), getLast(), removeFirst() 和 removeLast(), 这些方法 （没有在任何接口或基类中定义过）。这些操作使LinkedList可被用作堆栈（stack），队列（queue）或双向队列（deque）。
　　注意LinkedList没有同步方法。如果多个线程同时访问一个List，则必须自己实现访问同步。一种解决方法是在创建List时构造一个同步的List：
　　List list = Collections.synchronizedList(new LinkedList(...));

#####4.4 Stack 类

　　Stack继承自Vector，实现一个后进先出的堆栈。Stack提供5个额外的方法使得Vector得以被当作堆栈使用。基本的push和pop方法，还有peek方法得到栈顶的元素，empty方法[测试](http://lib.csdn.net/base/softwaretest)堆栈是否为空，search方法检测一个元素在堆栈中的位置。Stack刚创建后是空栈。

##### 4.5 ArrayList、vector和LinkedList的区别。

- ArrayList ， Vector ， LinkedList 是 List 的实现类

  ArrayList 是线程不安全的， Vector 是线程安全的，这两个类底层都是由数组实现的

  LinkedList 是线程不安全的，底层是由链表实现的  


- 1.ArrayList是实现了基于动态数组的数据结构，LinkedList基于链表的数据结构。
- 2.对于随机访问get和set，ArrayList觉得优于LinkedList，因为LinkedList要移动指针。
- 3.对于新增和删除操作add和remove，LinedList比较占优势，因为ArrayList要移动数据。
- 如果熟悉数据结构的同学，就会一下明白，ArrayList就是线性表的顺序表示，LinkedList就是线性表的链表表示。

#### 五、 Set接口，代表无序，不可重复的集合

Set具有与Collection完全一样的接口，因此没有任何额外的功能，不像前面有两个不同的List。实际上Set就是Collection,只是行为不同。（这是继承与多态思想的典型应用：表现不同的行为。）Set不保存重复的元素（至于如何判断元素相同则较为负责） 
Set : 存入Set的每个元素都必须是唯一的，因为Set不保存重复元素。加入Set的元素必须定义equals()方法以确保对象的唯一性。Set与Collection有完全一样的接口。Set接口不保证维护元素的次序。 

#####5.1 HashSet 

​     为快速查找设计的Set。存入HashSet的对象必须定义hashCode()。 

#####5.2 TreeSet

​     保存次序的Set, 底层为树结构。使用它可以从Set中提取有序的序列。 

##### 5.3 LinkedHashSet 

​     具有HashSet的查询速度，且内部使用链表维护元素的顺序（插入的次序）。于是在使用迭代器遍历Set时，结果会按元素插入的次序显示。





#### 六、如何选择

#####6.1 容器类和Array的区别、择取

​      1)容器类仅能持有对象引用（指向对象的指针），而不是将对象信息copy一份至数列某位置。
​      2)一旦将对象置入容器内，便损失了该对象的型别信息。

##### 6.2 其他

- 1)  在各种Lists中，最好的做法是以ArrayList作为缺省选择。当插入、删除频繁时，使用LinkedList()；Vector总是比ArrayList慢，所以要尽量避免使用。
- 2) 在各种Sets中，HashSet通常优于HashTree（插入、查找）。只有当需要产生一个经过排序的序列，才用TreeSet。HashTree存在的唯一理由：能够维护其内元素的排序状态。
- 3) 在各种Maps中,HashMap用于快速查找。
- 4)  当元素个数固定，用Array，因为Array效率是最高的。

**结论：最常用的是ArrayList，HashSet，HashMap，Array。而且，我们也会发现一个规律，用TreeXXX都是排序的。**

**注意：**

- 1、Collection没有get()方法来取得某个元素。只能通过iterator()遍历元素。

- 2、Set和Collection拥有一模一样的接口。

- 3、List，可以通过get()方法来一次取出一个元素。使用数字来选择一堆对象中的一个，get(0)...。(add/get)

- 4、一般使用ArrayList。用LinkedList构造堆栈stack、队列queue。

- 5、Map用 put(k,v) / get(k)，还可以使用containsKey()/containsValue()来检查其中是否含有某个key/value。HashMap会利用对象的hashCode来快速找到key。

- **hashing**    哈希码就是将对象的信息经过一些转变形成一个独一无二的int值，这个值存储在一个array中。

     我们都知道所有存储结构中，array查找速度是最快的。所以，可以加速查找。

​          发生碰撞时，让array指向多个values。即，数组每个位置上又生成一个梿表。

- **6、Map中元素，可以将key序列、value序列单独抽取出来。**
  - 使用keySet()抽取key序列，将map中的所有keys生成一个Set。
  - 使用values()抽取value序列，将map中的所有values生成一个Collection。
  - 为什么一个生成Set，一个生成Collection？那是因为，key总是独一无二的，value允许重复。

####七、常见面试题

##### 1、ArrayList和Vector的区别

- 相同点
- 不同点
  - 同步性
  - 数据增长

#####2、HashMap和Hashtable的区别

- 相同点
- 不同的
  - 历史原因
  - 同步性
  - 值

##### 3、List、Map和Set三个接口，存取元素时，各有什么特点？

- 先说单列元素的List和Set，他们有共同的父接口。
- 再说双列元素的Map。

##### 4、ArrayList、Vector和LinkedList的存储性能和特性。

##### 5、Collection和Collections的区别

##### 6、Set用什么方法区分重复和不重复？equals()和==有什么区别

##### 7、说出你所知道的集合类以及他们的主要方法以及他们的底层实现结构。

