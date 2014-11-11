## ElasticSearch的工作原理
<div>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;接下来简单了解一下ElasticSearch的工作原理。</div>

### 启动过程

<div>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;当ElasticSearch的节点启动后，它会利用多播(multicast)(或者单播，如果用户更改了配置)寻找集群中的其它节点，并与之建立连接。这个过程如下图所示</div>
<center><img src="../boostrap.png"/></center>
<div>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在集群中，一个节点被选举成主节点(master node)。这个节点负责管理集群的状态，当群集的拓扑结构改变时把索引分片分派到相应的节点上。 </div>
<br/>
<div style="height:162px;margin-left:20px;float:left;"><img src="../tipsL12.png"/></div>
<div style="height:162px;width:75%;float:left;word-wrap: break-word;word-break: normal; color:gray;font-family:COURIER;font-size:12px;background-color:#F7F7F7;padding-top:5px;">&nbsp;&nbsp;&nbsp;&nbsp;需要注意的是，从用户的角度来看，主节点在ElasticSearch中并没有占据着重要的地位，这与其它的系统(比如数据库系统)是不同的。实际上用户并不需要知道哪个节点是主节点；所有的操作需求可以分发到任意的节点，ElasticSearch内部会完成这些让用户感到不明觉历的工作。在必要的情况下，任何节点都可以并发地把查询子句分发到其它的节点，然后合并各个节点返回的查询结果。最后返回给用户一个完整的数据集。所有的这些工作都不需要经过主节点转发(节点之间通过P2P的方式通信)。</div>
<div style="height:162px;float:left;"><img src="../tipsR12.png"/></div>
<div style="clear:both;"></div>
<div>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;主节点会去读取集群状态信息；在必要的时候，会进行恢复工作。在这个阶段，主节点会去检查哪些分片可用，决定哪些分片作为主分片。处理完成后，集群就会转入到黄色状态。<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;这意味着集群已经可以处理搜索请求了，但是还没有火力全开(这主要是由于所有的主索引分片(primary shard)都已经分配好了，但是索引副本还没有)。接下来需要做的事情就是找到复制好的分片，并设置成索引副本。当一个分片的副本数量太少时，主节点会决定将缺少的分片放置到哪个节点中，并且依照主分片创建副本。所有工作完成后，集群就会变成绿色的状态(表示所有的主分片的索引副本都已经分配完成)。</div>

### 探测失效节点

<div>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在正常工作时，主节点会监控所有的节点，查看各个节点是否工作正常。如果在指定的时间里面，节点无法访问，该节点就被视为出故障了，接下来错误处理程序就会启动。集群需要重新均衡——由于该节点出现故障，分配到该节点的索引分片丢失。其它节点上相应的分片就会把工作接管过来。换句话说，对于每个丢失的主分片，新的主分片将从剩余的分片副本(Replica)中选举出来。重新安置新的分片和副本的这个过程可以通过配置来满足用户需求。更多相关信息可以参看<span style="font-style:oblique">&nbsp;第4章 分布式索引架构</span>。</div>

