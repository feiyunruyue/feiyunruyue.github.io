title: java 爬虫入门
date: 2016-01-11 21:16:58
categories: 编程
tags: 爬虫
---
很久很久以前我就想搞个爬虫试试，但直到最近才真正开始动手尝试，总结原因就是一个“懒”字。前一篇博客[《获取知乎推荐问题Java小脚本》](http://feiyunruyue.github.io/2015/03/30/zhihu-spider/)，可以算是爬虫的入门Demo，脚本完成了下载html代码、通过正则表达式提取信息、将提取的信息打印到控制台这些基本的功能，麻雀虽小，但也算是个爬虫雏形，先把这个小Demo搞一遍而不是直接上手爬虫框架，有利于我们理解爬虫的原理。
<!-- more -->
爬虫的原理可以在网上搜一下，我简单说下我自己的理解。

>给定一个入口url，将页面下载到本地，解析整个页面，将解析后的结果处理后保存，解析过程中不断发现新的url，重复这个过程。

当然，好多重要的细节都被我省略掉了，但是知道这些基本就可以开始写代码了。写吧，写完再改。

隆重推荐一款良心的java爬虫框架——[webmagic](http://webmagic.io/)。设计参考了Scrapy，实现采用了java常见的类库，由4个组件（Downloader、PageProcessor、Scheduler、Pipeline）构成，甚合我意。Downloader负责下载页面， PageProcessor负责分析页面提取信息，Scheduler负责调度，Pipeline负责善后，保存我们提取出的信息。柿子从最软的开始捏，我们也先从最简单的开始，只需实现PageProcessor接口。


	import us.codecraft.webmagic.Page;
	import us.codecraft.webmagic.Site;
	import us.codecraft.webmagic.processor.PageProcessor;
	
	public class ZhihuProcessor implements PageProcessor{
		  private Site site = Site.me().setCycleRetryTimes(5).setRetryTimes(5).setSleepTime(500).setTimeOut(3 * 60 * 1000)
		            .setUserAgent("Mozilla/5.0 (Windows NT 6.1; WOW64; rv:38.0) Gecko/20100101 Firefox/38.0")
		            .addHeader("Accept", "text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8")
		            .addHeader("Accept-Language", "zh-CN,zh;q=0.8,en-US;q=0.5,en;q=0.3")
		            .setCharset("UTF-8");
	
		@Override
		public void process(Page page) {
			
		}
	
		@Override
		public Site getSite() {
			return site;
		}
	}

开始的写法都是比较固定的，真正需要我们自己去花心思的是·`process`方法。

	@Override
	public void process(Page page) {
	        List<String> answerUrls =  page.getHtml().$(".question_link").links().regex("/question/\\d+/answer/\\d+").all();
	        page.addTargetRequests(answerUrls);
	        
	        List<String> answers = page.getHtml().xpath("//div[@id='zh-question-answer-wrap']/div").all();
	        logger.debug(page.getUrl());
	        for(String answer: answers){
	           page.putField("answer", new Html(answer).xpath("//div[@class='zm-editable-content]/html()").toString());
	        }
	}

首先选中所有class为question_link的元素，links方法负责提取所有的url，然后再用regex正则表达式限定一下，这样就能将本页的所有答案的url提取出来。`page.addTargetRequests(answerUrls);`负责把这些url加到抓取队列里。剩下的代码负责提取答案url里的内容。选取页面元素用到了XPath，这是w3c当年制定的一项标准（以前没注意这项技术，以为跟xhtml一样鸡肋的，没想到啊），XPath语法超简单，[网上的教程](http://www.w3school.com.cn/xpath/xpath_syntax.asp)就够用，但是真是实用！

  细心的同学应该会注意到，我们没有在PageProcessor处理提取出来的answer信息，以后会继续说，入门Demo不展示PipeLine。webmagic新增了一个PipeLine接口，专门用来处理PageProcessor提取出来的信息。这样有些好处，最起码第一印象，层次分明了，每个类分工比较清晰，符合设计模式里说的“单一职责”原则。把一件事情干好就不错了！

如何运行呢，看我们的main方法。

晕，想上网查点webmagic的资料的，结果一不小心浏览了半个多小时网页。言归正传，webmagic的入口是一个Spider类，该类实现了Runnable接口，包含了上面我们说过的4大组件，

	Spider.create(new ZhihuProcessor())
				.addUrl("https://www.zhihu.com/people/wang-liang-50-73/answers?page=1")
				.addPipeline(new ConsolePipeline())
				.thread(3)
				.run();

运行一下，可以看到控制台打印出了我们提取出的知乎答案。

大功告成。动手试一下吧。