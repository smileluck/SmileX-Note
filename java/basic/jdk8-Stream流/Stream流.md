[toc]

---

# 什么是Stream

`Stream` 是 `JDK 8` 以后新提供的流式API操作， 侧重对于源数据计算能力的封装，并且支持**串行与并行**两种操作方式 。

`Stream` 流操作可以分为三种：

1. 创建 `Stream` 流
2. 中间操作
3. 终端操作（会结束 `Stream` 流）

# 创建方式

`Stream` 创建方式主要有几种方式： 	

1. Collection.stream()
2. Array.stream(T array)
3. Stream的静态方法
   1. Stream.of
   2. Stream.iterate
   3. Stream.generate

创建的 `Stream` 流**默认都是串行序列**。

- 可以通过 `Stream.isParallel` 来判断是否是并行流。
- 通过 `Stream.parallel` 可以转为并行流。

## Collection.stream()

```java
List<String> array = Arrays.asList("a", "b", "c");
Stream<String> stream = array.stream();
```

## Array.stream(T array)

```java
int[] arr = {1, 2, 3, 4};
IntStream stream1 = Arrays.stream(arr);
```

## Stream.of

```java
Stream<String> q = Stream.of("q", "w", "e");
```

## Stream.iterate

```java
Stream<Integer> iterate = Stream.iterate(0, integer -> integer + 1);
```

## Stream.generate

```java
Stream<Double> limit = Stream.generate(Math::random).limit(10);
```

# 操作分类

在 `Stream` 中，流操作的分为 `中间操作` 和 `终端操作`。

- 中间操作：对流中的数据进行各种各样的处理，可以连续操作，每个操作的返回值都是 `Stream` 对象。又可以分为以下两种：
  - 有状态：该操作只有拿到所有元素才能正常操作下去。例如去重方法`distinct` ，必须拿到所有的值，才能判断当前值是否重复。
  - 无状态：元素的处理不受之前元素的影响。即不会记录元素状态，只处理当前元素。
- 终端操作：执行结束后，会关闭这个流，流中的数据都会销毁。
  - 短路：获取到预期的结果时，就会返回，不会将所有数据都处理。
  - 非短路：会将所有数据都按预期操作一遍，才能获取到最终结果。

所以其特性如下：

1. 不存储数据，按照特定的规则对数据进行计算处理，一般会输出结果；
2. 不改变数据源，通常会产生一个新的集合或值；
3. **具有延迟特性，只有调用终端操作，中间操作才会执行**。

<table>
	<tr>
    	<td>操作分类</td>
        <td>特征</td>
		<td>方法</td>
    </tr>    
    <tr>
        <td rowspan="2">中间操作</td>
        <td>操作分类</td>
        <td>方法</td>
    </tr> 
    <tr>
        <td>操作分类</td>
        <td>方法</td>
    </tr>
    <tr>
        <td rowspan="2">终端操作</td>
        <td>操作分类</td>
        <td>方法</td>
    </tr> 
    <tr>
        <td>操作分类</td>
        <td>方法</td>
    </tr>
</table>

# 注意事项

## 终端操作后，不能再继续操作

 **一旦一个 Stream 执行了终端操作之后，后续便不可以再读这个流执行其他的操作**了 。

```java
@Test
public void Streamttt() {
    Stream<User> stream = userList.stream();
    log.info("终端操作 count => {}", stream.count());
    log.info("再执行终端操作 foreach");
    stream.forEach(System.out::println);
}
```

输出结果

```java
18:16:53.449 [main] INFO top.zsmile.test.basic.stream.StreamTest - 终端操作 count => 3
18:16:53.455 [main] INFO top.zsmile.test.basic.stream.StreamTest - 再执行终端操作 foreach

java.lang.IllegalStateException: stream has already been operated upon or closed

```

 因为 stream 已经被执行`count()`终端方法了，所以对 stream 再执行`foreach`方法的时候，就会报错**`stream has already been operated upon or closed`**。 

## 并行Stream

