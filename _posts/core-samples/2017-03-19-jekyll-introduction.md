---
layout: post
title: 静态化博客构建工具
tagline: "jekyll"
description: "jekyll tutorial"
category : jekyll
tags : [beginner, jekyll, markdown]
last_updated: 2017-03-19
---

jekyll是一个简单的免费的Blog生成工具，类似WordPress。但是和WordPress又有很大的不同，原因是jekyll只是一个生成静态网页的工具，不需要数据库支持。但是可以配合第三方服务,例如Disqus。最关键的是jekyll可以免费部署在Github上，而且可以绑定自己的域名

## 概况

### 简单
  无需数据库、评论功能，不需要不断的更新版本——只用关心你的博客内容。
### 静态
  只用 Markdown (或 Textile)、Liquid、HTML & CSS 就可以构建可部署的静态网站。
### 博客形态
  自定义地址、分类、页面、博客内容 以及 自定义的布局设计。  
## 只需几秒钟就能让网站跑起来
Indented code
~~~
$ gem install jekyll

$ jekyll new my-awesome-site

$ cd my-awesome-site

~/my-awesome-site $ jekyll serve

# => Now browse to http://localhost:4000
~~~

## Next Steps
Please take a look at [{{ site.categories.api.first.title }}]({{ BASE_PATH }}{{ site.categories.api.first.url }})
or jump right into [Usage]({{ BASE_PATH }}{{ site.categories.usage.first.url }}) if you'd like.
