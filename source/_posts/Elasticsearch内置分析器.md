---
title: Elasticsearch内置分析器
description: 分析器（无论是内置还是自定义）只是一个包含三个较底层组件的包：字符过滤器，分词器，和令牌过滤器。内置分析器将这些组件预先封装成适用于不同语言和文本类型的分析器。Elasticsearch还暴露了各个组件，以便它们可以组合起来定义新的定制分析器。
date: 2018-03-10 20:40:00
comments: true
tags: 
    - elasticsearch
categories:
    - elasticsearch
    - 后端
---

# 概念介绍
具体请参考[elasticsearch内置分词器及字符过滤器][概念介绍]

# es内置分析器
内置分析器可以直接使用，无需任何配置。然而，其中一些配置选项用来改变其行为
## Standard Analyzer
standard（标准）分析器划分文本是通过词语来界定的，由Unicode文本分割算法定义。它删除大多数标点符号，将词语转换为小写，并支持删除停止词。
standard(标准)分析器是默认分析器，如果没有指定分析器，则使用该分析器

### 定义
- 分词器
    - 标准分词器 standard
- 过滤器
    - 标准过滤器 standard
    - 小写过滤器 lowercase
    - 停词过滤器 stop  默认情况禁用

### 配置可选项
1. max_token_length  //token最大长度,如果超多最大限度使用max_token_length间隔进行拆分,默认为255
2. stopwords // 预定义的停止词列表，如_english或包含停止词列表的数组。默认为\_none_。
3. stopwords_path 停词路径 每个字单独一行,编码格式为UTF-8

### 示例
```
PUT my_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_english_analyzer": {
          "type": "standard",
          "max_token_length": 5,
          "stopwords": "_english_"
        }
      }
    }
  }
}

POST my_index/_analyze
{
  "analyzer": "my_english_analyzer",
  "text": "The 2 QUICK Brown-Foxes jumped over the lazy dog's bone."
}

结果
[ 2, quick, brown, foxes, jumpe, d, over, lazy, dog's, bone ]
```
## Simple Analyzer
simple（简单）分析器每当遇到不是字母的字符时，将文本分割为词语。它将所有词语转换为小写。

### 定义
- 分词器
    - 小写分词器 lowercase

### 示例
```
POST _analyze
{
  "analyzer": "simple",
  "text": "The 2 QUICK Brown-Foxes jumped over the lazy dog's bone."
}
结果
[ the, quick, brown, foxes, jumped, over, the, lazy, dog, s, bone ]
```
## Whitespace Analyzer
whitespace（空白）分析器每当遇到任何空白字符时，都将文本划分为词语。它不会将词语转换为小写。

### 定义
- 分词器
    - 空白分词器 whitespace

### 示例
```
POST _analyze
{
  "analyzer": "whitespace",
  "text": "The 2 QUICK Brown-Foxes jumped over the lazy dog's bone."
}

结果
[ The, 2, QUICK, Brown-Foxes, jumped, over, the, lazy, dog's, bone. ]
```
## Stop Analyzer
stop（停止）分析器类似simple(简单)分析器，默认添加_english_停用词

### 配置可选项
1. stopwords 停用词列表
2. stopwords_path 停用词路径

### 示例
```
PUT my_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_stop_analyzer": {
          "type": "stop",
          "stopwords": ["the", "over"]
        }
      }
    }
  }
}
  
POST my_index/_analyze
{
  "analyzer": "my_stop_analyzer",
  "text": "The 2 QUICK Brown-Foxes jumped over the lazy dog's bone."
}
结果
[ quick, brown, foxes, jumped, lazy, dog, s, bone ]
```

## Keyword Analyzer
keyword（关键字）分析器是一个noop（空）分析器，可以接受任何给定的文本，并输出与单个词语相同的文本。
### 定义
- 分词器
    - keyword 