使用并行流，可以有效利用计算机的硬件资源。并行流通过将整个Stream划分为多个片段任务，然后对各个分片进行处理后，最后将其结果合并。

并行流类似多线程并行处理，所以多线程场景下的问题，都会存在，比如锁冲突等，必须要保证**`线程安全`**

# Stream API

## 数据准备

```java

@Data
public class User {
    private String name;
    private Integer age;
    private Integer score;
    private List<String> roles;

    public static User of(String name, Integer age, Integer score, List<String> roles) {
        User user = new User();
        user.setAge(age);
        user.setScore(score);
        user.setName(name);
        user.setRoles(roles);
        return user;
    }

    public static User of(String name, Integer age, List<String> roles) {
        User user = new User();
        user.setAge(age);
        user.setName(name);
        user.setRoles(roles);
        return user;
    }
}


private static List<User> userList;
private static List<User> userList2;

@Before
private void init() {
    userList = new ArrayList<>();
    userList.add(User.of("张三", 18, Arrays.asList("ADMIN", "USER")));
    userList.add(User.of("李四", 19, Arrays.asList("TEMP:USER")));
    userList.add(User.of("王五", 20, Arrays.asList("USER")));
    
    userList2 = new ArrayList<>();
    userList2.add(User.of("张三", 18, 55, Arrays.asList("ADMIN", "USER")));
    userList2.add(User.of("李四", 19, 100, Arrays.asList("TEMP:USER")));
    userList2.add(User.of("甲", 30, 100, Arrays.asList("TEMP:USER")));
    userList2.add(User.of("乙", 35, 90, Arrays.asList("TEMP:USER")));
    userList2.add(User.of("王五", 20, 80, Arrays.asList("USER")));
    userList2.add(User.of("丁", 20, 90, Arrays.asList("TEMP:USER")));
}
```

## 遍历

在 `Stream` 中，`peek` 和 `foreach`  都可以对元素进行遍历处理。

**但是 `peek` 属于中间方法，而 `foreach` 属于终端方法。**这意味着 `peek` 作为中途的一个操作步骤，没有办法直接执行得到结果，后面还必须有其他终端方法才会执行。而`foreach`  作为无返回值的终端方法，则可以执行相关操作。

### foreach

```java
log.info("start foreach");
userList.stream().forEach(System.out::println);
log.info("end foreach");
```

输出结果

```javascript
16:58:42.666 [main] INFO top.zsmile.test.basic.stream.StreamTest - start foreach
User(name=张三, age=18, roles=[ADMIN, USER])
User(name=李四, age=19, roles=[TEMP:USER])
User(name=王五, age=20, roles=[USER])
16:58:42.709 [main] INFO top.zsmile.test.basic.stream.StreamTest - end foreach
```

### peek

```java
log.info("start peek");
userList.stream().peek(System.out::println);
log.info("end peek");

log.info("start peek -> count");
userList.stream().peek(System.out::println).count();
log.info("end peek -> count");
```

输出结果

```javascript
17:09:23.759 [main] INFO top.zsmile.test.basic.stream.StreamTest - start peek
17:09:23.760 [main] INFO top.zsmile.test.basic.stream.StreamTest - end peek
17:09:23.760 [main] INFO top.zsmile.test.basic.stream.StreamTest - start peek -> count
User(name=张三, age=18, roles=[ADMIN, USER])
User(name=李四, age=19, roles=[TEMP:USER])
User(name=王五, age=20, roles=[USER])
17:09:23.763 [main] INFO top.zsmile.test.basic.stream.StreamTest - end peek -> count
```

## 过滤 filter

`Filter` 条件过滤，仅**保留**符合符合条件的数据。因为 `filter` 接收的是一个 `Predicate` 接口，该接口的 `boolean test(T t)` 方法对给定的参数判断后，返回 `Boolean` 。

```java
/**
  * 过滤年龄小于19岁的用户
  */
@Test
public void filter() {
    log.info("原数据：");
    userList.stream().forEach(System.out::println);
    log.info("过滤后数据：");
    userList.stream().filter(item -> item.getAge() >= 19).forEach(System.out::println);
}
```

