![image](https://gitee.com/ellen-eager/mypic/raw/master/image/Java%E5%AE%B9%E5%99%A8%E6%80%BB%E7%BB%93.png)

# 一、HashSet源码解析
## 1.1 成员变量
```
    //基于HashMap实现，底层使用HashMap保存所有元素
    private transient HashMap<E,Object> map;

    //HashSet存储的是HashMap的key，所以这个final的对象用来做HashMap的值
    private static final Object PRESENT = new Object();
```
## 1.2 构造方法
```
    public HashSet() {
        map = new HashMap<>();
    }

    public HashSet(Collection<? extends E> c) {
        map = new HashMap<>(Math.max((int) (c.size()/.75f) + 1, 16));
        addAll(c);
    }

    public HashSet(int initialCapacity, float loadFactor) {
        map = new HashMap<>(initialCapacity, loadFactor);
    }

    public HashSet(int initialCapacity) {
        map = new HashMap<>(initialCapacity);
    }

    HashSet(int initialCapacity, float loadFactor, boolean dummy) {
        map = new LinkedHashMap<>(initialCapacity, loadFactor);
    }
```
构造函数都是对HashMap的构造，包括容量大小和负载因子，最后一个是构造了一个LinkedHashMap。

## 1.3 add操作
```
    public boolean add(E e) {
        return map.put(e, PRESENT)==null;
    }
```
add操作其实就是HashMap的put操作，将要加入的元素作为HashMap的key，将一个不可变的对象作为HashMap的值。看了HashMap的put源码就可以知道HashSet的去重原理，下面分析HashMap的put操作源码：
```
    final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        else {
            Node<K,V> e; K k;
            //先比较hash，再比较内容
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
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
                    //先比较hash，再比较内容
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
    }
```
如果key的hashCode相同，并且key的内容也相同，那么就是同一个key，新的value会覆盖掉旧的value，所以key不允许重复。而HashSet存储的就是key，所以HashSet不允许重复。
# 二、HashSet常见面试问题
## 2.1 HashSet如何去重的？
根据对象的hashcode计算桶中的位置，比较hashcode是否与已经存在的对象的hashCode相等，如果hashcode相等，调用equals()方法判断内容是否也相等，如果相等则说明是同一个对象，就不会插入。
# 三、总结
HashSet就是HashMap的key的集合。