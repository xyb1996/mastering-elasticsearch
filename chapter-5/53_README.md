## 索引段数据的统计
<div style="text-indent:2em;">
<p>在<i>第3章 索引底层控制</i>中 <i>控制段合并</i> 一节中，我们探讨了调整Apache Lucene索引段合并过程来满足业务需求的可能性。此外，在<i>第6章 应对突发事件</i>的<i>当I/O过于繁忙——节流功能详解</i>一节中，我们将讨论更多功能的参数配置。然而，为了了解需要调整哪些方面，至少先得看看索引或者索引分片中索引段的结构。 </p>

<h4>段操作相关的API介绍</h4>
<p>为了更深入地了解Lucene的索引段，ElasticSearch提供了段操作相关的API，通过在执行的HTTP GET请求中附带\_segments REST端点，就可以调用相关的API了。比如，我们想了解集群分片中的所有的段信息，可以执行如下的命令：
<blockquote>curl -XGET 'localhost:9200/_segments'</blockquote>
如果想查看mastering索引的段信息，就应该执行如下的命令：
<blockquote>curl -XGET 'localhost:9200/mastering/_segments'</blockquote>
我们也可以同时查看多个索引的段信息，通过如下的命令即可：
<blockquote>curl -XGET 'localhost:9200/mastering,books/_segments'</blockquote>
</p>
<h4>返回信息</h4>
<p>调用segments API的返回信息都是基于分片的。这是因为我们的索引都是由一个或多个分片(以及分片副本)组成，而且众所周知每个分片都是一个完整的Apache Lucene的物理索引。我们假定我们有一个名为mastering的索引，而且索引中有一些文档。在索引创建时，我们已经指定索引由单个分片构成，而且没有分片副本：由于只是用于测试的索引，这样做没有什么问题。我们来看一下，执行如下的命令后，索引段信息会是什么样：
<blockquote>curl -XGET 'localhost:9200/_segments?pretty'</blockquote>
返回的JSON格式信息如下(有部分删减)：
<blockquote>
{
"ok" : true,
"_shards" : {
"total" : 1,
"successful" : 1,
"failed" : 0
},
"indices" : {
"mastring" : {
"shards" : {
"0" : [ {
"routing" : {
"state" : "STARTED",
"primary" : true,
"node" : "Cz4RFYP5RnudkXzSwe-WGw"
},
"num\_committed\_segments" : 1,
"num\_search\_segments" : 8,
"segments" : {
"_0" : {
"generation" : 0,
"num_docs" : 62,
"deleted_docs" : 0,
"size" : "5.7kb",
"size\_in\_bytes" : 5842,
"committed" : true,
"search" : true,
"version" : "4.3",
"compound" : false
},
...
"_7" : {
"generation" : 7,
"num_docs" : 1,
"deleted_docs" : 0,
"size" : "1.4kb",
"size\_in_bytes" : 1482,
"committed" : false,
"search" : true,
"version" : "4.3",
"compound" : false
}
}
} ]
}
}
}
}
</blockquote>
我们可以看到，ElasticSearch返回了大量可供分析的信息。最顶层的息是索引名称和和分片。在本例中，通过返回信息可以看到，我们有一个编号为0的分片，该分片已经启动而且正在运行("state" : "STARTED" ) ，该分片是一个主分片 ("primary" : true) ,该分片位于id为 Cz4RFYP5RnudkXzSwe-WGw的节点上。接下来的信息是关于已提交段的数量(通过num\_commited\_segments属性)和搜索段的数量(num\_search\_segments属性)。已提交段表示该段
</p>

</div>