输出结果：

```javascript
17:33:34.739 [main] INFO top.zsmile.test.basic.stream.StreamTest - 原数据：
User(name=张三, age=18, roles=[ADMIN, USER])
User(name=李四, age=19, roles=[TEMP:USER])
User(name=王五, age=20, roles=[USER])
17:33:34.787 [main] INFO top.zsmile.test.basic.stream.StreamTest - 过滤后数据：
User(name=李四, age=19, roles=[TEMP:USER])
User(name=王五, age=20, roles=[USER])
```

## 去重 distinct

 去除集合中重复的元素，该方法没有参数，去重的规则与 HashSet 相同。 

```java
@Test
public void distinct() {
    List<String> strings = Arrays.asList("A", "B", "C", "D", "A", "B");
    log.info("原数据");
    strings.stream().forEach(System.out::println);
    log.info("去重后数据");
    strings.stream().distinct().forEach(System.out::println);
}
```

## 排序 sorted

对集合中的数据进行排序，有默认排序和自定义排序。

- 默认排序是按照基本类型排序，字符串按照字典升序，数字按照大小升序
- 自定义排序，接收一个 `Comparator` 接口，该接口的`int compare(T o1, T o2);` 接收两个参数，将两个数比较的差额进行增减后，返回一个 `Integer`，如果为负数降序，正数升序。

```java
@Test
public void sorted() {
    List<Integer> list = Arrays.asList(1, 5, 6, 3, 4, 8, 48);
    log.info("基本类型Integer默认排序后数据：");
    list.stream().sorted().forEach(System.out::println);

    List<String> list2 = Arrays.asList("a", "cc", "b", "e", "s", "m");
    log.info("基本类型String默认排序后数据：");
    list2.stream().sorted().forEach(System.out::println);

    log.info("自定义排序后数据：");
    userList.stream().sorted((o1, o2) -> {
        System.out.println(o2.getAge() + o1.getAge());
        return o2.getAge() + o1.getAge();
    }).forEach(System.out::println);
}
```

输出结果：

```javascript
18:18:53.594 [main] INFO top.zsmile.test.basic.stream.StreamTest - 基本类型Integer默认排序后数据：
1
3
4
5
6
8
48
18:18:53.654 [main] INFO top.zsmile.test.basic.stream.StreamTest - 基本类型String默认排序后数据：
a
b
cc
e
m
s
18:18:53.655 [main] INFO top.zsmile.test.basic.stream.StreamTest - 自定义排序后数据：
18:18:53.657 [main] INFO top.zsmile.test.basic.stream.StreamTest - 排序比较值:37
18:18:53.660 [main] INFO top.zsmile.test.basic.stream.StreamTest - 排序比较值:39
User(name=张三, age=18, roles=[ADMIN, USER])
User(name=李四, age=19, roles=[TEMP:USER])
User(name=王五, age=20, roles=[USER])
```

### 扩展-Comparator

> 关于Comparator的使用

`Comparator` 的 `thenComparing` 用在 `comparing` 及相关方法之后, 是以上次排序为结果进行再排序。 

`thenComparing` 和 `comparing` 的参数相同。

- public static <T, U extends Comparable<? super U>> Comparator<T> comparing(Function<? super T, ? extends U> keyExtractor)
- public static <T, U> Comparator<T> comparing( Function<? super T, ? extends U> keyExtractor, Comparator<? super U> keyComparator)
- 这里有两个参数，第一个接收`Function` 函数，第二个参数为排序方式。

`Java8` 提供了以下几种常见的排序方式：

| 方法名       | 含义                                                         |
| ------------ | ------------------------------------------------------------ |
| naturalOrder | 自然顺序，正序排列                                           |
| reverseOrder | 自然顺序，倒序排列                                           |
| nullsFirst   | 自然顺序排序，如果有null，则null在最前。<br />注意：如果字符串类型为空，则对该字段进行排序时，会抛出异常。 |
| nullsLast    | 与nullsFirst类似，但null值会排在最后                         |

