title: 使用java爬取草榴图片
date: 2016-01-23 21:34:12
categories: 编程
tags: 爬虫
---
先用标题把人吸引过来，然后再搞个大新闻，嗯，就这么办。效果图如下：

![](/images/caoliu.png)


<!-- more -->

使用了[前面博客](/2016/01/11/java-spider/)提到的爬虫框架webmagic，再次安利一下这个国人写的框架，我写完爬草榴图片的脚本后数了下，所有代码加起来不超过200行，而且分层比较明确，看着舒服。

核心代码就是一个CaoLiuPageProcessor，打开页面，简单分析一下就能写出来。

		//获取当前页面上的帖子地址
		List<String> postUrls = page.getHtml().xpath("//tr[@class='tr3 t_one']/td/h3/a/@href").all();
		//将帖子地址加入要抓取的url队列
		page.addTargetRequests(postUrls);
		
		//抓取其他页
		List<String> otherPageUrls = page.getHtml().links().regex(".*thread0806.php\\?fid=8&search=&page=\\d+").all();
		page.addTargetRequests(otherPageUrls);
		
		//进入帖子后，获取帖子标题
		String postName = page.getHtml().xpath("//tr[@class='tr1 do_not_catch']/th/table/tbody/tr/td/h4/text()").toString();
		if(postName != null){
			logger.info(postName + " " + page.getUrl());
			//进入帖子之后，帖子内容，即要抓取的图片url
			List<String> postContent = page.getHtml().xpath("//div[@class='tpc_content do_not_catch']/input/@src").all();
			page.putField("postContent", postContent);
			page.putField("postName", postName);
		}

CaoLiuPageProcessor负责抓取url，解析帖子，这里我们没有处理解析后的内容，真正保存解析后的内容单独放在了ImgPipeline中。模块划分很严格，解析后的有用信息你愿意写入文件就写入文件，愿意记录到数据库就记录到数据库，解耦了。

	@Override
	public void process(ResultItems resultItems, Task task) {
	     Map<String, Object> fields = resultItems.getAll();
            List<String> postContent = (List<String>) fields.get("postContent");
            if(postContent != null){
            	String postName = (String) fields.get("postName");
				...
			}
	}

webmagic处理流程，下载代码-->页面解析-->处理解析结果：

    Page page = downloader.download(request, this);
    pageProcessor.process(page);
    if (!page.getResultItems().isSkip()) {
        for (Pipeline pipeline : pipelines) {
            pipeline.process(page.getResultItems(), this);
        }
    }


大功告成！源码在此，拿去用吧。
[https://github.com/feiyunruyue/myspider](https://github.com/feiyunruyue/myspider)

最后附上一张爬下来的图片，振奋下人心。
![](/images/elizabeth-marxs-wild-life-12.jpg)