[toc]

---

# 前言

栈上分配是java虚拟机提供的一项优化技术，基本思想是，对于那些线程私有的对象（这里指不可能被其他线程访问的对象），可以将它们分配在栈上，而不是分配在堆上。

**好处是可以在函数调用结束后自行销毁，不需要垃圾回收器的介入，从而提高性能。**

# 逃逸分析

栈上分配的技术基础是 **逃逸分析**。逃逸分析的目的是**判断对象的作用域是否有可能逃逸出函数体**。

下面就是一个逃逸对象。因为 静态变量u有可能被任何线程访问。

```java
private static User u;

public static void alloc(){
    u = new User();
    u.id = 1;
    u.name = "test";
}
```

下面就是一个非逃逸对象。因为 变量u 以局部变量存在，并且没有做公开/返回等操作。

```java
public static void alloc(){
    User u = new User();
    u.id = 1;
    u.name = "test";
}
```

对于这种情况，虚拟机就有可能将User分配到栈上，而不在堆上。



## 测试

测试代码：

```java
public class OnStackTest {
    public static class User {
        public int id = 0;
        public String name = "";
    }

    public static void alloc() {
        User u = new User();
        u.id = 1;
        u.name = "test";
    }

    public static void main(String[] args) {
        long b = System.currentTimeMillis();
        for (int i = 0; i < 100000000; i++) {
            alloc();
        }
        long e = System.currentTimeMillis();
        System.out.println(e - b);
    }
}

```

该代码执行了1一次alloc()方法创建对象，由于User对象需要占用16字节的空间，因此累计分配空间将达到1.5G。如果堆空间小于这个值，就必然发生GC。使用参数调试一下。

```shell
-server -Xmx10m -Xms10m -XX:+DoEscapeAnalysis -XX:+PrintGC -XX:-UserTLAB -XX:+EliminateAllocations
```

单独标签的作用可以查看启动参数说明。

这里程序执行后，会没有任何形式的GC输出。说明在执行过程中，分配过程被优化。

如果关闭逃逸分析或者标量替换中任何一个，再次执行程序，会输出大量的GC日志，说明**栈上分配依赖逃逸分析和标量替换的实现**。

# 总结

对于大量的**零散小对象**，栈上分配提供了一种很好的对象分配优化策略，栈上分配速度快，并可以有效避免垃圾回收带来的负面影响（STW），但由于**栈空间较小，因此对于大对象也不适合在栈上分配**

