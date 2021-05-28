---
title: Java核心技术-基础
date: 2019-3-5 19:23:57
tags: Java核心技术
---

# Exception 和 Error

### 定义

- Exception和Error都继承了Throwable类。Java中只有Throwable实例才可以被抛出或捕捉。

- Exception 是程序正常运行中，可以预料的意外情况，可能并且应该被捕获，进行相应处理。

- Error 是指在正常情况下，不大可能出现的情况，绝大部分的 Error 都会导致程序（比如 JVM 自身）处于非正常的、不可恢复状态。既然是非正常情况，所以不便于也不需要捕获。

- Exception又分为**可检查（checked）**异常和**不可检查（unchecked）**异常。
  - 可检查异常在源代码里必须进行显示的捕捉处理，这是编译器检查的一部分；
  - 不检查异常就是所谓的运行时异常，类似 `NullPointerException`、`ArrayIndexOutOfBoundsException`之类通常是可以避免的逻辑错误，需要根据需求来判断是否进行捕捉，编译时并不会强制要求。

### 异常处理的原则

1. 尽量不要捕捉Exception这样的通用一场，而是应该捕捉特定异常。

2. 不要生吞异常。

   - 即捕获异常之后应该做处理或者输出到日志中，而不是不做处理。

   - 对于异常的处理，如果没有清晰的业务逻辑支持，可以保留原有异常的cause信息，直接抛出或者重新构建新的异常抛出。

### 对性能的影响

- `try-catch`代码块会产生额外的性能开销：它会影响JVM对代码的优化。所以建议仅捕捉有可能产生异常的代码块，不要用`try-catch`包住一大块代码块。
- Java每实例化一个Exception，都会对当时的栈进行快照，这是一个相对较重的操作。

# final、finally和finalize

### 定义

- final可以用来修饰类、方法、变量：final用来修饰类表示不可继承，修饰方法表示不可重写（override），修饰变量表示变量不可修改。
- finally则是Java保证终点代码一定要被执行的一种机制，如`try-catch-finally`块。
- finalize是基础类`java.lang.Object`，设计目的是保证对象在被GC之前完成特定资源的回收。finalize现在已经不推荐使用，JDK9开始已经被标记为`deprecated`。
  - finalize被设计为在垃圾回收之前调用，JVM需要对它进行额外的处理。当实现了非空的finalize方法时，会是的垃圾回收出现指数倍的变慢。

### 扩展

- final不是真正的不可改变，final只能约束对象的引用不被修改，但是对象值本身不会被影响。

# 引用

强引用、软引用、弱引用、幻象引用

### 定义

- 强引用，最常见的引用，被强引用的对象不会被GC。
- 软引用，可以让对象豁免一些垃圾回收。JVM只有在确保抛出OOM之前对软引用指向的对象进行清理。软引用通常用来实现内存敏感的缓存，保证了使用缓存的同时节省内存。
- 弱引用，不能使对象被GC豁免。它维护了一种非强制性的映射关系：当尝试获取对象时对象还在，就获取它，否则重新进行实例化。
- 幻象引用，有时也译成虚引用，不能通过它访问对象。幻象引用提供了一种在对象确保被finalize之后做某些事情的机制。

### 对象的可达性状态流转

