---
title: Elasticsearch内置过滤器
description:  Elasticsearch过滤器
date: 2018-03-09 20:40:00
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
|Stemmer Override | stemmer_override |通过应用自定义映射来覆盖词干算法，然后保护这些术语不被词干修改。 必须放置在任何词干过滤器之前| rules && rules_path|
|Keyword Marker|keyword_marker|保护词语不被词干分析器修改。 必须放置在任何词干过滤器之前|keywords&& keywords_path &&keywords_pattern ignore_case|
|Keyword Repeat| keyword_repeat | 索引两边token，一个为原始，一个为词根，所以需要添加unique过滤器，并且设置only_on_same_position  为true| 无| 
|KStem | kstem  | 用于英语的高性能过滤器 |无|
|Snowball| snowball | 使用Snowball-generated 分词器| language| 
|Phonetic | phonetic| analysis-phonetic 插件提供。| 无| 
|Synonym | synonym | 分析过程中轻松处理同义词。 同义词使用配置文件配置 | synonyms_path && synonyms ignore_case && expand  |
|Synonym Graph | synonym_graph | 实验版，| 无| 
|Compound Word | hyphenation\_decompounder \|dictionary_decompounder| word_list && word_list_path &&min_word_size &&only_longest_match 