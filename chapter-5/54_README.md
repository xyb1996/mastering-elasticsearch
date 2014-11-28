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

<h4>index-level的过滤器缓存的配置</h4>
<p>ElasticSearch允许用户使用如下的属性来配置索引级别的过滤器缓存的行为：
<ul>
<li>index.cache.filter.type:该属性用来设置缓存的类型，以下选项可用：resident,soft,weak,node(默认值)。在resident缓存中存储的数据项是JVM无法删除的，除非用户用某种手段删除(用API手动删除或者设置缓存的最大容量或者有效日期) ，由于缓存数据不易丢失，而填充缓存的开销又比较大，ElasticSearch推荐使用该缓存。soft和weak过滤器缓存类型很容易在内存吃紧的时候被JVM清空。由于引用类型的不同，JVM在清除缓存时，会首先选择比较弱的引用类型对象，然后才选择soft引用(即软引用)。node属性表明缓存的控制级别是节点级别(参考本章的 <i>节点级别过滤器缓存配置</i> 一节的内容)。 </li>
<li>index.cache.filter.max_size:该属性指定了能够存储到缓存的数据项的最大数目(默认值为-1，表示对数量没有限制)。读者需要记住，该设置项不适合整个索引，而适合索引分片中的一个索引段。使用该属性，内存的使用量会根据不同的情况而有所不同，主要取决于具体索引中分片和分片副本的数量，以及该索引包含的索引段的数量。通常情况下，默认数值，即大小无限制适用于soft类型和能够让缓存得到充分复用的查询语句。 </li>
<li>index.cache.filter.expire:该属性指定了过滤器缓存中数据项的有效期，默认值为-1，代表永久有效。如果希望一段时间内缓存项没有命中就失效，用户可以设置一个静态的最大时间点。比如，如果希望缓存在60分钟内没有命中就失效，设置该属性值为60m即可. </li>
</ul>
</p>

<!--note structure -->
<div style="height:50px;width:650px;text-indent:0em;">
<div style="float:left;width:13px;height:100%; background:black;">
  <img src="../lm.png" height="40px" width="13px" style="margin-top:5px;"/>
</div>
<div style="float:left;width:50px;height:100%;position:relative;">
	<img src="../note.png" style="position:absolute; top:30%; "/>
</div>
<div style="float:left; width:550px;height:100%;">
	<p style="font-size:13px;margin-top:5px;">如果想了解Java语言中soft和weak引用类型更多的相关知识，请参考Java文档，特别是如下两种类型相关的文档：http://docs.oracle.com/javase/7/docs/api/java/lang/ref/SoftReference.html 和 http://docs.oracle.com/javase/7/docs/api/java/lang/ref/WeakReference.html.</p>
</div>
<div style="float:left;width:13px;height:100%;background:black;">
  <img src="../rm.png" height="40px" width="13px" style="margin-top:5px;"/>
</div>
</div> <!-- end of note structure -->

<h4>node-level过滤器缓存的配置</h4>
<p>默认和推荐的过滤器缓存类型作用于指定结点的所有分片(使用index.cache.filter.type属性值为node或者根本不设置该属性)。ElasticSearch允许用户使用indices.cache.filter.size属性来用配置缓存的大小，该属性值可以使用百分数，比如20%(默认值)，或者静态的内存值，比如1024mb。如果我们使用百分数，ElasticSearch会根据节点的最大heap内存来计算出它的实际大小。 </p>
<p>node-level过滤器缓存是一种LRU(Least Recently Used)类型的缓存，这意味着如果要删除缓存项，那么最长时间没有被命中到的缓存项将被丢弃，从而给新的缓存项腾出位置。</p>
<h4>field数据缓存</h4>
<p>Field Data缓存在查询语句中包含faceting功能或者基于Field值排序的情况下会用到。ElasticSearch所做的就是将查询语句用到的Field的所有取值到加载内存中。通过这一步，ElasticSearch就有能力提供文档快速取值功能。有两点需要我们记住：在硬件资源层面，field数据缓存的数据加载的开销通常比较大，因为整个field的所有取值都需要加载到内存中，而且这个加载过程需要I/O操作和CPU资源。 </p>
<!--note structure -->
<div style="height:50px;width:650px;text-indent:0em;">
<div style="float:left;width:13px;height:100%; background:black;">
  <img src="../lm.png" height="40px" width="13px" style="margin-top:5px;"/>
</div>
<div style="float:left;width:50px;height:100%;position:relative;">
	<img src="../note.png" style="position:absolute; top:30%; "/>
</div>
<div style="float:left; width:550px;height:100%;">
	<p style="font-size:13px;margin-top:5px;">我们需要记住的是：对于每个Field，无论是对其排序还是Faceting操作，所有的取值都会加载到内存中：每个Field的所有term。这个过程的开销就很大了，特别是基数比较高的Field，term值的取值千差万别。</p>
</div>
<div style="float:left;width:13px;height:100%;background:black;">
  <img src="../rm.png" height="40px" width="13px" style="margin-top:5px;"/>
</div>
</div> <!-- end of note structure -->
</div>
