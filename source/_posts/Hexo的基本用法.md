---
title: Hexo的基本用法
date: 2017-01-14 21:42:56
categories: Hexo  
tags: [Hexo]
---

# Hexo的基本用法
## 基础
最基础的，创建文件：hexo new [layout] “文件名”， 其中layout默认为post。
## layout
Hexo提供了3中布局，分别是post、page和draft。路径对应的分别是_post和_page还有_draft。

## 文件名
Hexo会默认使用文章的标题作文文件名，可以编辑_config.yml文件来修改默认的文件名

### 其他
- title 文章标题
- year 创建年份
- month 月份，如4月为04
- day 日期
- i_month 月份，单数字，比如4月就是4
- i_day 同上

## Drafts
在drafts目录下的文档会视为草稿，不会发布到网站上。可以通过hexo publish [layout] "文件名" 来发布位于该目录的草稿。也可以通过修改_config.yml文件来使该目录下的文章也可以默认发布。

## Scaffolds模版
当你使用new命令创建一篇文章的时候，Hexo会根据scaffolds目录中的模版帮你生成文章。假如执行hexo new photo "My Gallery"，Hexo会尝试在scaffolds目录中去寻找photo.md的模版文件，然后基于它创建标题为My Gallery的文章。

# 前置声明
文章开头title、data、categroise、tags等一部分内容。这一部分有两种书写方式
- [YAML](http://baike.baidu.com/link?url=a2b9mgj0dYJO0wNYGDX5LVBGZ00QUZpzpbyiJ0dSKm_xuicUfvX6lRUpwgWw7kTAhENjlQhJJGrpZDl3BHIj8_)方式，三短线结束。
- [JSON](http://baike.baidu.com/link?url=Isxbt-o3vGV0KGGsQuZpzR7JXOTcelfSwnFB_xSpB_75Fv-eF5YAlsJm81dbJspHUIELJ2wrWyplvhz_mbYTn_)方式，三分号结束。
## 前置声明预置的参数

| 名称        | 说明    |  
| --------   | -----:   |
| layout        | 布局，一般使用默认值。详见Hxeo基本用法->基础|   
| title        |标题      |   
| date        | 时间      |   
| updated        | 更新时间      |   
| comments        | 是否开启评论，默认为true      |   
| tags        | 文章标签      |   
| categories        | 文章分类      |   
| permalink        | 文章永久链接，一般不用写，默认就行      |   
# 标签插件
标签插件和 Front-matter 中的标签不同，它们是用于在文章中快速插入特定内容的插件
## 引用块
在文章中插入引言，可包含作者、来源和标题。

别号： quote

样例：

```

{% blockquote David Levithan, Wide Awake %}

Do not just seek happiness for yourself. Seek happiness for all. Through kindness. Through mercy.

{% endblockquote %}

```
显示效果

{% blockquote David Levithan, Wide Awake %}
Do not just seek happiness for yourself. Seek happiness for all. Through kindness. Through mercy.
{% endblockquote %}

{% youtube video_id %}
