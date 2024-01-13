[toc]

---

# final

## 用法

- 被final修饰的类不可以被继承 
- 被final修饰的方法不可以被重写 
- 被final修饰的变量不可以被改变.如果修饰引用,那么表示引用不可变,引用指向的内容可变. 
- 被final修饰的方法,JVM会尝试将其内联,以提高运行效率 
- 被final修饰的常量,在编译阶段会存入常量池中

两个重排序规则：

1. 在构造函数内对一个final域的写入,与随后把这个被构造对象的引用赋值给一个引用变量,这两个操作之间不能重排序
2. 初次读一个包含final域的对象的引用,与随后初次读这个final域,这两个操作之间不能重排序。

# 注意

## final和static修饰

用final和static修饰的属性变量（特别是在单例模式），static里面出现trycatch时，需要throw异常，否则会编译错误。

**解释：**
当属性被static和final同时修饰时，该属性属于类属性（类常量），就是说在类被加载进内存时就需要分配内存（初始化完成）。而构造函数是在被实例化的时候才会执行，对比static代码块是在类被加载的时候执行，且只执行这一次


# 一道面试题分析
最近看面试题的时候，看到这样一道题目。觉得有意思，就分析了一下看看。
```java
public class FinalStaticClass {
    public static void main(String[] args) {
        System.out.println(Price.P.Price);//result:-2.7
    }
}

class Price {
    /**
     * method1: give apple add final, result:17.3;
     * method2: Swap 18 to 19, result:17.3;
     */
    static Price P = new Price(2.7);
    static double apple = 20;
    double Price;

    public Price(double orange) {
        Price = apple - orange;
    }
}
```
我们先看一下执行结果。输出2.7

为什么不是17.3呢？让我们先执行一下以下两个指令，生成反汇编文件：
```shell
javac .\FinalStaticClass
javap -c -verbose .\FinalStaticClass.class > FinalStaticClass.txt
javap -c -verbose .\Price.class >Price.txt
```

## 分析

让我们查看一下Price.txt
```
// from 75 to 82
0: new           #4                  // class top/zsmile/jvm/base/Price
 3: dup
 4: ldc2_w        #5                  // double 2.7d
 7: invokespecial #7                  // Method "<init>":(D)V
10: putstatic     #8                  // Field P:Ltop/zsmile/jvm/base/Price;
13: ldc2_w        #9                  // double 20.0d
16: putstatic     #2                  // Field apple:D
19: return

// Constant Pool
#1 = Methodref          #11.#25        // java/lang/Object."<init>":()V
#2 = Fieldref           #4.#26         // top/zsmile/jvm/base/Price.apple:D
#3 = Fieldref           #4.#27         // top/zsmile/jvm/base/Price.Price:D
#4 = Class              #28            // top/zsmile/jvm/base/Price
#5 = Double             2.7d
#7 = Methodref          #4.#29         // top/zsmile/jvm/base/Price."<init>":(D)V
#8 = Fieldref           #4.#30         // top/zsmile/jvm/base/Price.P:Ltop/zsmile/jvm/base/Price;
#9 = Double             20.0d
```

这里可以看出执行过程是：
1. new Price
2. 将double 2.7d 入栈
3. 执行类初始化，构造方法
4. 将double 2.7d 出栈，存入 Price.P 
5. 将double 20.0d 入栈
6. 将double 20.0d 出栈，存入 Price.apple 

我们可以看出，在执行构造方法计算 Price(D)的时候，apple还没有赋值20。我们知道对于数字基本类型的时候，默认值都为0。于是在执行构造函数的时候，相当于：0-2.7，所以得出结果-2.7。

这是因为在随后的类加载的初始化阶段，由于static代码块执行顺序是由字段在源文件中出现的顺序决定的，所以会先执行new Price(2.7)，分配对象空间并对其做初始化，此时apple的值为0，所以最终结果是-2.7。

那么我们有什么方法解决这个问题？（暂时想到2种）
1. 交换P和apple
2. 给apple 添加关键字final。

## 方法1（交换p和apple）
交换后，Price代码如下：
```java
class Price {
    static double apple = 20;
    static Price P = new Price(2.7);
    double Price;

    public Price(double orange) {
        Price = apple - orange;
    }
}

// result : 17.3

```


老规矩生成汇编代码：

```shell
javac .\FinalStaticClass
javap -c -verbose .\Price.class >Price2.txt
```

让我们查看一下Price.txt
    
```
// from 75 to 82
 0: ldc2_w        #4                  // double 20.0d
 3: putstatic     #2                  // Field apple:D
 6: new           #6                  // class top/zsmile/jvm/base/Price
 9: dup
10: ldc2_w        #7                  // double 2.7d
13: invokespecial #9                  // Method "<init>":(D)V
16: putstatic     #10                 // Field P:Ltop/zsmile/jvm/base/Price;
19: return
```
这里我们可以看到：
1. 将20赋值给apple
2. new Price。dup是交换栈的操作，可以看做将栈顶空出。
3. 将2.7赋值给P。
4. 执行构造函数，


## 方法2（给apple加上final关键字）
添加后，代码如下：
```java

class Price {
    /**
     * method1: give apple add final keyword, result:17.3;
     * method2: Swap 18 to 19, result:17.3;sfsdfsdf
     */
    static Price P = new Price(2.7);
    final static double apple = 20;
    double Price;

    public Price(double orange) {
        Price = apple - orange;
    }
}
```

```shell
javac .\FinalStaticClass
javap -c -verbose .\Price.class >Price3.txt
```

还是看Price文件。不过这次看两个地方
```
// from 43 to 46
static final double apple;
descriptor: D
flags: ACC_STATIC, ACC_FINAL
ConstantValue: double 20.0d

// from 74 to 80
0: new           #2                  // class top/zsmile/jvm/base/Price
3: dup
4: ldc2_w        #6                  // double 2.7d
7: invokespecial #8                  // Method "<init>":(D)V
10: putstatic     #9                  // Field P:Ltop/zsmile/jvm/base/Price;
13: return

```
上面代码我们可以得出：apple已经有初始值了，值为20；static代码中只有存入p和执行构造函数。
也就是说现在构造函数的执行，是 20-2.7=17.3。

因为在类加载过程中，会将类中的字面量存入常量池中。**而在遇到final关键字时，在编译时，会为该静态属性赋予ConstantValue属性，ConstantValue的作用是使JVM自动为静态变量赋值。**

类变量，有两种方式赋值：
1. 类构造器方法<clinit> 
2. ConstantValue。将会在类加载的准备阶段被赋予所指定的值(这是final static字段必须赋值的原因)



---
老规矩写的不清晰的地方，可以指出，我会不定时更新和修改。一起加油.

所有的伟大，都来自于一个勇敢的开始。




