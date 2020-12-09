### 反射内省

通过getUserName/setUserName来访问userName属性，java JDK提供了一套api用来访问某个属性的getter/setter方法，这就是内省
```java
// 将名为 name 值为 value 的属性写入到 condiguration
PropertyDescriptor pd = new PropertyDescriptor(name, Configuration.class);
Method writeMethod = pd.getWriteMethod();
writeMethod.invoke(configuration, value);

//获取属性
PropertyDescriptor pd = new PropertyDescriptor(fieldName, obj.getClass());
//从属性描述器中获取 get 方法
Method method = pd.getReadMethod();
Object value = method.invoke(obj);
//执行方法并返回结果
return value==null?"":String.valueOf(value);
```

### 初始化的执行顺序

1. 父类静态代码块（ java 虚拟机加载类时，就会执行该块代码，故只执行一次）
2. 子类静态代码块（ java 虚拟机加载类时，就会执行该块代码，故只执行一次）
3. 父类属性对象初始化
4. 父类普通代码块（每次 new ,每次执行）
5. 父类构造函数（每次 new ,每次执行）
6. 子类属性对象初始化
7. 子类普通代码块（每次 new ,每次执行）
8. 子类构造函数（每次 new ,每次执行）

#### 实现代码
```java
package com.basic;

public class Person {
    // 静态成员变量
    private static String ID = "1111";

    // 静态代码块
    static {
        System.out.println(ID);
    }

    // 普通的成员变量
    private String name = "王五";

    // 普通的代码块
    {
        System.out.println(this.name);
    }

    // 父类构造方法
    public Person(){
        System.out.println("父类构造方法。。");
    }
}

```

```java
package com.basic;

public class Student extends Person {
    // 静态成员变量
    private static String ID = "2222";

    // 静态代码块
    static {
        System.out.println(ID);
    }

    // 普通的成员变量
    private String name = "张三";

    // 普通的代码块
    {
        System.out.println(this.name);
    }

    // 父类构造方法
    public Student(){
        System.out.println("子类构造方法。。");
    }

    public static void main(String[] args) {
        Student student = new Student();
        Student student2 = new Student();
    }
}

```

#### 输出结果
```text
1111
2222
王五
父类构造方法。。
张三
子类构造方法。。
王五
父类构造方法。。
张三
子类构造方法。。
```

#### 思考题
```java
public class Base
{
    private String baseName = "base";
    public Base()
    {
        callName();
    }
 
    public void callName()
    {
        System. out. println(baseName);
    }
 
    static class Sub extends Base
    {
        private String baseName = "sub";
        public void callName()
        {
            System. out. println (baseName) ;
        }
    }
    public static void main(String[] args)
    {
        Base b = new Sub();
    }
}
```

结果是 null

按照执行顺序，是执行 callName() 方法没有错，但是**父类初始化调用的方法为子类实现的方法，子类实现的方法中调用的 baseName 为子类中的私有属性。**
所以此时结果为 null

### Java 类的加载机制

#### 1. 加载
1. classloader 在 classpath 路径下获取到 class 文件将其以**二进制流**的形式加载到 内存 中
2. 将字节流所代表的静态存储结构转化为 方法区运行时数据结构
3. 在内存中为该类生成一个 java.lang.Class 对象，作为方法区中这个类的访问入口

#### 2. 连接
1. 验证文件格式、元数据(是否符合Java语言规范)、字节码（确定程序语义合法，符合逻辑）和 符号引用（确保下一步的解析能正常执行）
2. **为静态变量在方法区分配内存，并设置默认初始值**
3. 虚拟机将常量池内的符号引用替换为直接引用

#### 3. 初始化
初始化是类加载的最后一步，主要是根据语句给变量赋值。

当有继承关系时，会先初始化父类再初始化子类，所以创建一个子类在内存中实际存在两个对象实例。（从这个角度思考**在类设计的时候继承关系做多不超过 3 层**）
#### 4. 使用
程序之间的相互调用
#### 5. 卸载
销毁对象，代码层面的销毁就是将引用置为 null（JVM 自行回收）

#### new 关键字

#### Class 的 newInstance 方法
```java
Employee emp2 = (Employee)
Class.forName("org.programming.mitra.exercises.Employee").newInstance();
```

#### 使用Constructor类的newInstance方法
```java
Constructor<Employee> constructor = Employee.class.getConstructor();
Employee emp3 = constructor.newInstance();
```

#### clone方法
调用一个对象的clone方法，jvm就会创建一个新的对象，将前面对象的内容全部拷贝进去。用clone方法创建对象并不会调用任何构造函数

