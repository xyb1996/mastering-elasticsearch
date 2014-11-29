## 理解ElasticSearch缓存
<div style="text-indent:2em;">
<p>缓存对于已经配置好且正常工作的集群来说不过过多关注(这一条不仅适用于ElasticSearch)。缓存在ElasticSearch中扮演着重要的角色。通过缓存用户可以高效地存储过滤器的数据并且重复使用这些数据，比如高效地处理父子关系数据、faceting、对数据以索引中某个域来排序等等。在本节中，我们将详细研究filter cache和field data cache这些最重要的缓存，而且我们将会意识到理解缓存的工作原理对于集群的调优非常重要。 </p>
<h4>过滤器缓存</h4>
<p>过滤器缓存是负责缓存查询语句中过滤器的结果数据。比如，让我们来看如下的查询语句：
<blockquote style="text-indent:2em;">
{
"query" : {
"filtered" : {
"query" : {
"match_all" : {}
},
"filter" : {
"term" : {
"category" : "romance"
}
}
}
}
}
</blockquote>
执行该查询语句将返回所有category域中含有term值为romcance的文档。正如读者如看到的那样，我们将match_all查询类型与过滤器结合使用。现在，执行一次查询语句后，每个有同样过滤条件的查询语句都会重复使用缓存中的数据，从而节约了宝贵的I/O和CPU资源。
</p>
<h4>过滤器缓存的类型</h4>
<p>在ElasticSearch中过滤器缓存有两种类型：索引级别和节点层面级别的缓存。所以基本上我们自己就可以配置过滤器缓存依赖于某个索引或者一个节点(节点是默认设置)。由于我们无法时时刻刻来猜测具体的某个索引会分配到哪个地方(实际上分配的是分片和分片副本)，也就无法预测内存的使用，因此不建议使用基于索引的过滤器。 </p>

<h4>索引级别的过滤器缓存的配置</h4>
<p>ElasticSearch允许用户使用如下的属性来配置索引级别的过滤器缓存的行为：
<ul>
<li>index.cache.filter.type: </li>
<li>index.cache.filter.max_size: </li>
<li>index.cache.filter.expire: </li>
</ul>
</p>

<h4>节点级别的过滤器缓存的配置</h4>
<p></p>

<h4>域数据缓存</h4>
<p></p>

</div>
