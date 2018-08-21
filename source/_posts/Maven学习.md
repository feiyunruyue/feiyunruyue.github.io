title: Maven学习
date: 2018-08-20 16:50:46
categories: 编程
tags: maven
---
程序员每天除了写代码，还有很大一部分时间花在了编译、打包、运行、部署这些构建工作上。现在每天都在用maven，反而感觉不到maven的重要性了。

<!-- more -->

maven使用groupId、artifactId、version唯一标识一个项目。

mvn dependency:tree 命令可以列出项目依赖树，还挺实用。

mvn clean install -Dmaven.test.skip=true 可以跳过单元测试。

pom.xml文件中指定不同的profile，可以指定开发、测试等环境

```
<profiles>
  <profile>
    <id>dev1</id>
    <properties>
        <profile.dir>dev1</profile.dir>
    </properties>
    <activation>
        <activeByDefault>true</activeByDefault>
    </activation>
  </profile>
  <profile>
    <id>dev2</id>
    <properties>
        <profile.dir>dev2</profile.dir>
    </properties>
  </profile>
</profiles>

```

执行命令时加上-P参数

```
mvn clean install -Pdev2 
```

SNAPSHOT 测试版本，开发的时候用，发布到生产的使用RELEASE版本。

依赖范围有以下几种：

1. compile(默认)- 编译
2. test - 测试
3. provided - 在编译测试阶段使用，打包的时候可以不用包进去，典型的例子是servlet，容器会提供这个依赖
4. runtime - 运行时范围，比如jdbc驱动

exclusions 排除传递依赖。

其他的用到的时候再查吧。