要使用clone方法，我们需要先实现Cloneable接口并实现其定义的clone方法。


#### 使用反序列化
在反序列化时，jvm创建对象并不会调用任何构造函数，要实现反序列化需要实现 Serializable 接口
```java
ObjectInputStream in = new ObjectInputStream(new FileInputStream("data.obj"));
Employee emp5 = (Employee) in.readObject();
```

### 序列化

序列化是一种用来**处理对象流**的机制  

> 对象流：就是将对象的内容进行流化（转换成二进制流）。可以对流化后的对象进行读写操作，也可将流化后的对象传输于网络之间

**序列化是为了解决在对对象流进行读写操作时所引发的问题。**

具体实现：继承了 Serializable 接口实现可序列化，该接口不需要实现具体的方法，只是为了*标识*该对象是可以被序列化的。

#### 序列化的优势
- Java对象序列化不仅保留一个对象的数据，而且递归保存对象引用的每个对象的数据。可以将整个对象层次写入字节流中，可以保存在文件中或在网络连接上传递。
利用对象序列化可以进行对象的"深复制"，即**复制对象本身及引用的对象本身**。序列化一个对象可能得到整个对象序列
- 对象序列化可以实现分布式对象。
  主要应用例如：RMI(即远程调用Remote Method Invocation)要利用对象序列化运行远程主机上的服务，就像在本地机上运行对象时一样。
- 序列化可以将内存中的类写入文件或数据库中。再次使用时只需要将文件或者数据库中的数据反序列化就可以重新加载到内存中
- 对象、文件和数据格式很难统一

#### serialVersionUID
在反序列化时，java虚拟机会通过二进制流中的 serialVersionUID 与本地的对应的实体类进行比较，如果相同就认为是一致的，可以进行反序列化，正确获得信息，否则抛出序列化版本不一致的异常 NotSerializableException
而 Java 每次编译 class 的时候都会生成一个新的 id。

#### 如何避免序列化

1. `transient` 瞬态关键字可以修饰成员变量使其不能被序列化
2. 静态优先于非静态加载到内存中（静态优先于对象进入到内存）所以没有办法序列化
```java
private static void objectRead() throws IOException, ClassNotFoundException {
    ObjectInputStream ois = new ObjectInputStream(new FileInputStream("D:\\new.txt"));
    Object readObject = ois.readObject();
    System.out.println(readObject);
}

private static void objectWrite() throws IOException {
    Person person = new Person();
    person.setId("1");
    person.setName("zhangsan");
    person.setOther("game");
    // 对象也是以字节流写入到文件中
    ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("D:\\new.txt"));
    oos.writeObject(person);
    oos.flush();
    oos.close();
}
```


# 集合

