### java集合类的功能实现

#### 一、HashMap功能实现

##### a、遍历方法

```
    //Collection And Map
    public static void testCM(){
        //Collection
        Map<Integer , String> hs = new HashMap<Integer , String>();
        int i = 0;
        hs.put(199, "序号:"+201);
        while(i<50){
            hs.put(i, "序号:"+i);
            i++;
        }
        hs.put(-1, "序号:"+200);
        hs.put(200, "序号:"+200);

        //遍历方式一:for each遍历HashMap的entryset，注意这种方式在定义的时候就必须写成
        //Map<Integer , String> hs，不能写成Map hs;
        for(Entry<Integer , String> entry : hs.entrySet()){
            System.out.println("key:"+entry.getKey()+"  value:"+entry.getValue());
        }
        //遍历方式二：使用EntrySet的Iterator
        Iterator<Map.Entry<Integer , String>> iterator = hs.entrySet().iterator();
        while(iterator.hasNext()){
            Entry<Integer , String> entry =  iterator.next();
            System.out.println("key:"+entry.getKey()+"  value:"+entry.getValue());
        };
        //遍历方式三：for each直接使用HashMap的keyset
        for(Integer key : hs.keySet()){
            System.out.println("key:"+key+"  value:"+hs.get(key));
        };
        //遍历方式四：使用keyset的Iterator
        Iterator keyIterator = hs.keySet().iterator();
        while(keyIterator.hasNext()){
            Integer key = (Integer)keyIterator.next();
            System.out.println("key:"+key+"  value:"+hs.get(key));
        }   
    }
```

（1）使用keyset的两种方式都会遍历两次，所以效率没有使用EntrySet高。

（2）HashMap输出是无序的，这个无序不是说每次遍历的结果顺序不一样，而是说与插入顺序不一样。

##### b、按值排序

```
    //对HashMap排序
    public static void sortHashMap(Map<Integer , String> hashmap){

        System.out.println("排序后");

        //第一步，用HashMap构造一个LinkedList
        Set<Entry<Integer , String>> sets = hashmap.entrySet();
        LinkedList<Entry<Integer , String>> linkedList = new LinkedList<Entry<Integer , String>>(sets);

        //用Collections的sort方法排序
        Collections.sort(linkedList , new Comparator<Entry<Integer , String>>(){

            @Override
            public int compare(Entry<Integer , String> o1, Entry<Integer , String> o2) {
                // TODO Auto-generated method stub
                /*String object1 = (String) o1.getValue();
                String object2 = (String) o2.getValue();
                return object1.compareTo(object2);*/
                return o1.getValue().compareTo(o2.getValue());
            }

        });

        //第三步，将排序后的list赋值给LinkedHashMap
        Map<Integer , String> map = new LinkedHashMap();
        for(Entry<Integer , String> entry : linkedList){
            map.put(entry.getKey(), entry.getValue());
        }
        for(Entry<Integer , String> entry : map.entrySet()){
            System.out.println("key:"+entry.getKey()+"  value:"+entry.getValue());
        }
    }
```

##### c、按键排序

HashMap按键排序要比按值排序方法容易实现，而且方法很多，下面一一介绍。

第一种：还是熟悉的配方还是熟悉的味道，用Collections的sort方法，只是更改一下比较规则。

第二种：TreeMap是按键排序的，默认升序，所以可以通过TreeMap来实现。

```
    public static void sortHashMapByKey(Map hashmap){

        System.out.println("按键排序后");

        //第一步：先创建一个TreeMap实例，构造函数传入一个Comparator对象。
        TreeMap<Integer , String> treemap = new TreeMap<Integer , String>(new Comparator<Integer>(){

            @Override
            public int compare(Integer o1,Integer o2) {
                // TODO Auto-generated method stub
                return Integer.compare(o1, o2);
            }

        });
        //第二步：将要排序的HashMap添加到我们构造的TreeMap中。
        treemap.putAll(hashmap);
        for(Entry<Integer , String> entry : treemap.entrySet()){
            System.out.println("key:"+entry.getKey()+"  value:"+entry.getValue());
        }
    }
```

第三种：可以通过keyset取出所有的key，然后将key排序，再有序的将key-value键值对存到LinkedHashMap中，这个就不贴代码了，有兴趣的可以自己去尝试一下。

##### d、value去重

对于HashMap而言，它的key是不能重复的，但是它的value是可以重复的，有的时候我们要将重复的部分剔除掉。

方法一：将HashMap的key-value对调，然后赋值给一个新的HashMap，由于key的不可重复性，此时就将重复值去掉了。最后将新得到的HashMap的key-value再对调一次即可。

##### e、HashMap线程同步

第一种：

```
Map<Integer , String> hs = new HashMap<Integer , String>();
        hs = Collections.synchronizedMap(hs);
```

第二种：

```
ConcurrentHashMap<Integer , String> hs = new ConcurrentHashMap<Integer , String>();
```



#### 二、List

##### a、排序

List的排序的话就是使用Collections的sort方法，构造Comparator或者让List中的对象实现Comparaable都可以，这里就不贴代码了。

##### b、去重

第一种：用Iterator遍历，遍历出来的放到一个临时List中，放之前用contains判断一下。

第二种：利用set的不可重复性,只需三步走。

```
//第一步：用HashSet的特性去重
    HashSet tempSet = new HashSet(arrayList);
//第二步：将arrayList清除
    tempSet.clear();
//第三步：将去重后的重新赋给List
    arrayList.addAll(tempSet);
```

#### 三、ArrayList如何实现线程安全

##### a、使用synchronized关键字。

对比线程安全的类Vector和线程不安全的类ArrayList

- ArrayList的add()方法被多线程访问时，需要这样写

```
ArrayList<String> list = new ArrayList<>();
synchronized(list){
  list.add("233");
}
```

- 而Vector只需要这样写就行了

```
Vector<String> list = new Vector<>();
list.add("233");
```

- 为什么？

  因为ArrayList的add()方法是这样定义的

  ```
  public boolean add(E object){
    ....
  }
  ```

  而Vector的add()方法是这样定义的

  ```
  public synchronized boolean add(E object){
    ....
  }
  ```

  ​

##### b、使用Collections.synchronizedList()

使用方法如下：

```
List<Map<String,Object>>data = Collections.synchronizedList(new ArrayList<Map<String,Object>>())
```

相比第一种实现线程安全的方法，第二种是非常损耗性能的。



##### c、实现原理

线程安全，即线程同步，就是多线程访问时，仅允许一个线程持有对象的锁，执行其代码。执行完毕后，释放锁，其他线程竞争获得该锁，再进行排他性的访问。