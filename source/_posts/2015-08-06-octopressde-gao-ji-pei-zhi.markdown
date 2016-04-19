---
layout: post
title: "octopress的高级配置"
date: 2015-08-06 16:40:03 +0800
comments: true
categories: octopress
---

##前言
上篇博客主要讲了怎么样搭建octopress博客，今天主要讲解个性化设置cotopress.

##添加侧边栏分类
1.在`plugins`目录下创建`category_list_tag.rb`文件，内容如下:
<!--more-->
   
    module Jekyll 
	  class CategoryListTag < Liquid::Tag 
	    def render(context) 
	      html = "" 
	      categories = context.registers[:site].categories.keys 
	      categories.sort.each do |category| 
	        posts_in_category = context.registers[:site].categories[category].size 
	        category_dir = context.registers[:site].config['category_dir'] 
	        category_url = File.join(category_dir, category.gsub(/_|\P{Word}/, '-').gsub(/-{2,}/, '-').downcase) 
	        html << "<li class='category'><a href='/#{category_url}/'>#{category} (#{posts_in_category})</a></li>\n" 
	      end 
	      html 
	    end 
	  end 
	end

	Liquid::Template.register_tag('category_list', Jekyll::CategoryListTag)
	
2.添加`source/_includes/asides/category_list.html`文件，内容如下:
![MacDown logo](http://7xkxhx.com1.z0.glb.clouddn.com/step12.png)
			
		
3.修改`_config.yml`文件，在`default_asides`项中添加`asides/category_list.html`,多个文件用逗号分隔开，先后顺序代表了右边侧边栏显示的先后顺序。如下图
![MacDown logo](http://7xkxhx.com1.z0.glb.clouddn.com/b2_step1.png)
在侧边栏还可以添加其他组件，比如统计，微博，等等。
##添加分享
octopress自带的分享twitter,facebook等，都是国外，需要翻墙才能访问，其实国内也有很多好的分型的工具，比如我用的就是 [友言](http://www.jiathis.com),把获取到的代码复制到 `source/_includes/post/sharing.html`中，如图：![](http://7xkxhx.com1.z0.glb.clouddn.com/b2_step2.png)
##添加评论
自带的评论是disqus，也是国内被墙的原因，国内也有很多评论组件，我用的也是友言，把获取到的代码复制到`source/_includes/post/sharing.html`，跟分享一样，如图：![](http://7xkxhx.com1.z0.glb.clouddn.com/b2_step3.png)

##文章以摘要形式展示
1.在文章对应的markdown文件中，在需要展示首页文字的后面加上`<!--more-->`,重新生成和部署后，就可以看到<!-more->
前面的文字了，文字后面系那是`Rede on`点击显示文章的详细内容。
2.如果想把Read on修改成中文，可以修改_config.yml文件
		
	#excerpt_link: "Read on &rarr;"  # "Continue reading" link text at the bottom of excerpted articles
	excerpt_link: "阅读全文&rarr;"  # "Continue reading" link text at the bottom of excerpted articles
	
##添加访客统计
我的访客统计使用的是[FlagCount](http://www.flagcounter.com),可以先去获取代码，然后添加到`.\source\_includes\custom\asides\flag_counter.html`,如果没有该文件，就创建一个,文件内容如下：

		<section>
		  <h1>访客统计</h1>
		  <br/>
		<a href="http://info.flagcounter.com/Bkif"><img src="http://s01.flagcounter.com/count2/Bkif/bg_ffffff/txt_000000/border_CCCCCC/columns_3/maxflags_10/viewers_0/labels_1/pageviews_1/flags_1/percent_0/" alt="Free counters!" border="0"></a>
		</section>
将页面添加到侧边栏,在_config.yml配置文件中，添加边栏配置
	
`default_asides: [asides/recent_posts.html, asides/github.html,
 asides/delicious.html, asides/pinboard.html,
  asides/googleplus.html, custom/asides/about.html,
   asides/category_list.html,  custom/asides/flag_counter.html]` 
   
   最后添加控制开关，在_config.yml配置文件中添加配置:
   
     # Flag Counter
       flag_counter: true
   
##添加返回顶部按钮
如果文章太长的话，添加一个返回到顶部的按钮是很实用的

### *创建一个js文件，位置在`source/javascript/top.js`,文件内容如下* <br>


~~~
function goTop(acceleration, time)
{
        acceleration = acceleration || 0.1;
        time = time || 16;

        var x1 = 0;
        var y1 = 0;
        var x2 = 0;
        var y2 = 0;
        var x3 = 0;
        var y3 = 0;

        if (document.documentElement)
        {
                x1 = document.documentElement.scrollLeft || 0;
                y1 = document.documentElement.scrollTop || 0;
        }
        if (document.body)
        {
                x2 = document.body.scrollLeft || 0;
                y2 = document.body.scrollTop || 0;
        }
        var x3 = window.scrollX || 0;
        var y3 = window.scrollY || 0;

        var x = Math.max(x1, Math.max(x2, x3));
        var y = Math.max(y1, Math.max(y2, y3));

        var speed = 1 + acceleration;
        window.scrollTo(Math.floor(x / speed), Math.floor(y / speed));

        if(x > 0 || y > 0)
        {
                var invokeFunction = "goTop(" + acceleration + ", " + time + ")";
                window.setTimeout(invokeFunction, time);
        }
}
~~~



### 自定义返回按钮格式
在`source/_include/custom/footer.html`中添加如下代码

	<!--返回顶部开始-->
	<div id="full" style="width:0px; height:0px; position:fixed; right:180px; bottom:150px; z-index:100; text-align:center; background-color:transparent; cursor:pointer;">
	  <a href="#" onclick="goTop();return false;"><img src="/images/top.png" border=0 alt="返回顶部"></a>
	</div>
	<script src="/javascripts/top.js" type="text/javascript"></script>
	<!--返回顶部结束-->
####选择按钮的图片
找一张自己喜欢的图片，取名为top.png,我是把图片放到了 [七牛](https://portal.qiniu.com)上去托管，上面代码中图片的路径改为 七牛上的图片路径就行
![](http://7xkxhx.com1.z0.glb.clouddn.com/b2_step4.png)



