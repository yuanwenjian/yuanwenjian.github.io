---
title: elasticsearch内置分词器及字符过滤器
description:  elasticsearch内置分词器及字符过滤器
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

# es内置tokenizer
## Standard Tokenizer
1. **描述**:
标准分析器是Elasticsearch默认使用的分析器。它是分析各种语言文本最常用的选择。它根据 Unicode 联盟 定义的 单词边界 划分文本。删除绝大部分标点
2. **简称**: standard
3. **参数**: max\_token\_length //拆分token最大长度,如果超多最大限度使用max_token_length间隔进行拆分,默认为255
4. **示例**:
```
PUT my_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_analyzer": {
          "tokenizer": "my_tokenizer"
        }
      },
      "tokenizer": {
        "my_tokenizer": {
          "type": "standard",
          "max_token_length": 5
        }
      }
    }
  }
}

POST my_index/_analyze
{
  "analyzer": "my_analyzer",
  "text": "The 2 QUICKED Brown-Foxes jumped over the lazy dog's bone."
}

输出结果
[ The, 2, QUICK, ED, Brown, Foxes, jumpe, d, over, the, lazy, dog's, bone ]
```
## Letter Tokenizer
1. **描述**: 字母标记器只要遇到不是字母的字符就会将文本分解为字词。对于大多数欧洲语言该分词是合理的,但一部分亚洲语言不是使用空格分隔,使用该分词器效果很不友好
2. **简称**: letter
3. **参数**: 无
4. **示例**:
```
POST _analyze
{
  "tokenizer": "letter",
  "text": "The 2 QU3ICK Brown-Foxes jumped over the lazy dog's bone."
}

输出结果
[ The, QU, ICK, Brown, Foxes, jumped, over, the, lazy, dog, s, bone ]
```
## Lowercase Tokenizer
1. **描述**: 与Letter Tokenizer类似,遇到非字母时将文档拆分为token,并且转为小写,功能与letter tokenizer与lowercase filter组合一样,但是效率更高
2. **简称**: lowercase
3. **参数**: 无
4. **示例**
```
POST _analyze
{
  "tokenizer": "lowercase",
  "text": "The 2 QU6ICK Brown-Foxes jumped over the lazy dog's bone."
}

输出结果
[ the, qu, ick, brown, foxes, jumped, over, the, lazy, dog, s, bone ]
```
## Whitespace Tokenizer
1. **描述**: 通过空格将文本拆分为token
2. **简称**: whitespace
3. **参数**: 无
4. **示例**
```
POST _analyze
{
  "tokenizer": "whitespace",
  "text": "The 2 QUICK... Brown-Foxes jumped over the lazy dog's bone."
}

输出结果
[ The, 2, QUICK..., Brown-Foxes, jumped, over, the, lazy, dog's, bone. ]
```
<font color='red'>5. **注意**: </font>拆分token最大长度为255,如果文本超过255,将被分割为255字符
## UAX URL Email Tokenizer
1. **描述**: uax_url_email分词器与标准分词器类似,只是它将url与email单独处理
2. **简称**: uax_url_email
3. **参数**: max_token_length  默认为255
4. **示例**:
```
PUT my_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_analyzer": {
          "tokenizer": "my_tokenizer"
        }
      },
      "tokenizer": {
        "my_tokenizer": {
          "type": "uax_url_email"
        }
      }
    }
  }
}
POST my_index/_analyze
{
  "analyzer": "my_analyzer",
  "text": "My Email is john.smith@global-international.com"
}

[My,  Email , is , john.smith@global-international.com]
```
## Classic Tokenizer
1. **描述**: classic基于语法的tokenizer,适用于英文文档,该分词器对首字母缩略词，公司名称，电子邮件地址和互联网主机名称进行特殊处理具有启发式的作用。但是，这些规则并不总是有效，除英语以外的语言，该分词器大多数不起作用
    * 它以大多数标点符号分割字词，删除标点符号。 不过，后面没有空格的点被认为是 token 的一部分。
    * 它以连字符分隔单词。除非 token 中有一个数字，在这种情况下，整个 token 将被解释为产品编号，并且不分割。
    * 它将电子邮件地址和互联网主机名识别为一个词元。
2. **简称**:  classic
3. **参数**: max_token_length
4. **示例**:
```
POST _analyze
{
  "analyzer": "classic",
  "text": "The 2 QUICK Brown-Foxes 3 jumped over the lazy dog's bone 3-other t. t.o www@baidu.com  tue@"
}
输出结果
[The , 2,  QUICK ,Brown,Foxes, 3 , jumped, over, the, lazy ,dog, bone, 3-other , t , t.o ,www@baidu.com , tue]
```
5. **说明**
- t与t.o ~~对应后面没有空格的点被认为是 token 的一部分~~。
- 3-other未拆分,Brown-Foxes拆分 对应~~token 中有一个数字，在这种情况下，整个 token 将被解释为产品编号，并且不分割~~
- www@baidu.com , tue对比 对应~~将电子邮件地址和互联网主机名识别为一个词元~~
## Thai Tokenizer
1. **描述** :泰语分词器
2. **简称**: thai
3. **参数**: 无
## NGram Tokenizer
1. **描述**:遇到指定字符列表中的某一个时，它首先将文本分解为单词,然后返回指定长度的N-Gram,N-grams 就像一个滑动窗口在单词上移动，是一个连续的指定长度的字符序列,通常用于不使用空格或长复合词的语言,比如德语
2. **简称**: ngram
3. **参数**:
    - min_gram gram最小长度,默认为1
    - max_gram gram最大长度,默认为2
    - token_chars 应包含在词元中的字符类。 Elasticsearch将分割不属于指定类的字符。 默认为[]（保留所有字符）,字符类可能是以下任何一种：
    (需要处理的字符类)
        - letter  如:a, b, ï or 京
        - digit     如:3 or 7
        - whitespace  如: " " or "\n"
        - punctuation   如: ! or "
        - symbol    如:$ or √
    - 提示: 通常将min_gram与max_gram设为一样的值,值越小，匹配到的文档越多，但是匹配的质量越差。值越大，越能匹配到指定的文档。 3 是一个不错的初始值。    