```java
@Test
public void sorted2() {
    log.info("扩展：Comparator。");
    log.info("先按照年龄正序，再按照成绩排序倒叙");
    userList2.stream().sorted(Comparator.comparing(User::getAge).thenComparing(User::getScore, Comparator.reverseOrder())).forEach(System.out::println);

    log.info("使用reversed()方法");
    userList2.stream().sorted(Comparator.comparing(User::getAge).thenComparing(User::getScore).reversed()).forEach(System.out::println);

}
```

输出结果：

```java
20:19:51.251 [main] INFO top.zsmile.test.basic.stream.StreamTest - 扩展：Comparator。
20:19:51.255 [main] INFO top.zsmile.test.basic.stream.StreamTest - 先按照年龄正序，再按照成绩排序倒叙
User(name=张三, age=18, score=55, roles=[ADMIN, USER])
User(name=李四, age=19, score=100, roles=[TEMP:USER])
User(name=丁, age=20, score=90, roles=[TEMP:USER])
User(name=王五, age=20, score=80, roles=[USER])
User(name=甲, age=30, score=100, roles=[TEMP:USER])
User(name=乙, age=35, score=90, roles=[TEMP:USER])
20:19:51.319 [main] INFO top.zsmile.test.basic.stream.StreamTest - 使用reversed()方法
User(name=乙, age=35, score=90, roles=[TEMP:USER])
User(name=甲, age=30, score=100, roles=[TEMP:USER])
User(name=丁, age=20, score=90, roles=[TEMP:USER])
User(name=王五, age=20, score=80, roles=[USER])
User(name=李四, age=19, score=100, roles=[TEMP:USER])
User(name=张三, age=18, score=55, roles=[ADMIN, USER])
```

#### 注意事项

这里需要注意 `reverse()` 和 `Comparator.reverseOrder()` 的区别

1. `reverse()` 是将排序结果反转，而 `Comparator.reverseOrder()` 是直接进行排序。
2. 建议使用 `Comparator.reverseOrder()` ，更容易理解，并符合排序逻辑。

## 限制和跳过 limit&skip

- limit：截取数据流头 n 个数据
- skip：跳过数据流头 n 个数据

```java
@Test
public void limitAndSkip() {
    log.info("原数据");
    userList.stream().forEach(System.out::println);
    log.info("取前两个");
    userList.stream().limit(2).forEach(System.out::println);
    log.info("跳过第一个");
    userList.stream().skip(1).forEach(System.out::println);
    log.info("获取中间一个");
    userList.stream().skip(1).limit(1).forEach(System.out::println);
}
```

## 映射

`stream` 流中的映射都是属于中间操作，分为两类：

1. map：map对集合中的每个类进行操作，一般用于类型转换和结果映射，某种程度上，和`foreach` 效果相近。
2. flatMap： 与 `map` 相同的是对集合中的每个数据进行操作；不同的是，`flatMap` 可以将每个数据转换为任意数量个的输出值，即**扁平化**。

### map

```java
@Test
public void map() {
    log.info("原数据");
    userList.stream().forEach(System.out::println);

    log.info("将 User 转化为 String ，值来源于User::getName");
    userList.stream().map(User::getName).forEach(System.out::println);
}

```

输出结果

```javascript
20:53:17.999 [main] INFO top.zsmile.test.basic.stream.StreamTest - 原数据
User(name=张三, age=18, roles=[ADMIN, USER])
User(name=李四, age=19, roles=[TEMP:USER])
User(name=王五, age=20, roles=[USER])
20:53:18.091 [main] INFO top.zsmile.test.basic.stream.StreamTest - 将 User 转化为 String ，值来源于User::getName
张三
李四
王五
```

### flatMap

```java
@Test
public void flatMap() {
    log.info("原数据");
    userList.stream().forEach(System.out::println);

    log.info("将 User 转化为 String ，值来源于User::getName");
    log.info("使用flatMap 重复上面操作");
    userList.stream().flatMap(item -> Stream.of(item.getName())).forEach(System.out::println);
}

```

