[toc]

---

# 如何排查找不到Jar包的问题？
有时候我们会面临明明已经把某个jar加入到了环境里，可以运行的时候还是找不到。 那么我们有没有一种方法，可以直接看到各个类加载器加载了哪些jar，以及把哪些路径加到了classpath里？答案是肯定的，代码如下：
```java
package top.zsmile.jvm.classloader;

import java.lang.reflect.Field;
import java.net.URL;
import java.net.URLClassLoader;
import java.util.ArrayList;

public class JvmClassLoaderPrintPath {
    public static void main(String[] args) {
        // 启动类加载器
        URL[] urls = sun.misc.Launcher.getBootstrapClassPath().getURLs();
        System.out.println("启动类加载器");
        for (URL url : urls) {
            System.out.println(" ==> " + url.toExternalForm());
        }

        // 扩展类加载器
        printClassLoader("扩展类加载器", JvmClassLoaderPrintPath.class.getClassLoader().getParent());

        // 应用类加载器
        printClassLoader("应用类加载器", JvmClassLoaderPrintPath.class.getClassLoader());
    }


    public static void printClassLoader(String name, ClassLoader CL) {
        if (CL != null) {
            System.out.println(name + " ClassLoader ‐> " + CL.toString());
            printURLForClassLoader(CL);
        } else {
            System.out.println(name + " ClassLoader ‐> null");
        }
    }

    public static void printURLForClassLoader(ClassLoader CL) {
        Object ucp = insightField(CL, "ucp");
        Object path = insightField(ucp, "path");
        ArrayList ps = (ArrayList) path;
        for (Object p : ps) {
            System.out.println(" ==> " + p.toString());
        }
    }

    private static Object insightField(Object obj, String fName) {
        try {
            Field f = null;
            if (obj instanceof URLClassLoader) {
                f = URLClassLoader.class.getDeclaredField(fName);
            } else {
                f = obj.getClass().getDeclaredField(fName);
            }
            f.setAccessible(true);
            return f.get(obj);
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
    }
}

```

从打印结果，我们可以看到三种类加载器各自默认加载了哪些jar包和包含了哪些 classpath的路径。

# 如何排查类的方法不一致的问题？

假如我们确定一个jar或者class已经在classpath里了，但是却总是提示 **java.lang.NoSuchMethodError** ，这是怎么回事呢？很可能是加载了错误的或者重复加载了不同版本的jar包。这时候，用前面的方法就可以先排查一下，加载了具体什么jar，然后是不是不同路径下有重复的class文件，但是版本不一样。

# 怎么看到加载了哪些类，以及加载顺序？

还是针对上一个问题，假如有两个地方有Hello.class，一个是新版本，一个是旧的， 怎么才能直观地看到他们的加载顺序呢？也没有问题，我们可以直接打印加载的类清 单和加载顺序。 

**只需要在类的启动命令行参数加上 ‐XX:+TraceClassLoading 或者 ‐verbose 即 可**，注意需要加载java命令之后，要执行的类名之前，不然不起作用。例如：
> $ java ‐XX:+TraceClassLoading jvm.HelloClassLoader

# 怎么调整或修改ext和本地加载路径？
从前面的例子我们可以看到，假如什么都不设置，直接执行java命令，默认也会加载 非常多的jar包，怎么可以自定义加载哪些jar包呢？比如我的代码很简单，只加载rt.jar 行不行？答案是肯定的。

> java ‐Dsun.boot.class.path="D:\Program Files\Java\jre1.8.0_231\lib\rt.jar"

其中命令行**参数 ‐Dsun.boot.class.path 表示我们要指定启动类加载器加载什么**， 最基础的东西都在rt.jar这个包了里，所以一般配置它就够了。 
**参数 ‐Djava.ext.dirs 表示扩展类加载器要加载什么**，一般情况下不需要的话可以直接配置为空即可。

# 怎么运行期加载额外的jar包或者class呢？
简单说就是不使用命令行参数的情况下，怎么用代码来运行时改变加载类的路径和方 式。假如说，在 d:/app/jvm 路径下，有我们刚才使用过的Hello.class文件，怎么在 代码里能加载这个Hello类呢？
- 一个是前面提到的自定义ClassLoader的方式
- 还有一个就是直接在当前 的应用类加载器里，使用 URLClassLoader类的方法addURL，不过这个方法是protected的，需要反射处理一下，然后又因为程序在启动时并没有显示加载Hello类，所以在添加完了classpath以后，没法直接显式初始化，需要使用Class.forName的方式来拿到已经加载的Hello类（Class.forName("jvm.Hello")默认会初始化并执行静态代码块）

```java
package top.zsmile.jvm.classloader;

import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;
import java.net.MalformedURLException;
import java.net.URL;
import java.net.URLClassLoader;

public class JvmAppClassLoaderAddUrl {
    public static void main(String[] args) {
        String appPath = "file:/d:/app/";
        URLClassLoader urlClassLoader = (URLClassLoader) JvmAppClassLoaderAddUrl.class.getClassLoader();
        try {
            Method addURL = URLClassLoader.class.getDeclaredMethod("addURL", URL.class);

            addURL.setAccessible(true);
            URL url = new URL(appPath);
            addURL.invoke(urlClassLoader, url);
            Class.forName("top.zsmile.jvm.classloader.Hello"); // 效果跟Class.forName("jvm.Hello").newInstance()一样
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}


```