4. **示例**:
```
PUT my_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_analyzer": {
          "tokenizer": "my_tokenizer"
        }
      },
      "tokenizer": {
        "my_tokenizer": {
          "type": "ngram",
          "min_gram": 1,
          "max_gram": 3,
          "token_chars": [
            "letter",
            "digit"
          ]
        }
      }
    }
  }
}

POST my_index/_analyze
{
  "analyzer": "my_analyzer",
  "text": "2 Quick Foxes."
}

输出结果
[ Qui, uic, ick, Fox, oxe, xes ]
```
## Edge NGram Tokenizer
1. **描述**: 当遇到指定字符列表中的一个时，edge_ngram标记器首先将文本分解成单词,然后返回指定长度的每个单词的 N-grams,其中gram的开始处是单词的开始处

    - 当你搜索的类型具有广为人知的顺序,建议搜索使用edge_ngram比ngram更高效,尝试自动填充可能以任何顺序出现的单词时ngram效率更高
2. **简称**: edge_ngram
3. **参数** 与ngram参数相同
4. **示例**:
```
PUT my_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_analyzer": {
          "tokenizer": "my_tokenizer"
        }
      },
      "tokenizer": {
        "my_tokenizer": {
          "type": "edge_ngram",
          "min_gram": 2,
          "max_gram": 10,
          "token_chars": [
            "letter",
            "digit"
          ]
        }
      }
    }
  }
}

POST my_index/_analyze
{
  "analyzer": "my_analyzer",
  "text": "2 Quick Foxes."
}
输出结果
[ Qu, Qui, Quic, Quick, Fo, Fox, Foxe, Foxes ]
```
## Keyword Tokenizer 
1. **描述**:一个“noop”标记器,将整个输入完整一样输出
2. **简称**: keyword
3. **参数**:
    - buffer_size 一次读入术语缓冲区的字符数。默认为256.术语缓冲区将按此大小增长，直到所有文本被消耗完。建议不要更改此设置。
4. **示例**:
```
POST _analyze
{
  "tokenizer": "keyword",
  "text": "New York"
}
输出结果
[ New York ]
```
## Pattern Tokenizer
1. **描述**: 使用正则表达式将文本拆分为与分词匹配的词条，或者将匹配的文本作为词条捕获
2. **简称**: pattern
3. **参数**:
    - pattern
    - flag
    - group
4. **示例**:
```
PUT my_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_analyzer": {
          "tokenizer": "my_tokenizer"
        }
      },
      "tokenizer": {
        "my_tokenizer": {
          "type": "pattern",
          "pattern": ","
        }
      }
    }
  }
}

输出结果
[ comma, separated, values ]
```
## Simple Pattern Tokenizer
1. **描述**:使用正则表达式来捕获匹配的文本作为术语。它支持的一组正则表达式特征比pattern更有限，通常更快。
默认模式是空字符串，它不会生成任何术语。该标记器应始终配置为非默认模式。
2. **简称**: simple_pattern
3. **参数**: pattern 默认为空
4. **示例**:
```
PUT my_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_analyzer": {
          "tokenizer": "my_tokenizer"
        }
      },
      "tokenizer": {
        "my_tokenizer": {
          "type": "simple_pattern",
          "pattern": "[0123456789]{3}"
        }
      }
    }
  }
}

POST my_index/_analyze
{
  "analyzer": "my_analyzer",
  "text": "fd-786-335-514-x"
}
输出结果
[ 786, 335, 514 ]
```
## Simple Pattern Split Tokenizer
1. **描述**: 按照正则指定拆分输入文本,支持的正则表达式比pattern少,但是速度更快
2. **简称**:simple_pattern_split
3. **参数**:pattern
3. **示例**:
```
PUT my_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_analyzer": {
          "tokenizer": "my_tokenizer"
        }
      },
      "tokenizer": {
        "my_tokenizer": {
          "type": "simple_pattern_split",
          "pattern": "_"
        }
      }
    }
  }
}

POST my_index/_analyze
{
  "analyzer": "my_analyzer",
  "text": "an_underscored_phrase"
}

输出结果
[ an, underscored, phrase ]
``` 
## Path Hierarchy Tokenizer
1. **描述**: 把分层的值看成是文件路径，用路径分隔符分割文本，输出树上的各个节点。
2. **简称**: path_hierarchy
3. **参数**:
    - delimiter: 路径分隔符。默认是 / 。
    - replacement: 路径分隔符的替换字符。默认是 delimiter.
    - buffer_size: 一次读到词元缓冲区的字符数。默认是 1024 。 词元缓冲区以此大小增长，只到读完所有的文本。建议不要更改这项配置。
    - reverse: 若为 true ，以相反的顺序发射词元。默认是 false 。
    - skip:忽视的原始词元的数量。默认是0
4. **示例**:
```
PUT my_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_analyzer": {
          "tokenizer": "my_tokenizer"
        }
      },
      "tokenizer": {
        "my_tokenizer": {
          "type": "path_hierarchy",
          "delimiter": "-",
          "replacement": "/",
          "skip": 2
        }
      }
    }
  }
}

POST my_index/_analyze
{
  "analyzer": "my_analyzer",
  "text": "one-two-three-four-five"
}
输出结果
[ /three, /three/four, /three/four/five ]
如果reverse为true,结果为:
[ one/two/three/, two/three/, three/ ]
``` 