输出结果

```javascript
20:53:18.093 [main] INFO top.zsmile.test.basic.stream.StreamTest - 使用flatMap 重复上面操作
张三
李四
王五
```

### 差异

下面这个案例可以看出，`flatMap` 会将数组进行扁平化输出。 如果不使用 `flatMap` 返回值为 `List<List<String>>` 类型，使用之后为 `List<String>`。 

```java
@Test
public void mapAndFlatMap() {
    log.info("原数据");
    userList.stream().forEach(System.out::println);

    log.info("体现 Map 和 flatMap 差异");
    userList.stream().map(item -> item.getRoles()).forEach(System.out::println);
    userList.stream().flatMap(item -> item.getRoles().stream()).forEach(System.out::println);
}
```

输出结果

```javascript
20:56:47.573 [main] INFO top.zsmile.test.basic.stream.StreamTest - 原数据
User(name=张三, age=18, roles=[ADMIN, USER])
User(name=李四, age=19, roles=[TEMP:USER])
User(name=王五, age=20, roles=[USER])
20:56:47.630 [main] INFO top.zsmile.test.basic.stream.StreamTest - 体现 Map 和 flatMap 差异
[ADMIN, USER]
[TEMP:USER]
[USER]
ADMIN
USER
TEMP:USER
USER
```

## 匹配 match

- matchAll：集合中的所有元素是否都满足条件，才返回 True
- matchAny：集合中的元素是否有满足条件，才返回 True
- matchNone：集合中的所有元素都不满足条件，才返回 True

```java
@Test
public void match() {
    log.info("原数据");
    userList.stream().forEach(System.out::println);

    log.info("判断是否都已成年,{}", userList.stream().allMatch(item -> item.getAge() >= 18));
    log.info("判断是否有19岁以上的,{}", userList.stream().anyMatch(item -> item.getAge() >= 19));
    log.info("判断是未成年的,{}", userList.stream().noneMatch(item -> item.getAge() < 18));
}
```
## Optional 

### 操作与说明

#### 创建Optional

创建`Optional` 主要有三种方式：

1. Optional.empty。创建一个空的 `Optional`
2. Optional.of(T t)。创建一个存在T类型值的 `Optional`，注意不能传入null值，否则会报 空指针异常。
3. Optional.ofNullable(T t)。创建一个存在T类型值的 `Optional`，允许传入 `null` 值，会返回一个`Optional.empty`

```java
@Test
public void option() {
    Optional<Object> empty = Optional.empty();
    log.info("创建一个空的 Optional ==> {}", empty);
    Optional<String> s = Optional.of("1");
    log.info("创建一个有值的 Optional ==> {}", s);
    // 传入null值会抛出 java.lang.NullPointerException
    // Optional<String> s2 = Optional.of(null);
    // log.info("创建一个为null的 Optional ==> {}", s2);
    Optional<Object> o = Optional.ofNullable(null);
    log.info("创建一个为null的 Optional ==> {}", o);
}

```

#### 空检验

- isPresent()：判断 `Optional`  是否为空，为空返回false，否则返回true
- ifPresent(Consumer<? super T> consumer)：不为空则执行操作处理。

```java

@Test
public void optionEmpty() {
    Optional<Object> empty = Optional.empty();
    log.info("isPresent => {}", empty.isPresent());


    Optional<String> s = Optional.of("1");
    s.ifPresent(item -> {
        log.info("我的值 {}", item);
    });
}
```

输出结果

```javascript
isPresent => false
我的值 1
```

#### 获取值

- get。获取 `Optional` 中的对象，当 `Optional` 为空时报错 `No value present`
- orElse。 不论容器是否为空,只要调用该方法, 则对象`other`一定存在。但是为空时返回`Optional`的值，不为空才返回 `other`
- orElseGet。 只有当容器为空时,才调用`supplier.get()`方法产生对象。
- orElseThrow。当容器为空时，才调用 `supplier.get()` 方法抛出异常。