## List
![image.png](https://cdn.nlark.com/yuque/0/2020/png/1471883/1590546621415-77c37edb-341e-44f6-a389-67a894982029.png#align=left&display=inline&height=702&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1403&originWidth=2214&size=124189&status=done&style=none&width=1107)


### ArrayList
#### 实现原理
ArrayList 底层是基于数组实现的，并且实现了数组的扩容。
```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
```
ArrayList 实现了 Serializable 和 Cloneable 接口所以可以实现序列化和克隆
#### 属性
```java
transient Object[] elementData;    
private static final int DEFAULT_CAPACITY = 10;	// 数组的默认大小为 10
```
elementData 是被关键词 transient 修饰（transient 修饰后表示该字段不会被序列化），这就涉及到 ArrayList 底层的实现，因为底层是个数组，也就意味着 ArrayList 的** elementData 数组中不是所有的内存空间都存储了数据**。


采用外部序列化数组的方式就会序列化全部数组，所以 ArrayList 将 elementData 数组使用 transient 修饰，防止对象数组被外部序列化，所以额外提供了私有的 `writeObject` 和 `readObject` 来实现序列化。
#### 操作元素
新增元素分为两个方法：在数组结尾插入，数组任意位置插入
```java
    public boolean add(E e) {
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        elementData[size++] = e;
        return true;
    }
    public void add(int index, E element) {
        rangeCheckForAdd(index);

        ensureCapacityInternal(size + 1);  // Increments modCount!!
        System.arraycopy(elementData, index, elementData, index + 1,
                         size - index);
        elementData[index] = element;
        size++;
    }
```
先判断数组容量，是否需要扩容，如果容量不够就会 **1.5 倍**扩容。
#### 数组的拷贝
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
本质上来说 `Arrays.copyOf` 基于 `System.arraycopy` 实现的。
```java
    public static <T,U> T[] copyOf(U[] original, int newLength, Class<? extends T[]> newType) {
        @SuppressWarnings("unchecked")
        T[] copy = ((Object)newType == (Object)Object[].class)
            ? (T[]) new Object[newLength]
            : (T[]) Array.newInstance(newType.getComponentType(), newLength);
        System.arraycopy(original, 0, copy, 0,
                         Math.min(original.length, newLength));
        return copy;
    }
```
#### 遍历元素
ArrayList 则是基于数组实现的，数组在内存中是连续的，所以可以整个缓存读取进 CPU 缓存，遍历的时候就不需要频繁的到主内存中获取了，所以使用 for 循环效率非常高（for 循环就是基于下标去访问）。
#### 面试题
**问题1**：我们在查看 ArrayList 的实现类源码时，你会发现对象数组 elementData 使用了 transient 修饰，我们知道 transient 关键字修饰该属性，则表示该属性不会被序列化，然而我们并没有看到文档中说明 ArrayList 不能被序列化，这是为什么？


**问题 2**：我们在使用 ArrayList 进行新增、删除时，经常被提醒“使用 ArrayList 做新增删除操作会影响效率”。那是不是 ArrayList 在大量新增元素的场景下效率就一定会变慢呢？


**问题 3**：如果让你使用 for 循环以及迭代循环遍历一个 ArrayList，你会使用哪种方式呢？原因是什么？
### RandomAccess 
 RandomAccess 接口是一个标记接口没有具体的实现，实现了 RandomAccess 接口的 List 类能够实现快速随机访问。
在 ` Collections#binarySearch()` 方法中就通过 RandomAccess 来区分了查询方式
```java
    public static <T> int binarySearch(List<? extends T> list, T key, Comparator<? super T> c) {
        if (c==null)
            return binarySearch((List<? extends Comparable<? super T>>) list, key);
		// 底层实现是数组，size < 5000 
        if (list instanceof RandomAccess || list.size()<BINARYSEARCH_THRESHOLD)
            return Collections.indexedBinarySearch(list, key, c);
        else
            return Collections.iteratorBinarySearch(list, key, c);
    }
```


### LinkedList
#### 实现原理
LinkedList 底层是双向链表，1.7之前只包含一个 Entry 类型的 header 属性，初始化的时候前后指针指向自己形成**循环双向链表 **，1.7 以后改为 Node，取消了 header 使用 first 和 last 替换。

- first/last 更加清晰
- 初始化的时候不需要 new Entry
- 链头和链尾的操作更加快捷



LinkedList 同样是实现了 Cloneable 和 Serializable 接口，可以实现序列化和克隆，此外还实现了 Deque 接口具有 Queue 的特性。由于 LinkedList 内存上不是连续的，所以不能实现 RandomAccess 接口。
```java

public class LinkedList<E>
    extends AbstractSequentialList<E>
    implements List<E>, Deque<E>, Cloneable, java.io.Serializable
```
#### 属性
```java
    transient int size = 0;
    transient Node<E> first;
    transient Node<E> last;
```
LinkedList 的 3 个属性均被 transient 修饰了，因为 LinkedList 序列化的时候不是单纯的对 first 和 last 节点序列化就可以了，所以 LinkedList 也额外提供了私有的 `writeObject` 和 `readObject` 。
#### 操作元素
LinkedList 直接添加元素到队尾或者添加元素到两个元素之间效率都很高，但是删除就不同了，因为删除的时候需要遍历元素，当队列很长并且要擅长的元素处于中间位置时效率就很低了。
#### 遍历元素
LinkedList 遍历元素和删除元素相似，使用 for 循环每次循环都需要遍历半个 list，所以 LinkedList 循环遍历的时候建议使用 iterator 迭代循环，直接拿到元素。
### ArrayList 和  LinkedList
| 队列 | ArrayList | LinkedList |
| --- | --- | --- |
| 数据结构 | 数组 | 双向链表 |
| 内存 | 连续 | 不连续 |



#### 新增元素

- 头部：ArrayList > LinkedList
- 中部：ArrayList < LinkedList
- 尾部：ArrayList < LinkedList



#### 删除元素

- 头部：ArrayList > LinkedList
- 中部：ArrayList < LinkedList
- 尾部：ArrayList < LinkedList
#### 遍历元素

- for：ArrayList < LinkedList
- 迭代器：ArrayList ≈ LinkedList
## Map
## Set
