title: jdk版本兼容问题
date: 2018-08-28 14:56:12
categories: 编程
tags:
---

低版本jdk编译的class文件可以在高版本jre运行，高版本jdk编译的class文件不能在低版本的jre运行。

<!-- more -->

java的class文件，前4个字节为魔数（Magic Number）, 0xCAFEBABE，用来确定这是一个class文件。

第5,6个字节是次版本号(Minor Version)。

第7,8个字节是主版本号（Major Version）。

jdk版本号跟class主版本号对应关系如下：

> 
Java SE 10 = 54 (0x36 hex),
Java SE 9 = 53 (0x35 hex),
Java SE 8 = 52 (0x34 hex),
Java SE 7 = 51 (0x33 hex),
Java SE 6.0 = 50 (0x32 hex),
Java SE 5.0 = 49 (0x31 hex),
JDK 1.4 = 48 (0x30 hex),
JDK 1.3 = 47 (0x2F hex),
JDK 1.2 = 46 (0x2E hex),
JDK 1.1 = 45 (0x2D hex).

本地测试了下，写了个简单的hello world, 使用jdk10进行编译，然后用jdk8运行，报错如下：

```
Error: A JNI error has occurred, please check your installation and try again
Exception in thread "main" java.lang.UnsupportedClassVersionError: HelloWorld has been compiled by a more recent version of the Java Runtime (class file version 54.0), this version of the Java Runtime only recognizes class file versions up to 52.0
        at java.lang.ClassLoader.defineClass1(Native Method)
        at java.lang.ClassLoader.defineClass(Unknown Source)
        at java.security.SecureClassLoader.defineClass(Unknown Source)
        at java.net.URLClassLoader.defineClass(Unknown Source)
        at java.net.URLClassLoader.access$100(Unknown Source)
        at java.net.URLClassLoader$1.run(Unknown Source)
        at java.net.URLClassLoader$1.run(Unknown Source)
        at java.security.AccessController.doPrivileged(Native Method)
        at java.net.URLClassLoader.findClass(Unknown Source)
        at java.lang.ClassLoader.loadClass(Unknown Source)
        at sun.misc.Launcher$AppClassLoader.loadClass(Unknown Source)
        at java.lang.ClassLoader.loadClass(Unknown Source)
        at sun.launcher.LauncherHelper.checkAndLoadMain(Unknown Source)
```        

解决方案，要么升自己的jdk，要么降别人的jdk。哈哈，别人的估计降不了啊。

查资料说java的泛型不是真正的泛型，运行期会被擦除，一个原因就是为了版本兼容。