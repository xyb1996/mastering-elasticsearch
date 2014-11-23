  ## 改变分片的默认分配方式

<div style="text-indent:2em">
<p>在前面的章节中，我们学习了很多关于分片的知识以及与之相关的特性。我们也讨论了shard allocation的工作方式(本章的<i>调整集群的分片分配</i>一节)。然而除了默认的分配方式，我们并没有探讨其它的内容。ElasticSearch提供了更多的分片分配策略来构建先进的系统。在本节，我们将更深入地了解在分片分配方面，我们还能做哪些事情。</p>
<h3>ShardAllocator介绍</h3>
<p>ShardAllocator是决定分片安置到哪个节点起主要作用的一个。 当ElasticSearch改变数据在节点上的分配时，比如集群拓扑结构的改变(当节点添加或者移出集群)或者用户强制集群进行再平衡操作，ElasticSearch中分片就需要重新分配。在ElasticSearch内部，分配器继承org.elasticsearch.cluster.routing.allocation.allocator.ShardsAllocator接口。ElasticSearch提供了两种类型的分配器，它们分别是：
<ul>
<li>even\_shard</li>
<li>balanced(默认实现)</li>
</ul>
我们可以通过在elasticsearch.yml文件中或者settings API设置cluster.routing.allocation.type属性来指定一种分配器的接口实现方式。
</p>
<h4>even_shard 分片分配器</h4>
</div>
