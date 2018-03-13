---
title: Elasticsearch内置过滤器
description:  Elasticsearch过滤器
date: 2018-03-12 22:15:00
comments: true
tags: 
    - elasticsearch
categories:
    - elasticsearch
    - 后端
---
# 概念介绍
具体请参考[elasticsearch内置分词器及字符过滤器]()
# es内置词元过滤器(Token Filter)
| Token Filter  | 简称| 描述 | 可支持参数|
| -|-|-| -|
|Standard  | standard  | standard 类型的词元过滤器规范化从 Standard Tokenizer（标准分词器）中提取的 tokens。什么也不做| 无|
|ASCII Folding | asciifolding | 将不存在于ASCII中的字符转为ASCII字符| preserve_original |
|Flatten Graph  | flatten_graph |接受任意图形 token 流，例如由 Synonym Graph Token Filter 产生的图形 token 流，并将其平坦化为适用于索引的单个线性标记链| 无| 
|Length  | length | 删除流中过长或过短的字词| min && max |
| Lowercase | lowercase | 小写过滤器| 无| 
|Uppercase | uppercase |大写过滤器| 无|
|NGram  | nGram | 无| min_gram && max_gram|
|Edge NGram  | edgeNGram |无| min_gram && max_gram|
|Porter Stem | porter_stem | 无 tip:porter_stem限制输入必须为小写，所以之前使用lowercase货值lowercase Tokenizer| 无|
|Shingle | shingles | 它创建词元的组合作为单个词元| max_shingle_size && min_shingle_size && output_unigrams && output_unigrams_if_no_shingles&& token_separator && filler_token|
|Stop  | stop | 停词过滤器| stopwords && stopwords_path && ignore_case&& remove_trailing|
|Word Delimiter | word_delimiter| 将单词分解为子词| 。。。|
|Word Delimiter Graph | word_delimiter_graph| | |
|Stemmer  | stemmer | 通过单个统一接口提供（几乎）所有可用的词干词元过滤器的访问 | name | 
|Stemmer Override | stemmer_override |通过应用自定义映射来覆盖词干算法，然后保护这些术语不被修改词干。 必须放置在任何词干过滤器之前| rules && rules_path|
|Keyword Marker|keyword_marker|保护词语不被词干分析器修改。 必须放置在任何词干过滤器之前|keywords&& keywords_path &&keywords_pattern ignore_case|
|Keyword Repeat| keyword_repeat | 索引两边token，一个为原始，一个为词根，所以需要添加unique过滤器，并且设置only_on_same_position  为true| 无| 
|KStem | kstem  | 用于英语的高性能过滤器 |无|
|Snowball| snowball | 使用Snowball-generated 分词器| language| 
|Phonetic | phonetic| analysis-phonetic 插件提供。| 无| 
|Synonym | synonym | 分析过程中轻松处理同义词。 同义词使用配置文件配置 | synonyms_path && synonyms ignore_case && expand  |
|Synonym Graph | synonym_graph | 实验版，| 无| 
|Compound Word | hyphenation\_decompounder \|dictionary_decompounder| word_list && word_list_path &&min_word_size &&only_longest_match | 无|
|Reverse | reverse | 将每个token反转| 无| 
|Elision | elision | [音节省略][音节省略]| articles 设置停用词|
|Truncate | truncate | 将token截取为固定长度| length 默认为10| 
|Unique | unique | 在索引分析时仅索引唯一的token,默认为所有词条|only_on_same_position 设为true,只删除同一位置的token|
|Pattern Capture | pattern_capture| | preserve\_original true返回原始token&& patterns|
|Pattern Replace| pattern_replace| 基于正则替换| 
|Limit Token | limit | 限制每篇文档和字段索引的token数| max_token\_count 默认为1 ;consume_all_tokens,如果为ture,超出限制也会最大程度处理token|
|Trim| trim | 去掉空白| 无|
|Hunspell Token| hunspell | 用法暂时不明 | |
|Common Grams|common_grams | 保留常用词语| common_grams; 常用词common_words_path;ignore_case;query_mode|
|Normalization | | | |
|CJK Width  | cjk_width| | |
|CJK Bigram | cjk_bigram| | |
|Delimited Payload|delimited_payload_filter| 每当发现分隔符时，将标记分成标记和有效载荷。|delimiter 分隔符;encoding 负载类型|
|Keep Words | keep | 只保留具有预定义单词集中的文本的token| keep_words保留列表;keep_words_path;keep_words_case 忽略大小写|
|Keep Types | keep_types| 只保留定义集合中的token| types|
|Apostrophe | apostrophe | 过滤撇号后面字符,包括撇号|无|
|Classic | classic| 对Classic Tokenizer生成词元进行处理,从token删除后面所有权,删除省略符标点|无|
|Decimal Digit | decimal_digit | 将unicode数字转化为0-9| 无|
|Fingerprint | fingerprint|  排序token,删除重复token,连接返回单个token| separator;max_output_size 如果连接的指纹增长大于max_output_size ，则过滤器将退出并且不会发出token|
|Minhash|min\_hash|将token流中每个token一一哈希，并将生成的哈希值分成buckets，以保持每bucket最低值的散列值。 然后将这些哈希值作为token(词元)返回。[minhash原理][hashmin]|hash_count;bucket_count;hash_set_size;with_rotation|


[音节省略]:https://zh.wikipedia.org/wiki/%E9%9F%B3%E7%AF%80%E7%9C%81%E7%95%A5
[hashmin]:[http://jm.taobao.org/2012/10/29/minhash-intro/]