# 第1章 认识Elasticsearch


<div>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;希望读者通过阅读本书，能够扩展和巩固ElasticSearch的基础知识。假定读者已经知道用单个请求(curl)和批量索引向ElasticSearch导入数据；也知道如何发送请求获取目标文档；也知道如何通过filter过滤查询结果。使结果更精确；也知道如何使用facet/aggregation机制来对结果进行统计处理，在学习ElasticSearch那些激动人心的功能之前，还是需要快速了解一下Apache Lucene。Apache Lucene，一种全文检索工具。ElasticSearch就是构建在Lucene之上的。与此同时，ElasticSearch也沿袭了Lucene的基本概念。如果想更快地理解ElasticSearch,就必须牢记Lucene的基本概念。当然，记住概念是很简单的。 但是如果想掌握Elasticsearch,在记住Lucene概念概念的基础之上，还必须理解这些概念。在本章，我们将学到如下的知识。</div>
<hr>
* Apache Lucene的简单介绍
* Lucene的总体架构
* 文本解析(analysis)的过程
* ElasticSearch的基本概念
* ElasticSearch的内部通信机制






