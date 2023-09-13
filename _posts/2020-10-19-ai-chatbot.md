---
layout: post
title: 搭建智能问答机器人
categories: 人工智能 Q&A 智能问答 USE Milvus
excerpt: 如何将枯燥的Q&A列表改造成具备人工智能的问答机器人？
---

# 背景

场景一、大多数网站或者app都会有类似帮助中心的模块来帮助用户更好的使用网站或APP，目前大多数页面会以一问一答的形式展示。但是如果提供的QA或服务的内容较多，用户找起来会很麻烦，比如京东的帮助中心，提供的服务非常多，如果有一个简单的交互页面，用户输入口语化的问题就能正确定位到相关QA并直接给出答案，体验会更好。

场景二、购物类应用的客服通常要提供购物相关答疑，并且大多数问题会重复出现，如果有一个智能问答机器人在人工繁忙的时候预先提供服务，能很好的提高答疑效率。避免客户因长时间等待失去耐心和购物热情。

# 目标

构建一个智能问答机器人，能根据用户的提问从问答库中准确找出相应的答案并呈现。

这里我们不需要结构化语义，也不需要提供动态服务。

# 技术方案

这是一个文本相似度匹配的问题。

传统的工程技术解决方案是基于字符串匹配，首先对问题列表进行归一化处理，然后分词，利用lucene或elestic相关技术中间件将问题集存入倒排表进行索引存储。当用户输入提问时，对提问做归一化处理后利用中间件做分词或搜索字符串，中间件会自动计算匹配相似度并按匹配度从高向低给出匹配的问题及匹配度。满足匹配度阈值取第一条即可。

这种方式原问题和提问是基于对相同字符的出现频率和距离来匹配的。无法在语义上做匹配，泛化能力弱。举个例子：原问题：`一加一等于` ，用户如果提问`一加一`，大概率是能匹配到的，但是如果提问`1+1=？`，那大概率是匹配不到的。

论文 《Learning Semantic Textual Similarity from Conversations》 提供了一个语义相似度算法模型，她从对话数据中学习句子之间的语义相似度，通过无监督训练模型学习表征，并用多任务训练增强模型，并结合问答预测任务和自然语言推理任务提升性能。在给定的问答场景中，能有效区分语义相似的的问题。

Google提供了现成的universal-sentence-encoder模型（USE）：[https://tfhub.dev/google/universal-sentence-encoder-large/3](https://tfhub.dev/google/universal-sentence-encoder-large/3)

使用USE模型得到的是文本的向量表示，而要匹配提问和原问题的向量，则需要用到Milvus。Milvus是一个开源的分布式向量搜索引擎。我们把问答库中的原问题录入Milvus，在用户提示时，使用USE模型将提问转换成向量表示，并利用Milvus检索出相似度最高的原问题，可以找到对应的答案。

# 工程实现

源码：[https://github.com/hurongliang/ai-chatbot](https://github.com/hurongliang/ai-chatbot)

# Demo

搭建了一个简单应用，有兴趣的同学体验下。

地址：[https://chatbot.hurongliang.com/](https://chatbot.hurongliang.com/)

# 参考及引用 

[Learning Semantic Textual Similarity from Conversations](https://arxiv.org/abs/1804.07754)

[Advances in Semantic Textual Similarity](https://ai.googleblog.com/2018/05/advances-in-semantic-textual-similarity.html)


