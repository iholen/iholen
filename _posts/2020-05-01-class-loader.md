---
layout: post
title:  "JVM之类加载子系统"
date:   2020-05-01 16:18:50 +0800
categories: jvm
author: iholen
introduction: JVM之类加载子系统
---
## 类加载子系统

java程序运行时，首先会加载mian方法所在的类，在主类加载或程序运行过程中，如果使用到其他类，则会去加载这些类。所以java中的类不是一次性加载到内存中的。

#### 类加载过程

|过程|说明|
|:---|:---|
|加载|将磁盘上的字节码文件读取到jvm中|
|验证|校验字节码文件的正确性|
|准备|给静态变量分配内存空间，并初始化默认值|
|解析|将符号引用替换为直接引用，这就是静态链接(类加载期间完成)。动态链接是在程序运行期间完成的|
|初始化|给静态变量赋值，并执行静态代码块 |

**`Tips`**:

```
// VM OPTIONS - 用于查看类加载情况
-verbose:class
```

#### 类加载器

|加载器|说明|
|:---|:---|
|启动类加载器|加载lib目录下的核心类库，如rt.jar等|
|扩展类加载类|主要加载lib/ext目录下的类库|
|应用程序类加载器|加载classPath路径下的类|
|自定义类加载器|加载自定义路径下的类|

* 实现自定义类加载器
若需要加载自定义路径下的类时，则需要实现自定义类加载器。
实现方法如下：
    1. 继承`ClassLoader`
    2. 重写`findClass`方法

```java
public class MyClassLoader extends ClassLoader {

    private String classPath;

    MyClassLoader(String classPath) {
        this.classPath = classPath;
    }

    private byte[] loadByte(String name) throws Exception {
        name = name.replaceAll("\\.", "/");
        FileInputStream fis = new FileInputStream(classPath + "/" + name
                + ".class");
        int len = fis.available();
        byte[] data = new byte[len];
        fis.read(data);
        fis.close();
        return data;
    }

    @Override
    public Class<?> findClass(String name) throws ClassNotFoundException {
        try{
            byte[] data = loadByte(name);
            return defineClass(name, data, 0, data.length);
        } catch (Exception e) {
            e.printStackTrace();
            throw new ClassNotFoundException();
        }
    }

}
```

* 双亲委派机制

    * **加载优先级**：<br>
    启动类加载器 > 扩展类加载器 > 应用类加载器 > 自定义类加载器<br>
    若优先级高的类加载器可以成功加载，则直接加载。
    * **优点**<br>
        1. 可以防止核心API被随意修改。(沙箱安全机制)
        2. 防止重复加载。
    
* 打破双亲委派

Tomcat的WebappClassLoader






