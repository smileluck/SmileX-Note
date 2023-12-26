[toc]

---

# 概述

- 面向对象编程思想
  - 一切事物都是对象。
  - 对象是其属性和操作的封装体。
  - 强调的是对象，必须通过对象的形式来做事，一般情况下会比较复杂。
- 函数编程思想
  - 函数强调的是按照输入量、输出量，使用输入量计算得出输出量。【拿什么东西做什么事情】
  - 同样执行线程任务，使用函数编程思想，可以直接通过传递一段代码给线程对象执行，不需要创建任务对象。 



# 函数式接口

`Functional interface` 接口也称 SAM 接口，即Single Abstract Method interfaces，**有且只有一个抽象方法的接口，但可以有多个非抽象方法的接口。**

- 在 Java 8 中专门有一个包放函数式接口`java.util.function`，该包下的所有接口都有 `@FunctionalInterface` 注解，提供函数式接口。

- 在其他包中也有函数式接口，其中一些没有 `@FunctionalInterface` 注解，但是只要符合函数式接口的定义就是函数式接口，与是否有 `@FunctionalInterface` 注解无关，注解只是在**编译时起到强制规范定义**的作用。其在 Lambda 表达式中有广泛的应用。例如 `Runnable` 接口，也是没有使用 `@FunctionalInterface` 注解

# 什么是lambda

`lambda` 表达式，也可以称之为闭包或者匿名函数，是 `Java8` 引入的新特性。

`lambda` 允许把函数作为一个方法的参数（函数作为参数传入方法中执行）



# lambda 使用前提

1. 必须有相应的**函数式接口**。函数接口是指有且只有一个抽象方法的接口。

2. **类型推断机制**。在上下文信息足够的情况下，编译器可以推断出参数表的类型，而不需要显示指定。也就是方法的参数和局部变量的类型必须为 `lambda`  对应的接口类型，才能使用  `lambda`  表达式表示该接口的实例。



# 语法

## 简写

```java
(parameters) -> expression
    
(parameters) ->{ statements; }
```

重要特征：

- **可选类型声明：**不需要声明参数类型，编译器可以统一识别参数值。
- **可选的参数圆括号：**一个参数无需定义圆括号，但多个参数需要定义圆括号。
- **可选的大括号：**如果主体包含了一个语句，就不需要使用大括号。
- **可选的返回关键字：**如果主体只有一个表达式返回值则编译器会自动返回值，大括号需要指定表达式返回了一个数值。

## 方法引用

如果我们使用`lambda` 表达式的时候，如果要执行的表达式只是调用一个类已有的方法，那么就可以用方法引用的方式来替代。

重要特种：

- **引用静态方法：**类名::静态方法名。如果静态方法需要传参数，只要确保参数是一一对应的，编译器会自行推断出来。
- **引用对象方法**：对象引用::方法名。如果执行的类是调用 `lambda` 表达式所在的类的方法时，可以才作用一下写法 `this::方法名`。
- **引用构造方法**：类名::new。

# 实战

## 替代匿名内部类

```java
class LambdaTest {
    public static void main(String[] args) {
        // 匿名内部类, Runnable
        Thread thread = new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println("线程输出");
            }
        });

        // lambda
        Thread thread1 = new Thread(() -> {
            System.out.println("lambda线程输出");
        });
    }
}
```

### 原理分析

那么用 `lambda` 来替代匿名内部类有什么差别呢？我们通过编译一下来看看

```powershell
# 编译class文件
javac -encoding UTF-8 LambdaTest.java
```

可以看见产生了两个class文件。

- LambdaTest.class

  ```java
  
  import top.zsmile.test.basic.lambda.LambdaTest.1;
  
  public class LambdaTest {
      public LambdaTest() {
      }
  
      public static void main(String[] var0) {
          new Thread(new 1());
          new Thread(() -> {
              System.out.println("lambda线程输出");
          });
      }
  }
  ```

- LambdaTest$1.class。这是一个匿名内部类。

  ```java
  final class LambdaTest$1 implements Runnable {
      LambdaTest$1() {
      }
  
      public void run() {
          System.out.println("线程输出");
      }
  }
  ```

那么我们来看看这个代码字节码，看看程序内部是怎么运行的。

```powershell
javap -c “.\LambdaTest.class”

javap -c “.\LambdaTest$1.class”
```

这里我们只看主文件的字节码。

```java
Compiled from "LambdaTest.java"
public class top.zsmile.test.basic.lambda.LambdaTest {
  public top.zsmile.test.basic.lambda.LambdaTest();
    Code:
       0: aload_0
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
       4: return

  public static void main(java.lang.String[]);
    Code:
       0: new           #2                  // class java/lang/Thread
       3: dup
       4: new           #3                  // class top/zsmile/test/basic/lambda/LambdaTest$1
       7: dup
       8: invokespecial #4                  // Method top/zsmile/test/basic/lambda/LambdaTest$1."<init>":()V
      11: invokespecial #5                  // Method java/lang/Thread."<init>":(Ljava/lang/Runnable;)V
      14: astore_1
      15: new           #2                  // class java/lang/Thread
      18: dup
      19: invokedynamic #6,  0              // InvokeDynamic #0:run:()Ljava/lang/Runnable;
      24: invokespecial #5                  // Method java/lang/Thread."<init>":(Ljava/lang/Runnable;)V
      27: astore_2
      28: return
}
```

