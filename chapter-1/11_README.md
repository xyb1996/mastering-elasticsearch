# 认识Apache Lucene

<div>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 为了更深入地理解ElasticSearch的工作原理，特别是索引和查询这两个过程，理解Lucene的工作原理至关重要。本质上，ElasticSearch是用Lucene来实现索引的查询功能的。如果读者没有用过Lucene，下面的几个部分将为您介绍Lucene的基本概念。
</div>

## 熟悉Lucene

<div>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;读者也许会产生疑问，为什么ElasticSearch 的创造者决定用Lucene而不是自己开发相应的组件。我们也不知道为什么，因为我们不是决策者。但是我们可以猜想可能是因为Lucene是一个成熟的、高性能的、可扩展的、轻量级的，而且功能强大的搜索引擎包。Lucene的核心jar包只有一个文件，而且不依赖任何第三方jar包。更重要的是，它提供的索引数据和检索数据的功能开箱即用。当然，Lucene也提供了多语言支持，具有拼写检查、高亮等功能；但是如果你不需要这些功能，你只需要下载Lucene的核心jar包，应用到你的项目中就可以了。
</div>







