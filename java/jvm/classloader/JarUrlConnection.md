[toc]

---

# 前言
最近研究关于classloader，实现对模块的热插拔和类替换等功能。然后意外发现一个用于处理JAR包的官方库。


# 相关概念
## 什么是JarUrlConnetion
jarUrlConnection通过Jar协议建立一个访问jar包的连接，可以访问这个jar包内部数据

## jar协议格式 
jar协议的格式如：jar:{archive-url}!/{entry}。
- archive-url是文件地址
- !/是分隔符
- entry是jar包内部的目录或者文件

示例：
- 访问远程http服务器
    - 整个jar包。jar:http://www.zsmile.top/test.jar!/
    - jar包内的文件。jar:http://www.zsmile.top/test.jar!/Hello.class
    - jar包内的目录。jar:http://www.zsmile.top/test.jar!/top/smile/
- 访问本地文件系统  
    - 整个jar包。jar:file://E:/test.jar!/
    - jar包内的文件。jar:file://E:/test.jar!/Hello.class
    - jar包内的目录。jar:file://E:/test.jar!/top/smile/

# 功能
## 获取JarFile
```java
URL url = new URL("jar:file://E:/test.jar!/");
JarURLConnection conn = (JarURLConnection) url.openConnection();
JarFile jarfile = conn.getJarFile();
```

## 遍历JarEntry
```java
Enumeration<JarEntry> entries = jarFile.entries();
while (entries.hasMoreElements()) {
JarEntry jarEntry = entries.nextElement();
System.out.println(jarEntry.getName() + " is directory ：" + jarEntry.isDirectory());
}
```


## 执行class文件
```java
// 获取class数据流
 InputStream inputStream = jarFile.getInputStream(jarEntry);
 int available = inputStream.available();
 byte[] bytes = new byte[available];
inputStream.read(bytes);

// 借助classloader的defineClass将byte转换成class对象
Class<?> aClass = defineClass("top.zsmile.jvm.classloader.Hello", bytes, 0, bytes.length);

// 通过反射调用方法
Method method = aClass.getMethod("main", String[].class);
method.invoke(null, (Object) null);
```

## Manifest
```java
Manifest mainfest = jarfile.getManifest();

// 获取并遍历属性
 System.out.println("manifest Attributes:");
Attributes mainAttributes = manifest.getMainAttributes();
Set<Map.Entry<Object, Object>> entries = mainAttributes.entrySet();
for (Map.Entry<Object, Object> entry : entries) {
    System.out.println(entry.getKey() + "=>" + entry.getValue());
}

// 获取并遍历entry(不知道怎么用)
Map<String, Attributes> entries1 = manifest.getEntries();
for (Map.Entry<String, Attributes> entry : entries1.entrySet()) {
    System.out.println(entry.getKey() + "=>" + entry.getValue());
}

// 获取主函数
String mainClassName = manifest.getMainAttributes().getValue(Attributes.Name.MAIN_CLASS);
```
