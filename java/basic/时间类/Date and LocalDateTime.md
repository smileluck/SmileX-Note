[toc]

---



# 前言

JDK 8 里面提供了一个新的 LocalDateTime API，加强了对时间的管理。直接可以说就是为了替代 Date 使用。那么我们通过源码了解一下二者有何差异。

# 对比

## 可读性差

直接打印，不执行格式化。

```java
Date date = new Date();
LocalDateTime localDateTime = LocalDateTime.now();
log.info("Date => {}, year={}, month={}", date, date.getYear(), date.getMonth());
log.info("LocalDateTime => {}, year={}, month={}", localDateTime, localDateTime.getYear(), localDateTime.getMonthValue());

// 输出结果
Date => Tue Oct 04 15:20:16 CST 2022, year=122, month=9
LocalDateTime => 2022-10-04T15:20:16.709, year=2022, month=10
```

- Date获取出来的时间不够直观；LocalDateTime可以直观看出当前时间

- Date获取的年份还需要再次计算（加上1900），才能得出正确年份；LocalDateTime可以直接得出当前年份

- Date获取的月份从0计算少一个月；而LocalDateTime获取的月份是准确的。

- 最重要的是：Date里面有很多方法已经被弃用了，而LocalDateTime本身还更多的方法等

  - localDateTime.getDayOfMonth()。获取当前月的第几天
  - localDateTime.getDayOfWeek()。获取当前星期几

## 指定年月生成

```java
// 指定年月生成对象
Date pointDate = new Date(2022, 1, 1);
LocalDateTime pointDateTime = LocalDateTime.of(2022, 1, 1, 0, 0);
log.info("pointDate => {}", pointDate);
log.info("pointDateTime => {}", pointDateTime);
```

可以看到 Date 连构造函数都被弃用了，

## 日期格式化(重点)

```java
// 格式化输出
SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
log.info("Date format =>{}", simpleDateFormat.format(date));
log.info("LocalDateTime format =>{}", localDateTime.format(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss")));
```

Date 使用的是SimpleDateFormat，而LocalDateTime 是本身提供了方法。如果仅仅只是用法上的不同，那么无伤大雅。我们来看一下两者有什么区别。

### SimpleDateFormat

我们先看SimpleDateFormat的部分源码。

```java
/**
Date formats are not synchronized.
It is recommended to create separate format instances for each thread.
If multiple threads access a format concurrently, it must be synchronized externally.*/

public class SimpleDateFormat extends DateFormat {
    
    /**
     * The {@link Calendar} instance used for calculating the date-time fields
     * and the instant of time. This field is used for both formatting and
     * parsing.
     *
     * <p>Subclasses should initialize this field to a {@link Calendar}
     * appropriate for the {@link Locale} associated with this
     * <code>DateFormat</code>.
     * @serial
     */
    protected Calendar calendar;
    
    // Called from Format after creating a FieldDelegate
    private StringBuffer format(Date date, StringBuffer toAppendTo,
                                FieldDelegate delegate) {
        // Convert input date to time field list
        calendar.setTime(date);
        
        // etc
    }
}
```

从上面的源代码中可以看出几点。

1. 非线程安全的。官方说明DateForamt不是同步的。建议为每个线程创建单独的格式实例，如果多线程要使用同一个实例，则必须从外部同步它。
2. Calendar 是共享变量，没有做线程安全控制，多个线程同事调用format时，会调用 Calendar.setTime，导致多个线程执行时，会冲突覆盖 Calendar 的值，最终返回错误结果。

3. 由前面两点推断出，如果说每个线程需要一个单独的SimpleDateFormat，那么将会导致创建和销毁对象，开销大。
4. 对SimpleDataFormat 的format和parse方法加同步锁，会导致性能变差。

简而言之，SimpleDateForamt 是需要我们自己来保证多线程安全的。

### DateTimeFormatter

既然SimpleDataFormat 是线程不安全的，那么看看 DateTimeFormatter 是怎么样的呢？

```java
/*
 * This class is immutable and thread-safe.
 *
 * @since 1.8
 */
public final class DateTimeFormatter {
	
    
    public static DateTimeFormatter ofPattern(String pattern) {
        return new DateTimeFormatterBuilder().appendPattern(pattern).toFormatter();
    }
    
    /**
     * Completes this builder by creating the formatter.
     *
     * @param locale  the locale to use for formatting, not null
     * @param chrono  the chronology to use, may be null
     * @return the created formatter, not null
     */
    private DateTimeFormatter toFormatter(Locale locale, ResolverStyle resolverStyle, Chronology chrono) {
        Objects.requireNonNull(locale, "locale");
        while (active.parent != null) {
            optionalEnd();
        }
        CompositePrinterParser pp = new CompositePrinterParser(printerParsers, false);
        return new DateTimeFormatter(pp, locale, DecimalStyle.STANDARD,
                resolverStyle, null, chrono, null);
    }

}
```

- 从类定义上，可以很明显的看出，它是不可变的，而且也是线程安全的。
- 我们调用ofPattern时，最终会调用toFormatter方法，会创建一个新的对象，也就是说，不会使用缓存对象。

简而言之，对象不可变，线程安全，用就完事了。





## **线程不安全(重点)**

