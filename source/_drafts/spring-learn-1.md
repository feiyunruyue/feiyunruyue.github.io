spring源码学习笔记

以前的项目仅仅用过SSH框架，这次想看下spring的源码，提升下自己的功力。

先用一个小例子复习下spring。

定义beans的xml元素需要补充XML Schema（替代了之前老的DTD方式），spring的xsd文件可以在jar包中找到，/org/springframework/beans/factory/xml/spring-beans-4.2.xsd

自己实现的话，需要设计一个类读取xml文件，将bean的name和class放到map里，那spring又是如何实现的呢。