[toc]

---

# java 的四种引用

## 强引用

强引用是平常中使用最多的引用，强引用在程序内存不足（OOM）的时候也不会被回收。

- 使用方式：

```java
String str = new String("123");
```

## 软引用

软引用在程序内存不足时，会被回收。

- 使用方式：

```java
// 注意：wrf这个引用也是强引用，它是指向SoftReference这个对象的，
// 这里的软引用指的是指向new String("str")的引用，也就是SoftReference类中T
SoftReference<String> wrf = new SoftReference<String>(new String("str"));
```

- 使用场景 ：创建缓存的时候，创建的对象放进缓存中，当内存不足时，JVM就会回收早先创建的对象

## 弱引用

弱引用就是只要JVM垃圾回收器发现了它，就会将之回收。

- 使用方式： 

```java
WeakReference<String> wrf = new WeakReference<String>(str);
```

- 可用场景： Java源码中的 java.util.WeakHashMap 中的 key 就是使用弱引用，我的理解就是， 一旦我不需要某个引用，JVM会自动帮我处理它，这样我就不需要做其它操作。比较典型的就是`ThreadLocal`

## 虚引用

虚引用的回收机制跟弱引用差不多，但是**它被回收之前，会被放入 ReferenceQueue 中**。**注意，其它引用是被JVM回收后才被传入 ReferenceQueue 中的**。由于这个机制，所以虚引用大多被用于引用销毁前的处理工作。还有就是，虚引用创建的时候，必须带有 ReferenceQueue 。

- 使用例子：

```java
PhantomReference<String> prf = new PhantomReference<String>(new String("str"), 
new ReferenceQueue<>());
```

- 可用场景：对象销毁前的一些操作，比如说资源释放等。 Object.finalize() 虽然也可以做这类动作，但是这个方式即不安全又低效

# 容器

## HashMap

### **HashMap** 的长度为什么是 **2** 的 **N** 次方呢？

为了能让 HashMap 存数据和取数据的效率高，尽可能地减少 hash 值的碰撞，也就是说尽量把数据能均匀的分配，每个链表或者红黑树长度尽量相等。 

我们首先可能就会想到取模操作(%) 。取余（%）操作中如果除数是 2 的幂次，则等价于与其除数减一的与（&）操作（也就是说hash % length == hash &(length - 1) 的前提是 length 是 2 的 n 次方）。并且，采用二进制位操作 & ，相对于 % 能够提高运算效率。