由这个字节码信息可以分析出：

1. 根据 `4: new           #3` 这行可以看出，这里使用了 `LambdaTest$1.class` 类，`new` 了一个新的对象。

2. 根据 `19: invokedynamic #6,  0` 可以看出，`lambda` 是通过 `invodedynamic` 实现的，并且不会生成新的类和对象。 `invodedynamic` 是 `Jvm` 提供的一种指令语法。在这里作用是创建并调用 `Runnable` 接口 run 方法。

   而新增的 `invokedynamic` 指令，配合新增的方法句柄（Method Handles，它可以用来描述一个跟类型A无关 的方法m的签名，甚至不包括方法名称，这样就可以做到我们使用方法m的签名，但是直接执行的时候调用 的是相同签名的另一个方法b），可以在运行时再决定由哪个类来接收被调用的方法。在此之前，只能使用反射来实现类似的功能。该指令使得可以出现基于`Jvm`的动态语言，让 `Jvm` 更加强大。而且在 `Jvm` 上实现动态调用机制，不会破坏原有的调用机制。这样既很好的支持了Scala、Clojure这些JVM上的动态语言，又可以支持代码里的动态lambda表达式。

   > 简单来说就是以前设计某些功能的时候把做法写死在了字节码里，后来想改也改不了了。
   > 所以这次给lambda语法设计翻译到字节码的策略是就用invokedynamic来作个弊，把实际的翻译策略隐 藏在Jdk 库的实现里（MetaFactory）可以随时改，而在外部的标准上大家只看到一个固定的 invokedynamic。

3. 既然 `lambda` 不会生成新的类，那么这里的 `this` 指向外部类的。简单上个demo。可以看出使用匿名内部类时，`this`  指向当前类，而 `lambda` 没有生成新的类，所以指向的外部类，所以无法在静态 `static`方法中使用，因为 `static` 方法没有实例对象。

   ```java
   
   public class LambdaTest {
       public void run(){
           new Thread(new Runnable() {
               @Override
               public void run() 
                   System.out.println(this);
                   System.out.println("线程输出");
               }
           }).start();
   
           // lambda
           new Thread(() -> {
               System.out.println(this);
               System.out.println("lambda线程输出");
           }).start();
       }
   
       public static void main(String[] args) {
           new LambdaTest().run();
       }
   }
   
   // 输出结果
   // top.zsmile.test.basic.lambda.LambdaTest$1@1232cc0
   // 线程输出
   // top.zsmile.test.basic.lambda.LambdaTest@7e8572a1
   // lambda线程输出
   
   ```

## 为什么不能修改外部变量

`lambda` 表达式的局部变量可以不用声明为 `final` ，但是必须不可被 `lambda` 内部的代码修改（即具有隐性的 `final` ）。可是我们有时又能修改局部变量的值，这是为什么呢？

其实这和变量的类型和 `final` 的作用有关联，如果我们使用一些常用的类型。例如：

- 基础类型：int,float等
- 简单引用类型：String，Integer等
- 复杂对象引用
- 静态类型引用

上个简单的例子说明一下；

```java

public class LambdaTest {
    public void run() {
        String fff = "";
        new Thread(() -> {
            this.name = "456";
//                fff="456";
            //Variable used in lambda expression should be final or effectively final
            System.out.println(this.name);
            System.out.println("lambda线程输出");
        }).start();
    }
}
```

这里提到了 final or effectively final。

>  **对于一个变量，如果没有给它加final修饰，而且没有对它的二次赋值，那么这个变量就是effectively final(有效的不会变的)，如果加了final那肯定是不会变了哈。** 

那么如果我们想要在内部更改外部的属性，有什么方式？

1. 使用对象的属性
2. 使用引用的属性
3. 使用静态的属性


# 总结

`lambda` 的优点：

- 简化了部分的写法，使代码更为简洁紧凑
- 减少匿名内部类的创建，节省内存开支
- 不需要记忆所使用的接口和抽象函数

`lambda` 的缺点：

- 易读性较差，阅读代码的人需要熟悉 `lambda` 表达式和抽象函数中参与的类型。
- 不具备匿名内部类可扩展的特性

# 引用

- https://www.oracle.com/java/technologies/java8.html
- https://mp.weixin.qq.com/s?__biz=MjM5MDE0Mjc4MA==&mid=2651144680&idx=5&sn=cea5cf2b8748e4e94894558914bf26ec&chksm=bdb8b9bb8acf30ad90011a970dd18acbe2be555609d348601d76d566e4be7e4aad2b6c719e7a&scene=27#wechat_redirect
- https://www.infoq.cn/article/LlrBgvdmYPGNsVDOZuCZ
- https://xie.infoq.cn/article/a7a4b91228653c64f866367ca
- https://mp.weixin.qq.com/s?__biz=MzAxODcyNjEzNQ==&mid=2247550085&idx=5&sn=d13402dde3e283ce0140515d6347873f&chksm=9bd3a91daca4200bae75e718a36fc5aaa85f19d03f6436f1c64f86b6d1848051c4a9f6414754&scene=27#wechat_redirect
- https://xie.infoq.cn/article/b47fa71dff45feb333a31c6dc
- https://blog.csdn.net/qq_31635851/article/details/116484594

