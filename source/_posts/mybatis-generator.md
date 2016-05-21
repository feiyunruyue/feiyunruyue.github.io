title: mybatis-generator 生成注解代码
date: 2016-05-21 13:21:07
categories: 编程
tags: mybatis
---
5月18日，正式入职简理财。从体制内大公司来到创业型小公司，开始有些不适应，要学习的东西太多了。趁着还不忙，记录下我学习mybatis-generator的过程。
<!-- more -->
学生期间用过hibernate，听说过mybatis，mybatis-generator可以自动生成sql代码。我选择用maven插件的方式运行generator，要求生成带注解的Mapper类。

1. 在项目pom.xml文件中增加generator插件配置，需要连接mysql数据库，所以这里也把mysql的驱动写上了。

		<build>
			<plugins>
				<plugin>
					<groupId>org.mybatis.generator</groupId>
					<artifactId>mybatis-generator-maven-plugin</artifactId>
					<version>1.3.2</version>
					<dependencies>
						<dependency>
							<groupId>mysql</groupId>
							<artifactId>mysql-connector-java</artifactId>
							<version>5.1.25</version>
						</dependency>
					</dependencies>
				</plugin>
			</plugins>
		</build>

2. 在`src/main/resources`目录下新建`generatorConfig.xml`文件

		<?xml version="1.0" encoding="UTF-8"?>
		<!DOCTYPE generatorConfiguration
		  PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
		  "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">
		<generatorConfiguration>		
		  <context id="mysql_tables" targetRuntime="MyBatis3Simple">
		   
		   <!-- 去掉mybatis生成的注释 -->
		   <commentGenerator>
		        <property name="suppressDate" value="true"/>
		        <property name="suppressAllComments" value="true" />
		    </commentGenerator>
		
		    <jdbcConnection driverClass="com.mysql.jdbc.Driver"
		        connectionURL="jdbc:mysql://127.0.0.1:3306/test"
		        userId="xxx"
		        password="12345678">
		    </jdbcConnection>
		
		    <javaTypeResolver >
		      <property name="forceBigDecimals" value="false" />
		    </javaTypeResolver>
		    
		    <javaModelGenerator targetPackage="com.icbc.model" targetProject="src/main/java">
		      <property name="trimStrings" value="true" />
		    </javaModelGenerator>
		
		    <javaClientGenerator type="ANNOTATEDMAPPER" targetPackage="com.icbc.dao" targetProject="src/main/java">
		    </javaClientGenerator>
		
		    <table tableName="wechat_user" domainObjectName="WechatUser" >
		    	<generatedKey column="id" sqlStatement="Mysql"/>
		    </table>
		
		  </context>
		</generatorConfiguration>
3. 运行maven，Goals里填写`mybatis-generator:generate`

感觉`MyBatis3Simple`就足够了，用`MyBatis3`会生成好多Example代码。

这时候刷新下整个项目，就会发现源代码目录下有了两个Java文件，一个对应mysql表的POJO类，另外一个是映射sql语句的DAO类。

踩到的坑是，无法生成带注解的Java文件，报错`java.lang.ClassNotFoundException: ANNOTATEDMAPPER`，发现是generator版本有点低，我用的是1.3.0，换成1.3.2就可以了。

目前还没有看到mybatis有什么特别好用的地方。

剩下的自己看文档吧。