### 示例
```
POST _analyze
{
  "analyzer": "keyword",
  "text": "The 2 QUICK Brown-Foxes jumped over the lazy dog's bone."
}

结果
[ The 2 QUICK Brown-Foxes jumped over the lazy dog's bone. ]
```

## Pattern Analyzer
pattern（模式）分析器使用正则表达式将文本拆分为词语，它支持小写和停止字。
### 定义
- 分词器
    - pattern
- 过滤器
    - 小写过滤器
    - 停词过滤器 默认禁用

### 配置可选项

1. pattern java正则表达式 默认 \w
2. flags 
3. lowercase 是否转为小写 默认为true
4. stopwords 预先定义的停用词列表，如_english_或包含停用词列表的数组,默认为_none_
5. stopwords_path 停词文件列表

### 示例

```
PUT my_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_email_analyzer": {
          "type":      "pattern",
          "pattern":   "\\W|_", 
          "lowercase": true
        }
      }
    }
  }
}

POST my_index/_analyze
{
  "analyzer": "my_email_analyzer",
  "text": "John_Smith@foo-bar.com"
}

结果
[ john, smith, foo, bar, com ]
```
## Language Analyzers
Elasticsearch提供许多特定语言的分析器，例如english(英语)或french(法语)。
## Fingerprint Analyzer
fingerprint（指纹）分析器是专门的分析器，可以创建用于重复检测的指纹。将重复的token删掉并重新排序生成一个新的token
### 定义
- 分词器
    - 标准分词器
- 过滤器
    - 小写过滤器
    - asciifolding 过滤器
    - 停词过滤器
    - fingerprint 过滤器

### 配置可选项
1. separator 用于拼接的字符,默认为空格
2. max_output_size token最大长度,默认255,大于该值的token丢弃
3. stopwords 停词列表,默认禁用
4. stopwords_path 停词文件路径

### 示例
```
PUT my_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_fingerprint_analyzer": {
          "type": "fingerprint",
          "stopwords": "_english_"
        }
      }
    }
  }
}
 
POST my_index/_analyze
{
  "analyzer": "my_fingerprint_analyzer",
  "text": "Yes yes, Gödel said this sentence is consistent and."
}
结果
[ consistent godel said sentence yes ]
```
## Custom Analyzer
如果你没有找到适合你需求的分析器，则可以创建一个自定义分析器，结合适当的字符过滤器，分词器和词语过滤器。

### 配置
1. tokenizer 分词器,内置或者自定义的tokenizer 必需
2. char_filter 内置或自定义的字符过滤器
3. filter 内置或自定义的token过滤器

### 内置tokenizer和filter,char_filter示例
```
PUT my_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_custom_analyzer": {
          "type":      "custom",
          "tokenizer": "standard",
          "char_filter": [
            "html_strip"
          ],
          "filter": [
            "lowercase",
            "asciifolding"
          ]
        }
      }
    }
  }
}
 
POST my_index/_analyze
{
  "analyzer": "my_custom_analyzer",
  "text": "Is this <b>déjà vu</b>?"
}
结果

[ is, this, deja, vu ]
```
### 自定义tokenizer和filter,char_filter示例
```
PUT my_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_custom_analyzer": {
          "type": "custom",
          "char_filter": [
            "emoticons"
          ],
          "tokenizer": "punctuation",
          "filter": [
            "lowercase",
            "english_stop"
          ]
        }
      },
      "tokenizer": {
        "punctuation": {
          "type": "pattern",
          "pattern": "[ .,!?]"
        }
      },
      "char_filter": {
        "emoticons": {
          "type": "mapping",
          "mappings": [
            ":) => _happy_",
            ":( => _sad_"
          ]
        }
      },
      "filter": {
        "english_stop": {
          "type": "stop",
          "stopwords": "_english_"
        }
      }
    }
  }
}
 
POST my_index/_analyze
{
  "analyzer": "my_custom_analyzer",
  "text":     "I'm a :) person, and you?"
}
结果
[ i'm, _happy_, person, you ]
```
