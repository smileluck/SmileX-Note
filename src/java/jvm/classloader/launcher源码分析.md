[toc]

---



# sun.misc.launcher

java虚拟机的入口

## Launcher初始化
```java
private static URLStreamHandlerFactory factory = new Launcher.Factory();
private static Launcher launcher = new Launcher();
private static String bootClassPath = System.getProperty("sun.boot.class.path");
private ClassLoader loader;
private static URLStreamHandler fileHandler;

public Launcher() {
    Launcher.ExtClassLoader var1;
    try {
        var1 = Launcher.ExtClassLoader.getExtClassLoader();
    } catch (IOException var10) {
        throw new InternalError("Could not create extension class loader", var10);
    }

    try {
        this.loader = Launcher.AppClassLoader.getAppClassLoader(var1);
    } catch (IOException var9) {
        throw new InternalError("Could not create application class loader", var9);
    }

    Thread.currentThread().setContextClassLoader(this.loader);
    String var2 = System.getProperty("java.security.manager");
    if (var2 != null) {
        SecurityManager var3 = null;
        if (!"".equals(var2) && !"default".equals(var2)) {
            try {
                var3 = (SecurityManager)this.loader.loadClass(var2).newInstance();
            } catch (IllegalAccessException var5) {
            } catch (InstantiationException var6) {
            } catch (ClassNotFoundException var7) {
            } catch (ClassCastException var8) {
            }
        } else {
            var3 = new SecurityManager();
        }

        if (var3 == null) {
            throw new InternalError("Could not create SecurityManager: " + var2);
        }

        System.setSecurityManager(var3);
    }

}
```
从这段Launcher构造器的代码中，我们可以获得一下信息：
1. Launcher初始化了ExtClassLoader 和 AppClassloader，并将ExtClassLoader作为AppClassloader的父加载器（注意这里不是继承关系的）
2. 这里可以看到的是，并没有显式的将BootstrapClassloader作为ExtClassloader的父加载器。但是我们可以从classloader的loadClass方法中看到，会调用native方法，实际就是从BootstrapClassloader获取（因为BootstrapClassloader实际是C++实现，并没有继承Classloader的）
3. 将线程的默认类加载器设置为AppClassloader
4. 这里没有初始化BootstrapClassloader，但可以有这样的代码 System.getProperty("sun.boot.class.path") 来加载一些包。


输出一下System.getProperty("sum.boot.class.path")的内容，我们可以看到就bootstrapClassloader的内容。也可以通过sun.misc.Launcher.getBootstrapClassPath()获取路径信息
```java
 ==> file:/C:/Program%20Files/Java/jdk1.8.0_201/jre/lib/resources.jar
 ==> file:/C:/Program%20Files/Java/jdk1.8.0_201/jre/lib/rt.jar
 ==> file:/C:/Program%20Files/Java/jdk1.8.0_201/jre/lib/sunrsasign.jar
 ==> file:/C:/Program%20Files/Java/jdk1.8.0_201/jre/lib/jsse.jar
 ==> file:/C:/Program%20Files/Java/jdk1.8.0_201/jre/lib/jce.jar
 ==> file:/C:/Program%20Files/Java/jdk1.8.0_201/jre/lib/charsets.jar
 ==> file:/C:/Program%20Files/Java/jdk1.8.0_201/jre/lib/jfr.jar
 ==> file:/C:/Program%20Files/Java/jdk1.8.0_201/jre/classes
```

