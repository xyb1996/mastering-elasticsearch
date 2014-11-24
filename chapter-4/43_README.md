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
<p>早在0.90.0版本之前，ElasticSearch就已经支持该分配器。它唯一的作用就是确保每个节点中分片数量都相同(当然，这一点并不总是能得到保证的)。它也不允许把主分片和它的副本存储在同一个节点上。当需要重新调整分片分布时，如果使用even_shard,只要集群没有达到完全平衡的状态或者分片间已经无法再移动了，ElasticSearch就会把一些分片从分片密集节点移动到分片稀疏节点。需要注意的是，这种分配器的应用层面并非索引层面，这意味着只要分片和它的副本分布在不同的节点上即可，它并不关心来自同一个索引的不同分片分配到哪个节点。</p>
<h4>balanced 分片分配器</h4>
<p>该分配器是ElasticSearch 0.90.0版本新引入的，它是基于服务器的重要程度来分配分片,这个重要程度是用户可以掌控的。与前面提到的even_shard分配器相比，它通过暴露一些参数接口给用户来实现分片分配过程的调整，这些参数可以用集群的update API实时更新，它们是：
<ul>
<li>cluster.routing.allocation.balance.shard:默认值 0.45</li>
<li>cluster.routing.allocation.balance.index:默认值 0.5</li>
<li>cluster.routing.allocation.balance.primary:默认值 0.05</li>
<li>cluster.routing.allocation.balance.threshold:默认值 1.0</li>
</ul>
上述的参数决定了balanced分配器的行为。从第一个开始一一介绍，首先是基于分片数量的权重因子；其次是基于某个索引所有分片的权重因子；最后是基于主分片的权重因子。我们暂时将threshold参数放在一边，不作解说。对于某个特定的因子，其权重值越大，表明它越重要，对ElasticSearch在分片重新分配的决策上产生和影响也越大。
</p>
<p>第一个因子用来告诉ElasticSearch每个节点上分配数量相近的分片在用户心中的重要程度。第二个因子的作用差不多，只是不针对整个集群所有的分片，针对的是整个索引的所有分片。第三个因用来告诉ElasticSearch每个节点上分配数量相同的主分片在用户心中的重要程度。所以，如果你的集群中，保证每个节点拥有相同数量的主分片这一原则非常重要，那么就应该提升cluster.routing.allocation.balance.primary的权重因子值，同时降低除threshold外其它因子的权重值。</p>
<p>最后，如果所有因子乘以权重后的和如果比设置的threhold值要大，那么该索引的分片就需要进行重新分配了。如果由于某个原因，你希望一些因子不影响分片的分配，那么设置其值为0即可。</p>
<h4>自定义ShardAllocator</h4>
<p>有些情况下，系统内置的分配器可能不适用于用户的系统部署方案。比如，在在分配分片的过程中要考虑到索引的大小；又比如，一个由不同硬件组件，处理能力不同的CPU，数量不同的内存，容量不同的硬盘组成的超大集群。所有的这些因素都可能导致分布式系统分发数据到各个节点时效率不高。
令人高兴的是，在ElasticSearch中能够实现自己的解决方案，只需要编写自己的Java类，实现org.elaticsearch.cluster.routing.allocation.allocator.ShardsAllocator接口，然后把全限定类名作为cluster.routing.allocation.type属性的参数值，配置到集群中即可。
</p>
<h4>决策者(Deciders)</h4>
<p>为了了解分片分配器是如何决定什么时候分片会被移动，又该移向哪个节点，我们需要探究ElaticSearch的内部实现方式，这个内部实现方式称为决策者(deciders)。它们就像是人的大脑一样制定分配决策。ElasticSearch允许用户同时使用多个决策者，所有的决策者在决策时进行投票。它们遵循一致性原则，比如，如果一个决策者反对重新分配一个分片，那么这个分片就不会被移动。如下的决策者是ElasticSearch内置的，它们的决策方式一成不变，除非修改源代码。让我们来看看哪些决策者是默认的</p>
<h4>SameShardAllocationDecider</h4>
<p>正如它的名字一样，这个决策者不允许数据及副本(主分片和分片副本)安置到同一个节点。这样做的原因很明显：我们不希望备份数据和源数据放在同一个地方。说到这个决策者，我们就不得不提cluster.routing.allocation.same\_shard.host属性。它控制着ElasticSearch是否该留意分片所在的物理机。其默认值为false，因为同一个服务器上可以运行多个节点，可以是通过多虚拟机的方式。当设置它的值为true时，这个决策者就不允许把分片和它的分片副本分配到同一个物理机上。这看起来可能有点奇怪，但是考虑到现在各种虚拟化技术大行其道，操作系统都不清楚它运行在哪台物理机。正因为如此，最好</p>
</div>
