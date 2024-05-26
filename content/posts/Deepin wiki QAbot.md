---
title: "Deepin wiki QAbot"
date: 2024-05-26T17:22:13+08:00
draft: true
authors: ["zhaoyangzhang"]
tags: ["文档问答系统", "大模型"]
---

# <center>Deepin wiki QAbot</center>

## 简介
我们开发了根据Deepin Wiki回答问题的聊天机器人，使其在文档集合中能够找到和问题最相关的答案呈现给用户。我们可以将该模型扩展至其他文档资源，如玲珑使用手册，同时提供了全自动化的部署流程。
机器人基于中文大语言模型，利用TF-IDF、 SimBert等技术从文章集合中筛选与用户题目最为相关的文章和段落，之后将这些段落作为上下文输入给模型，得到问题的答案。我们的系统展示了问题和答案良好的相关性，同时能够快速针对每个问题生成答案，具有良好的可用性。
本文详细介绍了项目内容、开发流程与思路，之后介绍相比前人的创新与核心贡献，以及对未来的展望。

## 题目介绍
Deepin Wiki是一个涵盖了900多条与Deepin系统相关的中文教程和词条的资源库。为了提高用户体验，我们提出了开发一个能够根据Deepin Wiki内容回答问题的聊天机器人。该机器人不仅需要理解用户的问题，还要能够在庞大的文档集合中找到最相关的答案，并以自然语言的形式呈现给用户。我们可以将该模型扩展至其他文档资源，如玲珑使用手册，提供全自动化的部署流程。

## 核心贡献

我们提出**二次筛选方法**，先使用**TF-IDF**模型筛选与问题最相关的文章，之后使用**SimBert**模型筛选文章中与问题最相关的段落，极大降低了时间延迟。与此同时，我们相比之前一些项目提高了用户使用的便捷性，如无需要求用户给出自己问题的领域方向，允许用户提出涉及多个文档集合中子领域的问题。同时，针对二次筛选方法我们提出了**预处理方法**，事先对文章和段落构建向量，无需占用用户的时间。

在部署模型的过程中，我们实现了文档从文件集合到存储为数据库文件过程的**全自动化**。在预处理过程中，我们实现了用文件存储simbert模型生成的词向量的自动化流程，还解决了simbert模型与chatglm模型所需transformer版本的不兼容问题，使得两者能够顺利结合。

我们的系统能够快速准确地理解用户的问题，并找到最相关的文档段落作为回答。同时，本项目不仅支持Deepin Wiki内容，还可扩展至其他文档资源，如玲珑使用手册，展现了系统**良好的可扩展性**。


## 开发思路

一、 数据预处理
为了训练模型，我们首先从https://github.com/linuxdeepin/wiki.deepin.org 中下载所有需要的文件夹。
之后我们设计脚本，将所有markdown文件合并为一个.json文件，以文章链接+标题+内容的形式存储所有文档。
接下来我们利用生成的.json文件构建数据库，用于未来的查询。

二、 系统构建思路

由于问答系统需要一个基础中文语言模型，我们选取了目前比较先进的中文语言模型ChatGLM3，用于最后的答案生成。

为了构建问答系统，首先需要设计整体的系统架构。为此，我们提出了一些初步的设想方法：

一是把所有的文档微调到LLM中，之后输入问题。但因为所有的文档一共有20MB，在有限的算力资源下，一般的模型并不能支持这么多的文本量输入，因此我们否决了这个方案。

二是利用TF-IDF模型筛选出与问题相关性最高的文章，将这些文章作为上下文输入模型，但实践中发现，这样的方法导致上下文长度过大，模型的反应时间较长，因此需要进一步改进。

最后，我们创新性地提出了二次筛选方法，先使用TF-IDF模型筛选与问题最相关的文章，之后使用SimBert模型筛选文章中与问题最相关的段落，最后将筛选出的段落作为上下文与问题一起输给大语言模型，得到问题的答案，取得了良好的效果。

## 三、系统介绍

![alt text](image.png)

<p>

### TF-IDF模型 

TF-IDF模型被用来评估问题与各个文章的相似度。
TF-IDF（Term Frequency-Inverse Document Frequency）是一种用于信息检索与文本挖掘的常用加权技术。这种方法用于评估一个字词对于一个文件集或一个语料库中的其中一份文件的重要程度。字词的重要性随着它在文件中出现的次数成正比增加，但同时会随着它在语料库中出现的频率成反比下降。TF-IDF方法是一种统计方法，用以评估字词对于一个文件集中的其中一份文件的重要性。TF 表示词条（关键字）在文档中出现的频率。这个数字是对词条权重评估的一种直观感受。但是，由于每个文档的长度不同，它可能会对词条的重要性产生影响。因此，词频（TF）通常会被归一化（一般是词条在文档中的出现次数除以文档中的总词条数目）。IDF 的主要思想是如果包含词条 T 的文档越少，也就是说 T 越能够代表这个文档。逆文档频率是一个词条重要性增长的度量。计算某一特定词条的 IDF，可以将语料库中的文档总数除以包含该词条之文档的数目，然后将得到的商取对数得到。

通过将题目和每篇文章转换成TF-IDF权重向量，计算题目向量与每篇文章向量之间的余弦相似度，可以选择与问题最相关的文章。

###  SimBert模型

SimBert是由苏剑林开发的模型，以Google开源的BERT模型为基础，基于微软的UniLM思想设计了融检索与生成于一体的任务，来进一步微调后得到的模型，它同时具备相似问生成和相似句检索的能力。

我们利用Simbert模型，找出之前筛选出的文章中与问题最为相关的n个段落。为了提高效率，我们为SimBert过程进行预处理，将每篇文章的各个段落映射成向量，保存到本地文件中。之后当我们向问答系统输入问题流时，我们可以直接对前一步找到的文章的段落向量与问题进行相似度计算，找出这些文章各自最相关的段落。 

最后，我们将这些段落一起作为上下文，结合原来的问题输入给预训练中文语言模型，得到问题的答案。

###  参考链接的处理

当输入上下文较长的时候，大模型可能会对其中的链接产生遗忘，为此，我们直接将TF-IDF筛选出的文章的链接输给用户，不再利用模型生成，这样同时也保证了结果的准确性。

### 测试集的构建

我们使用脚本将markdown文件集合转换成txt文件，并对txt文件进行处理，最终调用gpt的API即可生成问答对测试集。

</p>

## 结果展示
![alt text](2942b488392d00cf2887071a4bdaba4c.png)
![alt text](2942b488392d00cf2887071a4bdaba4c.png)
通过采用ChatGLM3中文语言模型和预筛选技术，我们的机器人能够有效处理用户的问题，并从Deepin Wiki中找到最相关的内容作为回答。 

### 参考文献
https://arxiv.org/pdf/1704.00051

https://github.com/ZhuiyiTechnology/simbert