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

<h4>index-level过滤器缓存的配置</h4>
<p>ElasticSearch允许用户使用如下的属性来配置index-level过滤器缓存的行为：
<ul>
<li>index.cache.filter.type:该属性用于指定缓存的类型，有resident,soft和weak，node(默认值)4个值可供选择。对于resident类型的缓存，JVM无法删除其中的缓存项，除非用户来删除(通过API,设置缓存的最大容量及失效时间都可以对缓存项进行删除)。推荐使用该缓存类型(因为填充过滤器缓存开销很大)。soft和weak类型的缓存能够在内存不足时，由JVM自动清除。当JVM清理缓存时，其操作会根据缓存类型而有所不同。它会首先清理引用比较弱的缓存项，然后才会是使用软引用的缓存项。node属性表示缓存将在节点层面进行控制(参考本章的<i>Node-level过滤器缓存配置</i>一节的内容)</li>
<li>index.cache.filter.max\_size: 该属性指定了缓存可以存储缓存项的数量(默认值是-1，表示数量没有限制)。读者需要记住，该设置项不适用于整个索引，只适用于索引分片上的一个段。所以缓存的内存使用量会因索引中分片(以及分片副本)的数量，还有索引中段的数量的不同而不同。通常情况下，默认没有容量限制的缓存适用于soft类型和致力于缓存重用的特定查询类型。  </li>
<li>index.cache.filter.expire:该属性指定了过滤器缓存中缓存项的失效时间，默认是永久不失效(所以其值设为-1)。如果希望一段时间类没有命中的缓存项失效，缓存项沉寂的最大时间。比如，如果希望缓存项在60分钟类没有命中就失效，设置该属性值为60m即可。 </li>
</ul>
</p>
<!--note structure -->
<div style="height:80px;width:650px;text-indent:0em;">
<div style="float:left;width:13px;height:100%; background:black;">
  <img src="../lm.png" height="70px" width="13px" style="margin-top:5px;"/>
</div>
<div style="float:left;width:50px;height:100%;position:relative;">
	<img src="../note.png" style="position:absolute; top:30%; "/>
</div>
<div style="float:left; width:550px;height:100%;">
	<p style="font-size:13px;margin-top:5px;">想了解更多关于软引用和虚引用相关的内容，可以参考Java Document，特别是以下两类：http://docs.oracle.com/javase/7/docs/api/java/lang/ref/SoftReference.html 和 http://docs.oracle.com/javase/7/docs/api/java/lang/ref/WeakReference.html. </p>
</div>
<div style="float:left;width:13px;height:100%;background:black;">
  <img src="../rm.png" height="70px" width="13px" style="margin-top:5px;"/>
</div>
</div> <!-- end of note structure -->

<h4>Node-level过滤器缓存的配置</h4>
<p>让缓存作用于节点上，是ElasticSearch默认和推荐的设置。对于特定节点的所有分片，该设置都已经默认生效(设置index.chache.filter.type属性值为node,或者对此不作任何设置)。ElsticSearch允许用户通过indices.cache.filter.size属性来配置缓存的大小。用户可以使用百分数，比如20%(默认设置)或者具体的值，比如1024mb来指定缓存的大小。如果使用百分数，那么ElasticSearch会基于节点的heap 内存值来计算出实际的大小。 </p>
<p>node-level过滤器缓存是LRU类型的缓存(最近最少使用)，即需要移除缓存项来为新的缓存项腾出位置时，最长时间没有命中的缓存项将被移除。</p>

<h4>域数据缓存</h4>
<p>当查询命令中用到faceting功能或者指定域排序功能时，域数据缓存就会用到。使用该缓存时，ElasticSearch所做的就是将指定域的所有取值加载到内存中，通过这一步，ElasticSearch就可以提供文档域快速取值的功能。有两点需要记住：直接从硬盘上读取时，域的取值开销很大，这是因为加载整个域的数据到内存不仅需要I/O操作，还需要CPU资源。 </p>
<!--note structure -->
<div style="height:80px;width:650px;text-indent:0em;">
<div style="float:left;width:13px;height:100%; background:black;">
  <img src="../lm.png" height="70px" width="13px" style="margin-top:5px;"/>
</div>
<div style="float:left;width:50px;height:100%;position:relative;">
	<img src="../note.png" style="position:absolute; top:30%; "/>
</div>
<div style="float:left; width:550px;height:100%;">
	<p style="font-size:13px;margin-top:5px;">读者需要记住，对于每个用来进行faceting操作或者排序操作的域，域的所有取值都要加载到内存中：一个Term都不能放过。这个过程开销很大，特别是对于基数比较大的域，这种域的term对象数目巨大。</p>
</div>
<div style="float:left;width:13px;height:100%;background:black;">
  <img src="../rm.png" height="70px" width="13px" style="margin-top:5px;"/>
</div>
</div> <!-- end of note structure -->
<h4>index-level过滤器缓存的配置</h4>
<p></p>

<h4>node-level过滤器缓存的配置</h4>
<p></p>
<h4>缓存的清空</h4>
</div>
