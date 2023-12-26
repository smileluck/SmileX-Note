[toc]

---

# 加载包下的类

```java
/**
     * 查找包下面的类
     * 规则 top.zsmile\**\*.class
     *
     * @param searchPath 路径，支持ANT
     * @return
     */
public static Map<String, Class> getClassBySuperClass(String searchPath, Class superClass) throws IOException {
    Map<String, Class> handlerMap = new HashMap<String, Class>();
    //spring工具类，可以获取指定路径下的全部类
    ResourcePatternResolver resourcePatternResolver = new PathMatchingResourcePatternResolver();
    try {
        String pattern = ResourcePatternResolver.CLASSPATH_ALL_URL_PREFIX +
            searchPath;
        Resource[] resources = resourcePatternResolver.getResources(pattern);
        //MetadataReader 的工厂类
        MetadataReaderFactory readerfactory = new CachingMetadataReaderFactory(resourcePatternResolver);
        for (Resource resource : resources) {
            //用于读取类信息
            MetadataReader reader = readerfactory.getMetadataReader(resource);
            //扫描到的class
            String classname = reader.getClassMetadata().getClassName();
            // 记载class类
            Class<?> clazz = Class.forName(classname);
            //判断是否有指定注解
            if (superClass != null) {
                Class<?>[] interfaces = clazz.getInterfaces();
                if (interfaces.length > 0) {
                    if (interfaces[0].equals(superClass)) {
                        handlerMap.put(classname, clazz);
                    }
                }
            } else {
                handlerMap.put(classname, clazz);
            }
        }
        return handlerMap;
    } catch (IOException | ClassNotFoundException e) {
        throw new IOException("找不到指定Class");
    }
}
```

