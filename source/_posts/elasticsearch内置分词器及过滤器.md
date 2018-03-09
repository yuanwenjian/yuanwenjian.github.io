---
title: Elasticsearch内置分词器及过滤器
description:  Elasticsearch内置分词器及过滤器
date: 2018-03-09 20:40:00
comments: true
tags: 
    - elasticsearch
categories:
    - elasticsearch
    - 后端
---
# 概念介绍

## Character filter(字符过滤器)
首先，字符串按顺序通过每个 字符过滤器 。他们的任务是在分词前整理字符串。一个字符过滤器可以用来去掉HTML，或者将 & 转化成 `and`。
## Tokenizer(分词器)
其次，字符串被 分词器 分为单个的词条。一个简单的分词器遇到空格和标点的时候，可能会将文本拆分成词条。
## Token filter(Token 过滤器)
最后，词条按顺序通过每个 token 过滤器 。这个过程可能会改变词条（例如，小写化 Quick ），删除词条（例如， 像 a`， `and`， `the 等无用词），或者增加词条（例如，像 jump 和 leap 这种同义词）。
## Analyzer(分析器)
整个步骤为Analyzer(分析器)

## Token(词元)
文档通过Tokenizer 生成Token

## Term
被分析器处理后的结果为Term
## Frequency(词频)
文档中包含几个 Term成为Frequency

# es内置 Character filter
|过滤器|简称|描述|支持参数|
|---|---|---|--|
|HTML Strip Char Filter|html\_strip|去除HTML元素|escaped_tags(排除的标签数组)
|Mapping Char Filter| mapping | 根据配置的映射配置| mappings_path(一个key => value特定格式的文件路径,相对或config文件夹)
|Pattern Replace Char Filter| pattern_replace | 使用java正则替换| pattern,replacement,flags|