```java
@Test
public void optionGet() {
    Optional<String> s = Optional.of("1");
    log.info("创建一个有值的 Optional ==> {}", s);
    Optional<Object> o = Optional.ofNullable(null);
    log.info("创建一个为null的 Optional ==> {}", o);

    try {
        log.info("get => {}", s.get());
        log.info("get => {}", o.get());
    } catch (Exception ex) {
        log.error("null get =>{}", ex.getMessage());
    }
    log.info("orElse => {}", s.orElse(pp("2")));
    log.info("orElseGet => {}", s.orElseGet(() -> {
        log.info(" 我被执行了 s1");
        return "s1";
    }));

    log.info("orElse => {}", o.orElse(2));
    log.info("orElseGet => {}", o.orElseGet(() -> {
        log.info("s2");
        return "s2";
    }));
    
   
    o.orElseThrow(() ->{
        return new RuntimeException("空的异常");
    });
}

private String pp(String str) {
    log.info("我被执行了");
    return str;
}
```

输出结果

```javascript
17:03:21.566 [main] INFO top.zsmile.test.basic.stream.StreamTest - 创建一个有值的 Optional ==> Optional[1]
17:03:21.577 [main] INFO top.zsmile.test.basic.stream.StreamTest - 创建一个为null的 Optional ==> Optional.empty
17:03:21.578 [main] INFO top.zsmile.test.basic.stream.StreamTest - get => 1
17:03:21.578 [main] ERROR top.zsmile.test.basic.stream.StreamTest - null get =>No value present
17:03:21.578 [main] INFO top.zsmile.test.basic.stream.StreamTest - 我被执行了
17:03:21.578 [main] INFO top.zsmile.test.basic.stream.StreamTest - orElse => 1
17:03:21.633 [main] INFO top.zsmile.test.basic.stream.StreamTest - orElseGet => 1
17:03:21.633 [main] INFO top.zsmile.test.basic.stream.StreamTest - orElse => 2
17:03:21.633 [main] INFO top.zsmile.test.basic.stream.StreamTest - s2
17:03:21.633 [main] INFO top.zsmile.test.basic.stream.StreamTest - orElseGet => s2

java.lang.RuntimeException: 空的异常
// ....
```

可以从结果中看到，`Optional` 不为空时，**`orElse` 依然调用方法并产生了一个`String`对象，`orElseGet` 没有调用方法。**
所以可见`orElseGet` 更优, 但代价就是需要传入一个`Supplier<T>`类型的参数。

#### 过滤

`Optional<T> filter(Predicate<? super T> predicate)` 如果容器不为空，则返回原先的值，如果`predicate` 返回 true，则返回原先的值，否则返回 `empty`

```java
Optional<String> s = Optional.of("2");
log.info("原数据 {}", s);
Optional<String> s1 = s.filter(item -> {
    if (item.equalsIgnoreCase("1")) {
        return true;
    } else {
        return false;
    }
});
log.info("filter {}", s1);
```

输出结果

```javascript
17:45:34.594 [main] INFO top.zsmile.test.basic.stream.StreamTest - 原数据 Optional[2]
17:45:34.666 [main] INFO top.zsmile.test.basic.stream.StreamTest - filter Optional.empty
```

#### 映射

- map：如果 `Optional` 不为空，则将原先的值映射为新的对象，并用一个新的`Optional` 存放。
- flatMap：如果 `Optional` 不为空，则将原先的值映射为新的`Optional`对象。
- 二者区别在于：
  1. `map` 允许返回空值，而flat不允许为空。
  2. `map` 的 `Function` 接口方法都是返回 Object，而`flatMap`返回的是 `Optional` 对象。

### 查找 find

- findFirst：获取第一个匹配的元素
- findAny：获取任意一个，经实验，多数为第一个