这里还有初始化一个BootClassPathHolder
```java
private static class BootClassPathHolder {
    static final URLClassPath bcp;

    private BootClassPathHolder() {
    }

    static {
        URL[] var0;
        if (Launcher.bootClassPath != null) {
            var0 = (URL[])AccessController.doPrivileged(new PrivilegedAction<URL[]>() {
                public URL[] run() {
                    File[] var1 = Launcher.getClassPath(Launcher.bootClassPath);
                    int var2 = var1.length;
                    HashSet var3 = new HashSet();

                    for(int var4 = 0; var4 < var2; ++var4) {
                        File var5 = var1[var4];
                        if (!var5.isDirectory()) {
                            var5 = var5.getParentFile();
                        }

                        if (var5 != null && var3.add(var5)) {
                            MetaIndex.registerDirectory(var5);
                        }
                    }

                    return Launcher.pathToURLs(var1);
                }
            });
        } else {
            var0 = new URL[0];
        }

        bcp = new URLClassPath(var0, Launcher.factory, (AccessControlContext)null);
        bcp.initLookupCache((ClassLoader)null);
    }
}
```
下面是我的分析：
1. 当BootClassPath(即System.getProperty("sum.boot.class.path"))时，执行器读取这些文件并加载成File[]
2. 遍历file[]，判断是否为路径，如果不是路径，则获取上层路径（D:/test/1.jar => D:/test）
3. 当不为null时，将其添加到set（用于去重）并判断是否添加成功。如果添加成功，则进入到MetaIndex.registerDirectory将路径传输进去，读取目录下meta-index(用来提供jvm启动的加载速度)，并添加到jarMap里面。
4. initLookupCache这个方法将查找到的classPath缓存到jvm中。其中null对应到的是bootstrapClassloader。这里后面有时间可以深入一下jvm的源码。

## ExtClassloader 
这里提取了部分代码
```java
private static Launcher.ExtClassLoader createExtClassLoader() throws IOException {
    try {
        return (Launcher.ExtClassLoader)AccessController.doPrivileged(new PrivilegedExceptionAction<Launcher.ExtClassLoader>() {
            public Launcher.ExtClassLoader run() throws IOException {
                File[] var1 = Launcher.ExtClassLoader.getExtDirs();
                int var2 = var1.length;

                for(int var3 = 0; var3 < var2; ++var3) {
                    MetaIndex.registerDirectory(var1[var3]);
                }

                return new Launcher.ExtClassLoader(var1);
            }
        });
    } catch (PrivilegedActionException var1) {
        throw (IOException)var1.getException();
    }
}
public ExtClassLoader(File[] var1) throws IOException {
    super(getExtURLs(var1), (ClassLoader)null, Launcher.factory);
    SharedSecrets.getJavaNetAccess().getURLClassPath(this).initLookupCache(this);
}
private static URL[] getExtURLs(File[] var0) throws IOException {
    Vector var1 = new Vector();

    for(int var2 = 0; var2 < var0.length; ++var2) {
        String[] var3 = var0[var2].list();
        if (var3 != null) {
            for(int var4 = 0; var4 < var3.length; ++var4) {
                if (!var3[var4].equals("meta-index")) {
                    File var5 = new File(var0[var2], var3[var4]);
                    var1.add(Launcher.getFileURL(var5));
                }
            }
        }
    }

    URL[] var6 = new URL[var1.size()];
    var1.copyInto(var6);
    return var6;
}
```
这里可以看出几点：
1. ExtClassloader 的父加载器为null
2. 加载路径为java.ext.dirs
3. 初始化方式与BootClassPathHolder有相似的地方，都采用了meta-index记载
4. ExtClassloader 继承了 URLClassloader


## AppClassLoader
```java
public static ClassLoader getAppClassLoader(final ClassLoader var0) throws IOException {
    final String var1 = System.getProperty("java.class.path");
    final File[] var2 = var1 == null ? new File[0] : Launcher.getClassPath(var1);
    return (ClassLoader)AccessController.doPrivileged(new PrivilegedAction<Launcher.AppClassLoader>() {
        public Launcher.AppClassLoader run() {
            URL[] var1x = var1 == null ? new URL[0] : Launcher.pathToURLs(var2);
            return new Launcher.AppClassLoader(var1x, var0);
        }
    });
}

public Class<?> loadClass(String var1, boolean var2) throws ClassNotFoundException {
    int var3 = var1.lastIndexOf(46);
    if (var3 != -1) {
        SecurityManager var4 = System.getSecurityManager();
        if (var4 != null) {
            var4.checkPackageAccess(var1.substring(0, var3));
        }
    }

    if (this.ucp.knownToNotExist(var1)) {
        Class var5 = this.findLoadedClass(var1);
        if (var5 != null) {
            if (var2) {
                this.resolveClass(var5);
            }

            return var5;
        } else {
            throw new ClassNotFoundException(var1);
        }
    } else {
        return super.loadClass(var1, var2);
    }
}
```
这里可以看出：
1. AppClassloader 加载路径为java.class.path
2. 重载了一下loadClass方法
    - 校验class文件
    - 查找类是否存在