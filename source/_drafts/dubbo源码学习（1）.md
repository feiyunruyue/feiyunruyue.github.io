title: dubbo源码学习（1）
date: 2018-09-17 15:04:54
categories: 编程
tags: dubbo
---
dubbo是一个高性能的RPC框架。好多互联网公司在用，研究下代码还是很有好处的。以2.6.0版本为例。

## 问题：dubbo如何与spring结合的

看代码之前，我先想的问题是，项目启动的时候，dubbo是怎么启动的。带着问题，打开dubbo源码，在META_INF/目录下发现有个文件spring.handlers，里面的DubboNamespaceHandler继承了spring的NamespaceHandlerSupport，这个类就可以解析dubbo自定义的标签，将各个标签解析为Bean，比如<dubbo:application/>被解析成ApplicConfig。dubbo自定义的标签里有两个最重要的，<dubbo:service/> 和 <dubbo:reference/>，分别负责服务暴露和服务引用。


## 问题： dubbo服务是如何暴露的

解析到<dubbo:service/>标签时，会生成一个ServiceBean，

ExtensionLoader.getExtensionLoader(T).getExtension(name) 获取真正的实现类