```java
@Test
public void find() {

    List<Double> collect = Stream.generate(Math::random).limit(999999).distinct().collect(Collectors.toList());
    Optional<Double> first = collect.stream().findFirst();
    log.info("findFirst => {}", first);

    Optional<Double> any = collect.stream().findAny();
    log.info("findAny => {}", any);
}
```

输出结果

```java
11:15:02.155 [main] INFO top.zsmile.test.basic.stream.StreamTest - findFirst => Optional[0.7103524491747812]
11:15:02.161 [main] INFO top.zsmile.test.basic.stream.StreamTest - findAny => Optional[0.7103524491747812]
```





### 归并 reduce

`reduce` 有三个重要概念：

1. Identity(初始值定义)：归并操作的初始值，如果`Stream` 为空，也是默认返回值。
2. accumulator(累进器)：定义两个参数，第一个是上一次归并操作的返回值，第二个是`Stream` 的下一个元素。
3. combiner(组合器)：调用一个函数来组合归并操作的结果，当归并是用来并行执行或当累进器及实现类型不匹配时，才调用。

`reduce` 有三个重载方法。

- Optional<T> reduce(BinaryOperator<T> accumulator);
  - 接收一个`BinaryOperator` 的 lambda表达式，返回一个 `Optional` 
  - 当执行结果为空时，返回的是 `Optional.empty`
- T reduce(T identity, BinaryOperator<T> accumulator);
  - 与上一个不同的，它定义了一个初始值，并且返回值是与初始值类型相同
  - 当执行结果为空时，返回的是默认值
- <U> U reduce(U identity,BiFunction<U, ? super T, U> accumulator,BinaryOperator<U> combiner);
  - 主要是为了解决并发和复杂类型的计算，如例子所言，我想统计所有人的年龄总和，可是光用前两种参数，并没有办法执行统计后的数据类型。

```java
@Test
public void reduce() {
    List<Integer> collect = Stream.iterate(0, integer -> integer + 1).limit(10).collect(Collectors.toList());

    log.info("原数据");
    collect.stream().forEach(System.out::println);

    log.info("总计：{}", collect.stream().reduce((a, b) -> a + b));
    log.info("总计：{}", collect.stream().skip(10).reduce((a, b) -> a + b));
    log.info("总计：{}", collect.stream().skip(10).reduce(0, (a, b) -> a + b));

    log.info("原数据2");
    userList.stream().forEach(System.out::println);
    // 注意：这个写法无法通过编译，Stream的实现类型是 User，而累进器返回的类型是Integer，所以这需要另外使用Integer::sum 做组合操作。
    // userList.stream().reduce(0, (a, b) -> a + b.getAge())
    log.info("总计年龄：{}", userList.stream().reduce(0, (a, b) -> a + b.getAge(), Integer::sum));
}
```

输出结果

```javascript
11:42:38.197 [main] INFO top.zsmile.test.basic.stream.StreamTest - 原数据
0
1
2
3
4
5
6
7
8
9
11:42:38.202 [main] INFO top.zsmile.test.basic.stream.StreamTest - 总计：Optional[45]
11:42:38.205 [main] INFO top.zsmile.test.basic.stream.StreamTest - 总计：Optional.empty
11:42:38.206 [main] INFO top.zsmile.test.basic.stream.StreamTest - 总计：0
11:42:38.206 [main] INFO top.zsmile.test.basic.stream.StreamTest - 原数据2
User(name=张三, age=18, roles=[ADMIN, USER])
User(name=李四, age=19, roles=[TEMP:USER])
User(name=王五, age=20, roles=[USER])
11:42:38.207 [main] INFO top.zsmile.test.basic.stream.StreamTest - 总计年龄：57

```



 # 总结

合理正确的使用 `Stream` 流，可以给我们带来：

1. 代码简洁。更贴合数据流处理的思想。
2. 逻辑解耦。可以将各个任务分成多段流式进行处理。
3. **延迟执行**。只有调用终端方法时，前面编写的中间操作才会执行，避免不必要的操作损耗。

当然也会有些新的问题

1. 调试不方便
2. 需要适应语法，熟悉lambda的使用，才能更高效开发。