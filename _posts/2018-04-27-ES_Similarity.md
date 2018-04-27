---
layout: post
title:  "论ElasticSearch相关性打分机制"
date:   2018-04-27 23:13:00 +0800
categories: ElasticSearch ES
---

### 总

先按层次结构总结下ES的相关新打分机制，如若相关概念不明白可以看完详请部分后，再回顾此部分。

ES的相关性打分机制主要分为以下四部分：
1. 文本相关性（_score 分数计算）
2. 相关性评分查询：
	1. boosting 查询：加减查询权重
    2. constant_score 查询：启禁文本相关性因子 
    3. function_score 查询：允许为每个与主查询匹配的文档应用一个函数，以达到改变甚至完全替换原始查询评分 _score 的目的。
3. 脚本评分：结合query中param参数与从文档中提取字段值计算最后得分
4. 打分插件：更改_score的评分过程，可自定义打分规则
