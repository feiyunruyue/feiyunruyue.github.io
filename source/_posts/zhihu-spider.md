title:  获取知乎推荐问题Java小脚本
date: 2015-03-30 21:52:10
categories: 编程
tags:
---
今天参照[github](https://github.com/callmewhy/ZhihuDown)写了个一百多行的Java小脚本，目前仅仅能爬取[知乎发现](http://www.zhihu.com/explore)上的推荐问题，生成markdown格式的文本，效果大体如下：

>  - [洗手间整理收纳有哪些最佳实践？](http://www.zhihu.com/question/28173041) 

> - [职场菜鸟，自己不擅长的工作任务压到身上，该怎么应对？](http://www.zhihu.com/question/29162468)

> 入职差不多一年，一开始工作还蛮顺利的。可是最近老板经常布置一些自己不擅长的工作给我，而且都是特别重要的，像做代表公司形象的客户提案PPT、产品的新媒体推广方案、搭建网站等等。自己专业不是这一块，根本不懂这些东西，所以觉得压力特别大。感觉自己就算做出来也一定不是老板要求的那样，所以总是无从下手，经常拖延，老板过来询问进度的时候我也只能敷衍过去，不敢跟他们讲我不会做……

<!--more-->

代码很简单，主要利用了正则表达式。
一个Java POJO类，定义了我们后面要用到的知乎问题、URL以及对问题的简单描述：

    public class Zhihu {
		private String questionTitle;// 问题标题
		private String questionDetail;// 问题描述
		private String questionUrl;// 问题url

      		//getters and setters
    }

知乎采用REST风格的URL，其实我们只需要获取问题的id即可。

比如[http://www.zhihu.com/question/29097694](http://www.zhihu.com/question/29097694)这个问题Html源码为

	<div id="zh-question-title" data-editable="true">	
	<h2 class="zm-item-title zm-editable-content">	
	非985 211大学的计算机系本科生进不了腾讯、网易、百度这些公司工作吗？	
	</h2>
	</div>
	<div id="zh-question-detail" class="zm-item-rich-text" data-resourceid="3821992" data-action="/question/detail">	
	<div class="zm-editable-content">今天同学去南大的招聘会，回来很失望地告诉我，网易在南京只招南大和东南的毕业生。<br>而且据说投简历的时候抬头就要写自己是什么大学的，大学比较垃圾的都是直接pass掉，连笔试面试机会都没有。这是真的吗？听很多人说，毕业后第一份工作一定要去大公司，否则以后很难发展。像我这种非211学校的苦学3年计算机，真的就没戏了吗？</div>	
	</div>

要想解析出问题标题来，需要用正则表达式匹配`zh-question-title`这个id下的文字，可以使用下面的正则表达式：
> zh-question-title.+?<h2.+?>(.+?)</h2>

`+?`中的问号表明是最小匹配

同理，问题描述可以用下面的正则表达式匹配：
> zh-question-detail.+?<div.+?>(.+?)</div>

可以试下把最后一个问号去掉是什么效果（它会贪婪地往下匹配）。

代码逻辑如下：

	//匹配问题title
	private static final String QUESTION_TITLE = "zh-question-title.+?<h2.+?>(.+?)</h2>";
	private static final Pattern patternTitle = Pattern.compile(QUESTION_TITLE);
	
	Matcher matcher = patternTitle.matcher(Html源码);
	String questionTitle = null;
	if(matcher.find()){
		questionTitle = matcher.group(1);
	}

matcher.group(0)匹配的是正则表达式本身！

OK，可以运行了！以后慢慢加功能！

[源码在此！](https://github.com/feiyunruyue/zhihu/tree/master/Spider/src)