![对象生命周期与可达状态](https://ws3.sinaimg.cn/large/005BYqpgly1g0tbd02pv1j30dt0g9dfz.jpg)

所有的引用类型，都是`java.lang.ref.Reference`的子类，该类提供了`get()`方法。

除了幻象应用，如果对象还没有被销毁，都可以通过get方法获取到原有对象，而幻象引用的get方法永远会返回null。利用该方法可以将软引用与弱引用访问的对象重新指向强引用。

若错误地保持了强引用，可能导致内存泄露。

### 引用队列

当一个引用所指向的对象被GC之后，该引用便会被enqueue到引用队列中。

利用引用队列可以再对象被GC之后进项一系列操作。

### 显示影响软引用的GC

软引用在最后一次引用之后，还会存在一段时间，默认值是根据剩余空间值计算的：

- 在client模式中，剩余空间是计算当前堆空间空闲大小，所以倾向于回收
- 在server模式中，是根据-Xmx的最大值计算的

在Java1.3.1开始，JVM提供参数`-XX:SoftRefLRUPolicyMSPerMB`，可以指定存在时间（以毫秒为单位）。

### Reachability Fence

JDK9的新特性，可以使用该模式来使得对象在没有强引用的情况下强可达。

`Reference.reachabilityFence(this)`可以使对象保持不被GC。

# String、StringBuffer与StringBuilder

- String是一个Immutable类，声明为final，其属性也都是final的。由于其不可变性，所有的拼接裁剪等操作都会产生新的字符串。
- StringBuffer是可变类。其本质上是一个线程安全的可修改字符串，保证了线程安全的同时加大了开销。
- StringBuilder是Java1.5之后新增的，功能与StringBuffer相同，但是去掉了线程安全的部分，减小了开销。

### 字符串设计

String类是Immutable的典型实现，原生地保证了线程安全。由于不可以对内部数据进行更改，在函数拷贝时不需要额外复制其他数据。

StringBuffer通过将所有修改数据的方法都加上`synchronized`关键字来实现线程安全。

StringBuffer和StringBuilder底层是同时可修改的char数组（JDK9以后是byte数组），二者都继承了`AbstractStringBuilder`。数组的默认初始空间是16，如果后续有大量拼接操作，应该手动指定合适的大小，避免过多的扩容操作，影响性能。

在JDK8，非静态的String拼接会被JVM编译成StringBuilder操作；JDK9则将字符串的拼接与javac生成的字节码解耦。所以一般情况下不需要特别需要注意String造成的性能影响，仅需在关键节点（如循环中）注意即可。

### 字符串缓存

String在内存中有大量的重复。JDK提供了`intern()`方法，该方法可以将字符串缓存起来以便重复使用。

在JDK6中，不建议使用该方法：JDK6是将字符串缓存放置在`PermGen（永久代）`中，容易引起OOM；在之后的版本，缓存被移至堆中，且提供了修改默认缓存空间大小的JVM参数`-XX:StringTableSize=N`，提供了解决占用空间的问题。

intern是一种**显式的排重机制**，但并不方便实用，且难以保证效率。在`JDK 8U20`之后，Java提供了一个新特性，也就是G1 GC下的排重。该方法是JVM底层的改变，不需要改变代码。该特性默认是关闭的，可以使用参数`-XX:+UseStringDeduplication`来打开。

# Java反射机制、动态代理

[反射](https://www.sczyh30.com/posts/Java/java-reflection-1/)

### 定义

- 反射机制是Java语言提供的一种基础功能，赋予程序在运行时能够观察并修改自身的能力。

  通过反射我们可以直接操作类或者对象，比如获取某个对象的类定义，获取类声明的属性和方法，调用方法或者构造对象，甚至可以运行时修改类定义。

- 动态代理是一种方便运行时动态构建代理、动态处理方法调用的机制，很多场景都是利用类似机制做到的，比如用来包装RPC调用、面向切面的变成（AOP）。

  实现动态代理的方法很多，比如JDK自身提供的动态代理，就是主要利用了反射机制。还有其他的实现方式，比如利用传说中更高性能的字节码操作机制，类似ASM、cglib（基于ASM）、Javassist等、

### 反射

反射机制最大的作用之一便是可以加载一个在运行时才知道名称的class，获悉其构造方法，并生成其实体对象，能对其对象设值并唤起其方法。

### 动态代理

它首先是一个**代理机制**，代理可以看做是对调用目标的一个包装。很多动态代理场景，也可以看做是装饰器模式的应用。通过代理可以实现调用者与实现者之间的解耦。

#### 一个简单的JDK动态代理实现：

```java
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;

public class MyDynamicProxy {
    public static void main(String[] args) {
        HelloImpl hello = new HelloImpl();
        MyInvocationHandler handler = new MyInvocationHandler(hello);
        Hello helloProxy = (Hello) Proxy.newProxyInstance(HelloImpl.class.getClassLoader(), HelloImpl.class.getInterfaces(), handler);
        helloProxy.sayHello("Lu");
    }
}

interface Hello {
    void sayHello(String name);
}

class HelloImpl implements Hello {
    @Override
    public void sayHello(String name) {
        System.out.println("Hello " + name);
    }
}

class MyInvocationHandler implements InvocationHandler {

    private Object target;
    MyInvocationHandler(Object target) {
        this.target = target;
    }

    /**
     * 实现反射调用
     * @param proxy
     * @param method
     * @param args
     * @return
     * @throws Throwable
     */
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        return method.invoke(target, args);
    }
}
```

#### cglib 动态代理

cglib采用的是创建目标类的子类的方式，因为是子类化，可以达到近似使用被吊用者本身的效果。Spring框架中支持JDK代理和cglib两种代理方式，可以显式指定。

#### JDK代理和cglib代理优势对比

- JDK
  - 最小化依赖关系，意味着简化开发和维护；
  - 平滑进行JDK升级，而字节码库通常需要更新以保证在新版本Java上使用；
  - 代码实现简单。
- cglib
  - 不限定调用者实现接口；
  - 只操作我们关心的类，而不必为其他类增加工作量；
  - 高性能。

# int和Integer

- int是Java八个基础数据类型（boolean、byte、char、short、int、long、double、float）之一。

  Java虽然是面向对象编程，但基础数据类型却不是对象。

- Integer是int的包装类。它有一个int类型的字段存储值数据，并提供了对该数据的一些基本操作方法。在Java5中引入了自动装箱和自动拆箱功能，可根据上下文自动转换数据类型。

  Integer的值缓存是Java5中引入的另一项改进：传统构建Integer的方法是直接调用其构造函数，new一个对象。但根据实践发现大部分的数据操作集中在一个较小的数据范围内，于是添加了静态工厂方法`valueOf`，使用该方法时会利用一个缓存机制来提升性能，这个缓存范围默认是-128到127。

  其他的包装类也有对应的缓存和自动装拆箱机制。

### 自动装箱、自动拆箱

自动装箱拆箱实际上是一种语法糖，发生在**编译**阶段。

javac帮我们把自动装箱替换为`Integer.valueOf()`，把自动拆箱替换为`Integer.intValue()`。

原则上，建议避免无意义的装箱和拆箱，过多的对象占用的空间和时间与基础数据类型有着数量级上的差距。

### Integer源码分析

- Integer缓存可以使用JVM参数`-XX:AutoBoxCacheMax=N`来指定；
- Integer等其他包装类同String一样，是不可变类型，这样做的目的是保证数据安全；

### Java原始数据类型和引用类型的局限性

- 基础数据类型不能用作泛型

  Java的泛型是伪泛型，Java编译时会讲泛型转换为对应的特定类型，这就保证泛型必须可以转换为Object。

- 无法高效的表达数据，也不便于表达复杂的数据结构

### 扩展：对象的内存结构

对象有三部分构成：对象头、对象实例、对齐填充。

- 对象头包含两个部分：
  - 第一部分存储对象自身和运行时数据，如哈希码，GC分代年龄、锁状态标志，线程持有的锁、偏向线程ID、偏向时间戳等等，这部分数据的长度在32位和64位的虚拟机（未开启压缩指针）中分别为32bit和64bit，官方称它为`Mark Word`；
  - 另一部分是类型指针，即对象指向它的类元数据的指针，虚拟机通过这个指针来确定这个对象时哪个类的实例。
- 对象实例就是对象存储的真正有效信息，也就是程序代码中所定义的各种类型的字段内容，无论是在父类中继承下来的，还是在子类中定义的，都需要记录起来。
- 对其补充不是必要的，仅仅是起到占位符的作用。由于HotPot VM的自动内容管理系统要求对象的起始地址必须是8字节的整数倍，所以使用该部分对内容进行填充。

# Vector、ArrayList和LinkedList

这三者都是实现集合框架中的有序集合（List），功能比较接近；但在行为、性能和安全性方面有较大差别。

- Vector是Java早期提供的**线程安全的动态数组**。Vector内部使用对象数组保存数据，使用数据拷贝进行动态扩容。如果不需要线程安全，不建议使用Vector，毕竟同步是需要额外开销的。
- ArrayList也是**动态数组**，但并不是线程安全的，所以性能更好。ArrayList的扩容机制与Vector相同，不同之处在于Vector每次扩容100%，而ArrayList扩容50%。
- LinkedList是Java提供的双向链表，所以它不需要调整容量，也不是线程安全的。

### 效率

- Vector和ArrayList是动态数组，本质上也是数组操作，所以其随机访问能力更为优秀，而除了尾部的添加与删除操作性能较差。
- LinkedList是链表，其插入和删除操作性能都很优秀，随机访问能力较差。

## 狭义的集合框架

该框架不包括java.uitl.concurrent下的线程安全容器添加进来；也没有Map容器，虽然它不是真正的集合，但我们通常概念上也会把它当做集合框架的一部分。

![集合框架](https://ws3.sinaimg.cn/large/005BYqpggy1g0xpb37ofpj30n70cc3z1.jpg)

Java的集合框架，Collection接口是所有集合的根，然后扩展了三大集合：

- List，有序集合，提供了方便的访问、插入、删除等操作。
- Set，与List最大的差别是不允许有重复元素。
- Queue/Deque，Java提供的标准队列结构的实现。除了集合的基本功能，还包括先入先出、先入后出等特定行为。这里并不包括BlockingQueue，因为通常是并发编程场合，所以放在并发包里。

每种集合的通用逻辑被抽象到对用的抽象类中，如AbstractList。但每种集合也不是完全孤立的，比如LinkedList便同时是List和Deque。

### 线程安全

这些容器都不是线程安全的，但是在Collections工具类中提供了一系列的synchronized方法；它的实现就是将类中的方法都加上synchronized关键词。

### 默认排序算法

- 对于基础数据类型，Java采用的是[双轴快速排序算法（Dual-Pivot QuickSort）](http://hg.openjdk.java.net/jdk/jdk/file/26ac622a4cab/src/java.base/share/classes/java/util/DualPivotQuicksort.java)，是一种改进的快速排序算法，早期版本采用的则是相对传统的排序算法。
- 对于对象数据类型，目前使用的是[TimSort](http://hg.openjdk.java.net/jdk/jdk/file/26ac622a4cab/src/java.base/share/classes/java/util/TimSort.java)，思想上也是一种归并和二分插入排序结合的优化排序算法。

### 扩展

在Java9中，Java标准类库提供了一系列的静态工厂方法，如`List.of()`、`Set.of()`等，大大简化了构建小容器的代码量。

```
List<String> simpleList = List.of("Hello", "World");
```

# HashTable、HashMap和TreeMap

三者都是最为常见的一些Map实现，是以**键值对**存储和操作数据的容器类型。

- HashTable是早期Java类库提供的一个哈希表实现，本身是同步的，不支持null键和值，由于同步导致的性能开销，很少被推荐使用。
- HashMap是应用更加广泛的哈希表实现，与HashTable的主要区别在于它本身不是同步的，且支持null键和值等。它的put和get操作通常情况下可以达到常量级的时间复杂度。它是绝大部分利用键值对存取场景的首选。
- TreeMap是基于红黑树的一种提供顺序访问的Map，与HashMap不同的是它的数据操作都是`O(logn)`的时间复杂度。

## Map结构

![Map结构](https://ws3.sinaimg.cn/large/005BYqpggy1g0xre0tjzaj30lc0dpmxe.jpg)

- HashTable作为早期集合相关类型，继承了Dictionary类，类结构上与其他Map明显不同。
- HashMap等其他Map实现继承了AbstractMap，实现了里边的通用方法抽象。

### HashMap

大部分的Map场景是访问、增删等，对顺序没有要求，HashMap是这些场景的最优选择。HashMap的表现非常依赖哈希码的有效性，需要掌握hashCode和equals的一些基本约定，比如：

- equals相同，hashCode一定要相等；
- 重写了hashCode也一定要重写equals；
- hashCode需要保证一致性，状态改变返回的哈希值仍然要一致；
- equals的对称、反射、传递等特性。

### 顺序访问

- LinkedHashMap通常提供的是遍历顺序符合插入顺序或者访问顺序。

  一个以LinkedHashMap为例设计的空间占用敏感的资源池，可以将最不长访问的对象释放掉：

  ```java
  import java.util.LinkedHashMap;
  import java.util.Map;
  
  public class LinkedHashMapSample {
      public static void main(String[] args) {
          LinkedHashMap<String, String> accessOrderedMap = new LinkedHashMap<String, String>(16, 0.75F, true){
              @Override
              protected boolean removeEldestEntry(Map.Entry<String, String> eldest) { // 实现自定义删除策略，否则行为就和普遍 Map 没有区别
                  return size() > 3;
              }
          };
          accessOrderedMap.put("Project1", "Valhalla");
          accessOrderedMap.put("Project2", "Panama");
          accessOrderedMap.put("Project3", "Loom");
          accessOrderedMap.forEach( (k,v) -> {
              System.out.println(k +":" + v);
          });
          // 模拟访问
          accessOrderedMap.get("Project2");
          accessOrderedMap.get("Project2");
          accessOrderedMap.get("Project3");
          System.out.println("Iterate over should be not affected:");
          accessOrderedMap.forEach( (k,v) -> {
              System.out.println(k +":" + v);
          });
          // 触发删除
          accessOrderedMap.put("Project4", "Mission Control");
          System.out.println("Oldest entry should be removed:");
          accessOrderedMap.forEach( (k,v) -> {// 遍历顺序不变
              System.out.println(k +":" + v);
          });
      }
  }
  
  ```

- TreeMap则是由键的顺序决定的。

## HashMap源码分析

### HashMap结构

![HashMap结构](https://ws3.sinaimg.cn/large/005BYqpggy1g0xsmofjsnj30mg0ce3yp.jpg)

HashMap可以看做是由数组和链表结合组成的复合结构，数组被分成一个个桶，通过Hash值决定了键值对在数组中的寻址；哈希值相同的键值对则以链表形式存储。如果链表大小超过阈值（TREEIFY_THRESHOLD, 8），链表则会被改成树结构。

### HashMap的put逻辑

HashMap的数组并不是在创建时就初始化好，而是在put的时候进行初始化。

```java
final V putVal(int hash, K key, V value, boolean onlyIfAbent,
               boolean evit) {
    Node<K,V>[] tab; Node<K,V> p; int , i;
    if ((tab = table) == null || (n = tab.length) = 0)
        n = (tab = resize()).length;
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    else {
        // ...
        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for first 
           treeifyBin(tab, hash);
        //  ... 
     }
}
```

put方法调用了putVal方法，在该方法中：

- 如果数组为null，会调用resize方法进行初始化；

- resize方法除了初始化数组外，还会在容量不满足需求的时候进行扩容；

- 扩容的条件：

  ```java
  if (++size > threshold)
      resize();
  ```

- 具体的键值对在哈希表中的位置取决于：

  ```java
  i = (n - 1) & hash
  ```

  不使用key本身的hashCode而是采用HashMap里的另一个Hash方法，原因是有些数据计算出的哈希值差异主要在高位，而 HashMap 里的哈希寻址是忽略容量以上的高位的，那么这种处理就可以有效避免类似情况下的哈希碰撞。

  ```java
  static final int hash(Object kye) {
      int h;
      return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>>16);
  }
  ```

#### resize方法

```java
final Node<K,V>[] resize() {
    // ...
    else if ((newCap = oldCap << 1) < MAXIMUM_CAPACIY &&
                oldCap >= DEFAULT_INITIAL_CAPAITY)
        newThr = oldThr << 1; // double there
       // ... 
    else if (oldThr > 0) // initial capacity was placed in threshold
        newCap = oldThr;
    else {  
        // zero initial threshold signifies using defaultsfults
        newCap = DEFAULT_INITIAL_CAPAITY;
        newThr = (int)(DEFAULT_LOAD_ATOR* DEFAULT_INITIAL_CAPACITY；
    }
    if (newThr ==0) {
        float ft = (float)newCap * loadFator;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?(int)ft : Integer.MAX_VALUE);
    }
    threshold = neThr;
    Node<K,V>[] newTab = (Node<K,V>[])new Node[newap];
    table = n；
    // 移动到新的数组结构 e 数组结构 
   }
```

不考虑极限情况（容量理论最大极限由MAXIMUM_CAPACITY指定，数值为1<<30，即2 的30次方）情况下：

- 门限值等于`负载因子×容量`，如果构建HashMap的时候没有指定他们，那么就是依据相应的默认常量值；
- 门限通常以倍数进行调整（`newThr = oldThr << 1`），根据putVal中的逻辑，当元素个数超过门限大小时，则调整Map大小；
- 扩容后，需要将老的数组中的元素重新放置到新的数组，这是扩容的一个主要开销来源。

### 容量、负载因子和树化

容量和负载因子决定了可用的桶数量，如果空桶太多会浪费空间，使用太满又会严重影响性能。

哈希表空闲位置不多的时候，哈希冲突的概率都会大大提高。为了尽可能保证散列表的操作效率，一般情况下会尽可能保证散列表中有一定比例的空闲槽位。使用**负载因子（load factor）**来表示空位的多少。

负载因子的计算公式：`装入表中的元素个数 / 散列表的长度`。

装在因子越大，说明空闲位置越少，冲突越多，性能会下降。

对于负载因子

- 如果没有特殊需求，不要随意修改，因为JDK自身的默认负载因子是非常符合通常场景的；
- 如果调整，建议不要超过0.75；
- 如果设置了过小的负载因子，会导致频繁的扩容。

#### 树化与哈希碰撞拒绝服务攻击

在放置元素的过程中，如果发生哈希冲突，就会放到一个桶里形成链表。而链表是线性的，严重影响存取性能。

而哈希碰撞拒绝服务攻击便是构建大量哈希冲突的数据，利用这些数据大量与服务器交互，导致服务器CPU占满。

# 容器的线程安全、ConcurrentHashMap

Java在传统集合框架内部，除了提供同步容器，还提供了**同步包装器（Synchronized Wrapper）**；我们可以调用Collections工具类提供的包装方法，来获取一个同步包装容器。但是它们采用的是非常粗粒度的同步方式，高并发情况下，性能比较低下。

## ConcurrentHashMap

HashTable本身比较低效，因为它的同步方式就是在方法上添加synchronized关键字。Collections提供的同步包装类也类似。

### ConcurrentHashMap早期的实现方式：

![ConcurrentHashMap的早期实现方式](https://ws3.sinaimg.cn/large/005BYqpgly1g0z47blpvpj30oh0gzglz.jpg)

- 分离锁，也就是将内部进行分段（Segment），里边则是HashEntry的数组，和HashMap类似，哈希相同的条目也是以链表形式存放。
- HashEntry内部使用volatile的value字段来保证可见性，也利用了不可变对象的机制以改进利用Unsafe提供的底层能力，比如 volatile access，去直接完成部分操作，以最优化新更能，毕竟Unsafe中的很多操作都是JVM intrinsic优化过的。

ConcurrentHashMap的get操作主要是保证可见性，没有同步逻辑；其put方法时进行二次哈希避免哈希冲突，获取其Segment，然后进行线程安全的put操作。

ConcurrentHashMap在进行并发写操作时

- ConcurrentHashMap会获取再入锁，以保证数据一致性，Segment本身就是基于ReentrantLock的扩展实现，所以，在并发修改期间，相应Segment是被锁定的；
- 在最初阶段，进行重复性的扫描，以确定相应key值是否已经在数组里面，进而决定是更新还是放置操作；重复扫描、检测冲突时ConcurrentHashMap的常见技巧；
- ConcurrentHashMap的扩容不是整体扩容，而是对Segment的扩容。

### JDK8及之后的ConcurrentHashMap实现方式：

- 结构上与HashMap非常相似：大的桶数组，内部是链表结构；
- 内部仍有Segment定义，但仅仅是为了保证序列化时的兼容性，不再有结构上的用处；
- 不再使用Segment使得初始化更加简化，修改为lazy-load模式，有效避免了初始开销；
- 数据存储利用volatile来保证可见性；
- 使用CAS等操作，在特定场景进行无锁并发操作；
- 使用Unsafe、LongAdder之类底层手段，进行极端情况的优化。

初始化操作实现在 initTable 里面，这是一个典型的 CAS 使用场景，利用 volatile 的 sizeCtl 作为互斥手段：如果发现竞争性的初始化，就 spin 在那里，等待条件恢复；否则利用 CAS 设置排他标志。如果成功则进行初始化；否则重试。

PS：1.8以后的锁的颗粒度是加在链表头上的。

# IO、NIO

- 传统的java.io包基于流模型实现，提供了一些最熟知的IO功能，它的交互功能是同步、阻塞的；有时候java.net下提供的部分网络api也归类到同步阻塞io库；
- java1.4中引入了java.nio包，可以构建多路复用的同步非阻塞IO程序；
- java1.7中NIO有了进一步改进（NIO2），引入了异步非阻塞IO方式，也被称作`AIO(Asynchronous IO)`。异步IO基于事件和回调机制。

### 基本概念

- 同步或异步（synchronous/asynchronous）：同步是一种可靠的有序运行机制，进行同步操作时，后续的任务是等待当前调用返回才进行下一步；而异步不需要等待，通常依靠事件、回调等机制来实现任务次序关系。
- 阻塞与非阻塞（blocking/non-blocking）：当进行阻塞操作时，当前线程无法从事其它任务，只有条件就绪才能继续；而非阻塞操作不管IO是否结束，直接返回，相应结果在后台继续完成。

![Java IO结构图](https://ws3.sinaimg.cn/large/005BYqpgly1g1jw4hsgmrj30mt0het9i.jpg)

### java.io

输入输出流（InputStream/OutputStream）都是用于读写字节的

Reader/Writer是用于操作字符的，增加了字符解码编码功能。本质上计算机操作的都是字节，Reader/Writer相当于构建了应用逻辑和原始数据之间的桥梁

BufferedOutputStream等带缓冲区的实现，可以避免频繁的磁盘读写，进而提高了IO效率

很多IO工具类都实现了Closeable接口，因为需要进行资源的释放

### java.nio

#### 主要组成部分：

- Buffer，高效的数据容器，除了boolean型，所有原始数据类型都有相应的Buffer实现

- Channel，类似于在Linux上看到的文件描述符，是NIO中被用来支持批量式IO操作的一种抽象。

  File或者Socket，通常被认为是比较高层次的抽象，而Channel则是更加操作系统层面的一种抽象，这也使得NIO得以充分利用现代操作系统底层机制，获得特定场景的性能优化。不同层次的抽象是相互关联的，可以通过Socket获取Channel，反之亦然。

- Selector，是NIO实现多路复用的基础，它提供了一种高效机制可以检测到注册在Selector上的多个Channel中，是否有Channel处于就绪状态，进而实现了单线程对多Channel的高效管理。

  Selector同样是基于底层操作系统，不同模式、不同版本都有区别。

- Charset，提供Unicode字符串定义，NIO也提供了相应的编码解码器。

## 拓展：文件拷贝

### 拷贝实现机制分析

#### io库的输入输出流方式

- 用户空间（User Space）和内核空间（Kernel Space）：操作系统内核，硬件驱动等运行在内核空间，具有相对较高的特权；而用户空间则是给普通应用和服务使用
- 使用输入输出流读写时，实际上是进行了多次上下文切换，如读取数据时，先将内核态数据从磁盘读取到内核缓存，再切换到用户态数据从内核缓存读取到用户缓存；写入则步骤相反。

#### nio库的trasferTo方式

- 在Linux和Unix上，则会用到零拷贝技术：不需要用户空间参与，提高性能。

#### Files.copy

# 接口和抽象类

- 接口是对行为的抽象，他是抽象方法的合集，利用接口可以达到API与实现分离的目的，接口不能实例化；不能包含任何非常量成员，任何filed都是隐含着public static final的意义；同时，没有非静态方法实现，要么是抽象方法，要么是静态方法。
- 抽象类是不能实例化的类，用abstract关键字修饰class，其目的主要是代码重用。除了不能实例化，形式上和一般的Java类没有太大区别，可以有一个或多个抽象方法，也可以没有抽象方法。抽象类大多用于抽取相关Java类的共同方法实现或者是共同成员变量，然后通过继承的方式达到代码复用的目的。

## 拓展

- Java单继承，多实现
- 抽象类可以提供非抽象方法作为对子类的能力扩展，子类无需添加额外的代码便可以实现相应的功能
- 对于接口，可以提供一系列的抽象方法，其实现类需要将其实现；同时，也有一些接口没有任何抽象方法，仅仅是为了声明某些东西，它们通常被称作Marker Interface。
- Java8增加了对函数式编程的支持，所以有增加了一类定义，即functional interface，是只有一个抽象方法的接口。使用`@FunctionalInterface`标记。Lamda表达式本身就可以看做是一类functional interface。
- Java8以后，接口可以有方法实现。Java8增加了interface对default method的支持。

### 面向对象设计

- **封装**：隐藏内部实施细节，提高安全性和简化编程。
- **继承**：代码复用的基础机制。
- **多态**：重载，重写，向上转型

#### S.O.L.I.D原则

- 单一原则（Single Responsibility），类或者对象最好只有单一职责，在程序设计中如果某个类承担多种义务，需要进行拆分。
- 开关职责（Open-CLose，Open for extension，Close for modification），设计要对扩展开放，对修改关闭。
- 里氏替换（Liskov Substitution），面向对象的基本要素之一，进行继承关系抽象时，凡是可以用父类或基类的地方，都可以用子类代替。
- 接口分离（Interface Segregation），在进行类和接口设计时，保证接口的功能单一性，可以功能复杂的接口拆分成多个接口，保证程序的内聚性。
- 依赖反转（Dependency Inversion），实体应该依赖于抽象而不是实现。就是说高层次模块不应该依赖于低层次模块，而应该基于抽象。