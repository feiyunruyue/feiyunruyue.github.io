title: mybatis 拼接字符串问题
date: 2016-08-28 16:13:57
categories: 编程
tags: mybatis
---

mybatis可以通过`@SelectProvider`的方式在Java代码里写sql语句，也还算方便。但是前不久看到公司有人用拼字符串的方式来传参数，如下：
<!--more -->

		WHERE("act_id = '" + request.getActivity() + "'");
或者

		WHERE("act_id = '" + map.get("activity") + "'");

这完全是没有理解mybatis，又倒退回拼字符串的思路上去了。建议有时间再把官方文档看一遍。
优雅的方式是

		 WHERE("act_id = #{activity}");

这样的好处是，

1. 不再拼接字符串了，写起来不容易出错，看着也舒服
2. 能够防止sql注入等问题
3. 使用#{}这种方式能够预编译，效率应该能高一点


哈哈，这种问题。