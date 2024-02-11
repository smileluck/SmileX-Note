[toc]

---


# 简介
java NIO中的Buffer用于和NIO通道进行交互。数据从通道中读取和写入到缓冲区。本质上是一块固定大小的内存区，其作用是是用来存储或传输使用。

java1.4之前，javaio最核心的点是在“数据流”

# Buffer是什么
Buffer就是一个存储Byte基本类型的线性的有限序列。
就是说，他是一个固定大小的序列数组。

Buffer底层的本质是数组，我们以ByteBuffer为例，它的底层是：
```java
final byte[] hb; 
```

## Buffer的重要属性
各属性关系：
mark<=position<=limit=capacity

### Position（位置）
- 当前操作的索引，会自动由相应的get()和put()函数更新。
- 默认为0

### limit(限制)
- 表示不能读或写的第一个索引，即从这个索引开始，后续直到capacity都不能进行操作。
- 默认为缓冲区容量(capacity)。

### capacity（容量）
- 表示缓冲区的容量，由构建时设置，不能小于0，小于0会抛出异常

### mark(标记)
- 一个标记索引。默认值是-1
- 通过调用mark()来设定mark=position；调用reset()设定position=mark。


# 进阶操作
源代码：[文件地址](https://github.com/smileluck/geek-study/blob/main/project/src/main/java/top/zsmile/nio/buffer/)

## 创建Buffer
细分来说，创建Buffer可以使用四种方式
- 直接缓冲区（DirectByteBuffer）
    - allocateDirect(int capacity)。
        - 只有ByteBuffer具备。
- 堆内存缓冲区（HeapBuffer）。
    - java对象的底层存储单位是byte字节。但是因为java中还有int,double,short等基础类型，为了简化操作，nio中都是为他们实现了一套Buffer
    - 操作方式
        - allocate(int capacity)。
        - wrap(<T> array)
        - wrap(<T> array,int offset,int length)


Buffer是一个接口，它下面有诸多实现，包括最基本的ByteBuffer和其他的基本类型封装的其他Buffer。

```java
public class CreateBufferDemo {
    public static void main(String[] args) {
        ByteBuffer allocateDirect = ByteBuffer.allocateDirect(8);
        System.out.println(allocateDirect.hasArray() + "-" + allocateDirect);

        ByteBuffer allocate = ByteBuffer.allocate(8);
        System.out.println(allocate.hasArray() + "-" + allocate);

        byte[] array = new byte[10];
        ByteBuffer wrap = ByteBuffer.wrap(array);
        System.out.println(wrap.hasArray() + "-" + wrap);

        ByteBuffer wrap1 = ByteBuffer.wrap(array, 4, 3);
        System.out.println(wrap1.hasArray() + "-" + wrap1);
    }
}

```
让我们运行一下看看输出结果。
```
java.nio.DirectByteBuffer[pos=0 lim=8 cap=8]
java.nio.HeapByteBuffer[pos=0 lim=8 cap=8]
java.nio.HeapByteBuffer[pos=0 lim=10 cap=10]
java.nio.HeapByteBuffer[pos=4 lim=7 cap=10]
```
这里需要注意的有2点：
- 当我们使用wrap(<T> array,int offset,int length)进行创建Buffer时，offset表示的是偏移量即position，而limit=offset+length，所以我们要保证limit<=capacity，不然会抛出异常。
- 我们可以看到allocateDirect()生成的ByteBuffer是DirectByteBuffer，并且不为数组。这是为什么呢？因为DirectByteBuffer是通过unsafe向操作系统申请的空间，虽然操作结构上也是一致的，但是实际存储中，并非使用的是java内部的数组。而其它三个最终都是基于数组的。

## 读写数据
```java
 public static void main(String[] args) {
        IntBuffer buffer = IntBuffer.allocate(10);
        System.out.println("create buffer：" + buffer);

        buffer.put(new int[]{1, 2, 3, 4, 5});
        System.out.println("put buffer after：" + buffer);

        buffer.put(0, 6);
        System.out.println("update buffer after：" + buffer);

        buffer.flip();
        System.out.println("flip buffer after：" + buffer);

        printBuffer(buffer);
    }
```
我们通过put写入数据，这时position会不断递增，这时我们可以通过flip切换读模式，即limit=position、position=0。

## rewind(倒带)
rewind会将position赋值为0，并清空mark，但不会更改limit的值。
```
rewind buffer position：java.nio.HeapIntBuffer[pos=2 lim=5 cap=10]
rewind buffer position&mark：java.nio.HeapIntBuffer[pos=0 lim=5 cap=10]
```
由于rewind会将mark清空，所以不要在rewind()后执行reset操作，会抛出InvalidMarkException。

## clear(清空)
clear会将buffer还原回默认状态，即position=0;limit=capacity;mark=-1;

## compact(压缩)
有时候，我们会考虑将buffer的一部分数据释放，并重新填充。为了要实现这样的需求，我们会不断反复的读写数据，但是这样性能比较差，而buffer自带的compact会很好的解决这一点。
```java
public class BufferDemo {
    public static void main(String[] args) {
        IntBuffer buffer = IntBuffer.allocate(10);
        System.out.println("create buffer：" + buffer);

        buffer.put(new int[]{1, 2, 3, 4, 5});
        System.out.println("put buffer after：" + buffer);

        buffer.put(0, 6);
        System.out.println("update buffer after：" + buffer);

        buffer.flip();
        System.out.println("flip buffer after：" + buffer);

        printBuffer(buffer);
        buffer.position(2);
        System.out.println("compact buffer before：" + buffer);
        buffer.compact();
        System.out.println("compact buffer after：" + buffer);

        printBuffer(buffer);
        System.out.println("print buffer after：" + buffer);

        buffer.position(0);
        printBuffer(buffer);
        System.out.println("print buffer after：" + buffer);

    }

    public static void printBuffer(IntBuffer buffer) {
        System.out.print("buffer print：");
        while (buffer.hasRemaining()) {
            System.out.print(buffer.get());
        }
        System.out.println();
    }
}

```
代码输出：
```
create buffer：java.nio.HeapIntBuffer[pos=0 lim=10 cap=10]
put buffer after：java.nio.HeapIntBuffer[pos=5 lim=10 cap=10]
update buffer after：java.nio.HeapIntBuffer[pos=5 lim=10 cap=10]
flip buffer after：java.nio.HeapIntBuffer[pos=0 lim=5 cap=10]
buffer print：62345
compact buffer before：java.nio.HeapIntBuffer[pos=2 lim=5 cap=10]
compact buffer after：java.nio.HeapIntBuffer[pos=3 lim=10 cap=10]
buffer print：4500000
print buffer after：java.nio.HeapIntBuffer[pos=10 lim=10 cap=10]
buffer print：3454500000
print buffer after：java.nio.HeapIntBuffer[pos=10 lim=10 cap=10]
```
这里做了几件事情：
1. 将position到limit的数据赋值到position为0的位置，即从buffer最前面开始覆盖，并丢弃了原先的数据。
2. 将position设置为被复制的数据数量。即position=limit-position，也就是说这时position指向的是缓冲区最后一位保留数据后的位置。
3. limit再次设置capacity，即整个buffer的大小，因此缓冲区可以再次填充数据。

![image](5EC73ED2BFC24A068F7496B405568432)

可以使用这种类似先入先出（FIFO）队列的方式使用buffer。尽管不是非常高效的方法，但是compact对于使buffer与从端口中读取的数据包流的同步来说，还是一种较为便捷的方法。


## duplicate(复制)
```java
public class DuplicateBufferDemo {
    public static void main(String[] args) {
        IntBuffer buffer = IntBuffer.allocate(10);
        buffer.put(new int[]{0, 1, 2, 3, 4});
        System.out.println("create buffer：" + buffer);
        printBuffer(buffer, true);

        IntBuffer duplicate1 = buffer.duplicate();
        System.out.println("duplicate buffer：" + duplicate1);
        printBuffer(duplicate1, true);

        duplicate1.put(0, 7);
        System.out.println("origin and duplicate buffer start=====");
        printBuffer(buffer, true);
        printBuffer(duplicate1, true);
        System.out.println("origin and duplicate buffer end=====");


        IntBuffer duplicate2 = buffer.asReadOnlyBuffer();
        System.out.println("asReadOnlyBuffer buffer：" + duplicate2);
        buffer.put(0, 8);
        printBuffer(duplicate2, true);

        try {
            duplicate2.put(0, 6);
        } catch (Exception e) {
            System.err.println("asReadOnlyBuffer buffer update Exception:" + e.toString());
        }

        System.out.println("slice buffer before,origin buffer status：" + buffer);
        buffer.position(3);
        System.out.println("slice buffer before,origin buffer position 3 after：" + buffer);
        IntBuffer slice = buffer.slice();
        System.out.println("slice buffer：" + slice);
        printBuffer(slice, false);

        buffer.put(3, 13);
        buffer.rewind();
        slice.put(0, 23);
        System.out.println("origin buffer:" + buffer);
        printBuffer(buffer, false);
        System.out.println("slice buffer：" + slice);
        printBuffer(slice, true);
    }

    public static void printBuffer(IntBuffer buffer, boolean needFlip) {
        if (needFlip) {
            buffer.flip();
        }
        System.out.print("buffer print：");
        while (buffer.hasRemaining()) {
            System.out.print(buffer.get() + ",");
        }
        System.out.println();
    }
}

```
输出结果：
```java
create buffer：java.nio.HeapIntBuffer[pos=5 lim=10 cap=10]
buffer print：0,1,2,3,4,
duplicate buffer：java.nio.HeapIntBuffer[pos=5 lim=5 cap=10]
buffer print：0,1,2,3,4,
origin and duplicate buffer start=====
buffer print：7,1,2,3,4,
buffer print：7,1,2,3,4,
origin and duplicate buffer end=====
asReadOnlyBuffer buffer：java.nio.HeapIntBufferR[pos=5 lim=5 cap=10]
buffer print：8,1,2,3,4,
slice buffer before,origin buffer status：java.nio.HeapIntBuffer[pos=5 lim=5 cap=10]
slice buffer before,origin buffer position 3 after：java.nio.HeapIntBuffer[pos=3 lim=5 cap=10]
slice buffer：java.nio.HeapIntBuffer[pos=0 lim=2 cap=2]
buffer print：3,4,
origin buffer:java.nio.HeapIntBuffer[pos=0 lim=5 cap=10]
buffer print：8,1,2,23,4,
slice buffer：java.nio.HeapIntBuffer[pos=2 lim=2 cap=2]
buffer print：23,4,
asReadOnlyBuffer buffer update Exception:java.nio.ReadOnlyBufferException

```
buffer提供的复制有三种方式：
- duplicate()。拷贝原buffer的position、limit、mark和capacity，他是和原先的buffer是共享数据的。
- asReadOnlyBuffer()。
    - 与duplicate类似，区别在于是只读的，并且也是与原buffer共享数据，修改原buffer也会影响到asReadOnlyBuffer。
    - 修改只读buffer，会抛出异常ReadOnlyBufferException
- slice()。与duplicate类似，区别在于他拷贝的是从原buffer的poisition到limit的部分，同时也是共享数据的。