其实不单单日期格式化的类是有差别的，单单Date和LocalDateTime本身我们也能更好的取舍。

```java
public class Date
    implements java.io.Serializable, Cloneable, Comparable<Date>{}
    
/*
 * This class is immutable and thread-safe.
 */
public final class LocalDateTime{}
```

- Date 是可变对象，线程不安全
- LocalDateTime 是不可变对象，线程安全。



# LocalDateTime使用

## 创建事件对象

```java
log.info("==========创建时间对象==========");
// 获取当前时间
LocalDateTime localDateTime = LocalDateTime.now();

// 指定时间
LocalDateTime pointDateTime = LocalDateTime.of(2022, 1, 1, 0, 0);
```

## 获取年月日时分秒信息

```java
 log.info("==========获取时间信息==========");
 log.info("Year : {}",localDateTime.getYear());
 log.info("Month : {}",localDateTime.getMonth());
 log.info("Day : {}",localDateTime.getDayOfMonth());
 log.info("Hour : {}",localDateTime.getHour());
 log.info("Minute : {}",localDateTime.getMinute());
 log.info("Second : {}",localDateTime.getSecond());
```

## 转换类型

````java
log.info("==========转换时间==========");
log.info("LocalDate : {}",localDateTime.toLocalDate());
log.info("LocalTime : {}",localDateTime.toLocalTime());	
````

## 修改时间

PS：修改后会生成新的对象，而不影响原先的对象。

```java
log.info("==========加减时间==========");
log.info("操作年 : {}", localDateTime.plusYears(1));
log.info("操作月 : {}", localDateTime.plusMonths(1));
log.info("操作日 : {}", localDateTime.plusDays(1));
log.info("操作时 : {}", localDateTime.plusHours(1));
log.info("操作分 : {}", localDateTime.plusMinutes(1));
log.info("操作秒 : {}", localDateTime.plusSeconds(1));

log.info("==========设置时间==========");
log.info("操作年 : {}", localDateTime.withYear(2021));
log.info("操作月 : {}", localDateTime.withMonth(1));
log.info("操作日 : {}", localDateTime.withDayOfMonth(1));
log.info("操作时 : {}", localDateTime.withHour(1));
log.info("操作分 : {}", localDateTime.withMinute(1));
log.info("操作秒 : {}", localDateTime.withSecond(1));
```

## 格式化

```java
log.info("==========格式化==========");
        log.info("BASIC_ISO_DATE : {}", localDateTime.format(DateTimeFormatter.BASIC_ISO_DATE));
        log.info("ISO_DATE_TIME : {}", localDateTime.format(DateTimeFormatter.ISO_DATE_TIME));
        log.info("自定义 : {}",localDateTime.format(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss")));
```

## 反解析

```java
log.info("==========反解析==========");
log.info("parse : {}", localDateTime.parse("2021-01,01 11:22-02",DateTimeFormatter.ofPattern("yyyy-MM,dd HH:mm-ss")));
```

## 日期间隔(Period)

```java
log.info("==========间隔(Period)==========");
LocalDate beforeLocalDate = LocalDate.of(2021, 1, 1);
LocalDate afterLocalDate = LocalDate.of(2022, 12, 1);
log.info("first : {}", beforeLocalDate);
log.info("sec : {}", afterLocalDate);
Period period = Period.between(beforeLocalDate, afterLocalDate);
log.info("period => {}", period);
log.info("period Year : {}", period.getYears());
log.info("period Month : {}", period.getMonths());
log.info("period Day : {}", period.getDays());
log.info("period TotalMonths : {}", period.toTotalMonths());
```

## 时间间隔(Duration)

```java
log.info("==========时间间隔(Duration)==========");
LocalDateTime beforeLocalDateTime = LocalDateTime.of(2021, 1, 1,1,1);
LocalDateTime afterLocalDateTime = LocalDateTime.of(2022, 12, 1,1,1);
log.info("first : {}", beforeLocalDateTime);
log.info("sec : {}", afterLocalDateTime);
Duration duration = Duration.between(beforeLocalDateTime, afterLocalDateTime);
log.info("duration => {}", duration);
log.info("duration to days : {}", duration.toDays());
log.info("duration to hours : {}", duration.toHours());
log.info("duration to millis : {}", duration.toMillis());
log.info("duration seconds : {}", duration.getSeconds());
```

## 时间比较

```java
log.info("==========时间比较==========");
log.info("first : {}", beforeLocalDateTime);
log.info("sec : {}", afterLocalDateTime);
log.info("now : {}", localDateTime);
log.info("first isAfter sec: {}", beforeLocalDateTime.isAfter(afterLocalDateTime));
log.info("sec isAfter now: {}", afterLocalDateTime.isAfter(localDateTime));
log.info("first isBefore sec: {}", beforeLocalDateTime.isBefore(afterLocalDateTime));
log.info("sec isBefore now: {}", afterLocalDateTime.isBefore(localDateTime));
```

# 总结

官方推出LocalDateTime也是为了能够替代Date使用，尽管网上有些人说性能上Date更好一点，但是LocalDateTime所带来的好处，远不是这一点性能上可以超越的。所以在使用上见仁见智，我本身也更偏向使用LocalDateTime。



