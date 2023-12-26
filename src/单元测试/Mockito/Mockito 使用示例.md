[toc]

---

# 前言

## 环境配置

本篇笔记基于环境如下：

- SpringBoot：2.1.3.RELEASE
- junit：4.13.2
- mockito：2.23.4
- Mybatis-Plus：3.4.2

`pom.xml`  文件相关内容如下：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
    <exclusions>
        <exclusion>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <scope>test</scope>
    <version>4.13.2</version>
</dependency>
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
    <version>3.4.2</version>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>
```

## 参考文档

# 基本使用

## 常用注解

-  `@RunWith(MockitoJUnitRunner.class)` 。 告诉`JUnit`使用`Mockito`模拟框架提供的"单元测试运行器" 

- `@InjectMocks`。 **创建一个实例，简单的说是这个Mock可以调用真实代码的方法，其余用@Mock（或@Spy）注解创建的mock将被注入到用该实例中。** 
- `@Mock`。**对函数的调用均执行mock（即虚假函数），不执行真正部分。** 是完全模拟出来的，就算项目中没有这个类的实例，也能自己mock出来一个。 
- `@Spy`。 **对函数的调用均执行真正部分。** 如果不存在需要我们创建一个。

`@Mock` 和 `@Spy` 都可用于拦截那些尚未实现或不期望被真实调用的对象和方法，并为其设置自定义行为。二者的区别在于Mock不真实调用，Spy会真实调用。 

## 常用代码配置

- `Mockito.initMocks(this)`。初始化当前的测试类 

# Mybatis-Plus

## lambda的使用

```java
@InjectMocks
MainServiceImpl mainService; // 测试的真实主类

@Mock
GoodsServiceImpl goodsService; // Mockito 自动模拟的类

@Mock
GoodsMapper goodsMapper;

@Mock
BaseMapper baseMapper;

@Before
public void setUp() {
    MockitoAnnotations.initMocks(this);

    MapperBuilderAssistant mapperBuilderAssistant = new MapperBuilderAssistant(new MybatisConfiguration(), "");
    TableInfoHelper.initTableInfo(mapperBuilderAssistant, Main.class);
    TableInfoHelper.initTableInfo(mapperBuilderAssistant, Goods.class);

 Mockito.when(goodsService.lambdaQuery()).thenReturn(ChainWrappers.lambdaQueryChain(baseMapper));
    Mockito.when(goodsService.lambdaUpdate()).thenReturn(ChainWrappers.lambdaUpdateChain(baseMapper));
 
    Mockito.when(baseMapper.insert(Mockito.any())).thenReturn(1);
}
```

- 之所以需要模拟 `goodsService.lambdaQuery` 是因为因为 `Mockito` 的 `@Mock` 注入的模拟示例，是没有提供 `LambdaQueryChain`示例的，需要我们手动注入一下，不然再后续执行中，会抛出异常。
-  `baseMapper.insert(Mockito.any())` 使用来处理在 `IService` 抽象接口中提供的 `save`  等方法，实际调用的是 `BaseMapper` 的 `insert` 方法。此外其它的 `IService.delete` 等方法也都可以直接模拟 `BaseMapper.delete` 。
- `TableInfoHelper.initTableInfo(mapperBuilderAssistant, Goods.class);`。**需要初始化，否则会提示异常 `can not find lambda cache for this entity`**

## Mockito.mock和new

```java
// new 真实对象
List<Long> realList = new ArrayList<>();
realList.add(1L);
realList.add(2L);
// 返回真实对象
Mockito.when(mainService.list(Mockito.any())).thenReturn(realList);

// Mockito.mock 模拟对象
List<Long> mockList = Mockito.mock(List.class);
mockList.add(1L);
mockList.add(2L);
// 返回模拟对象
Mockito.when(mainService.list2(Mockito.any())).thenReturn(mockList);
```

这里会有一个很神奇的地方，我才用 **`Mockito.mock` 创建类的时候，执行了 `set/add` 方法，但是实际并没有影响到实体数据。** 所以需要参与真实业务的地方需使用 `new`  创建类。

另外，在 `Mockito 2.0.6` 的中文文档中，使用的是 `Mockito.mock` 进行操作和校验，但是试验了后，无法满足需要，具体分析可能看实际场景吧。

## Mock Mapper
>Mock 主测试service的mapper时，会显示 NullPointerException 或者 ClassCastException: com.amiba.heitan.mall.admin.mapper.KbActivityAnnounceMapper$MockitoMock$332138621 cannot be cast to com.amiba.heitan.mall.admin.mapper.KbActivityAnnounceDmMapper

正确 Mock Mapper 的代码如下：

```java

@RunWith(MockitoJUnitRunner.class)
public class KbActivityAnnounceDmServiceTest {

    @InjectMocks
    KbActivityAnnounceDmServiceImpl kbActivityAnnounceDmService;
    
    @Mock(name="baseMapper")
    KbActivityAnnounceDmMapper kbActivityAnnounceDmMapper;
    
    @Before
    public void setUp() {
        MockitoAnnotations.initMocks(this);

        MapperBuilderAssistant mapperBuilderAssistant = new MapperBuilderAssistant(new MybatisConfiguration(), "");
        TableInfoHelper.initTableInfo(mapperBuilderAssistant, KbActivityAnnounce.class);
    }

    @Test
    public void selectDmListByActivityId() {
        KbActivityAnnounceListCompereDTO dto = new KbActivityAnnounceListCompereDTO();
        dto.setId(1L);
        KbActivityAnnounce one = new KbActivityAnnounce();
      
        kbActivityAnnounceDmService.selectDmListByActivityId(dto);
    }
}

public interface KbActivityAnnounceDmMapper extends BaseMapper<KbActivityAnnounceDm> {
    List<KbActivityAnnounceCompereListVO> selectDmListByActivityId(@Param("dto") KbActivityAnnounceListCompereDTO dto);
}
```

这样才能正确模拟实际测试类的 `BaseMapper` 。核心在于以下这句话

```java
@Mock(name="baseMapper")
KbActivityAnnounceDmMapper kbActivityAnnounceDmMapper;
```



## MockMultipartFile

- 读取真实文件进行文件模拟

```java
File file = ResourceUtils.getFile("classpath:excel/demo.xlsx");
MultipartFile mulFile = new MockMultipartFile(
    "demo.xlsx", //文件名
    "demo.xlsx", //originalName 相当于上传文件在客户机上的文件名
    ContentType.APPLICATION_OCTET_STREAM.toString(), //文件类型
    new FileInputStream(file) //文件流
);
```

- 模拟文件流进行文件模拟

```java
byte[] content = new byte[2];
content[0] = 1;
content[1] = 2;
MultipartFile mulFile = new MockMultipartFile(
    "demo.xlsx", //文件名
    "demo.xlsx", //originalName 相当于上传文件在客户机上的文件名
    ContentType.APPLICATION_OCTET_STREAM.toString(), //文件类型
    content //文件流
);
```

