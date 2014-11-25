## 查询命令的execution preference

<div style="text-indent:2em;">
    <p>暂时先忘记分片分配以及分配方式，ElasticSearch不仅提供了关于分片和分片副本各式各样的设置方式，同时也提供了指定查询命令(还有其它的操作，比如real time get)执行位置的功能。在详细了解该功能之前，先看看样例集群：</p>
    <center><img src="../44_cluster.png" /></center>
    <p>通过上图可以看到，样例集群有3个节点和一个名称为<b>Mastering</b>的索引。索引拆分成两个主分片，每个分片都有一个分片副本。</p>

<h4>preference参数简介</h4>
<p>为控制客户端发送的查询(及其它操作)命令在集群中执行的位置，我们用到了preference参数，该参数可指定如下的值：
<ul>
<li>\_primary: 使用该属性值，发送到集群的相关操作请求只会在主分片上执行。如果发送查询命令到mastering索引时附带了值为\_primary的preference参数，该命令将只在名字为node1和node2的节点上执行。比如，如果用户集群的主分片在一个机架中，分片副本在另一个机架中，用户就可能希望命令只在主分片中执行以避免使用网络流量。 </li>
<li>\_primary\_first:该属性值与\_primary属性值导致相似的集群行为，但是具有容错机制。如果发送查询命令到mastering索引时附带了值为\_primary\_first的preference参数，该命令将在名称为node1和node2的节点上执行，但是如果有一个(或者更多)的主分片失效，查询命令将转到其它的分片上执行，在本例中会转到node3上执行。正如我们所说，该属性值与\_primary属性值相似，但是如果由于某些原因主分片失效了，那么命令就会回转到分片副本上执行。</li>
<li>\_local: ElasticSearch会优先在本地的节点上执行相关操作。比如，如果我们向node3发送附带一条preference参数值为\_local的查询命令，最终该查询命令会在node3上执行。但是，如果我们把相同的命令发送到node2节点，那么最终该命令不仅会在编号为1的分片(分片位于本地节点)上执行，同时也会分发到node1或者node3上执行，这两个节点上有编号为0的分片。该属性值在减小网络传输时间上特别有用。只要用到了\_local preference参数值，我们就能确保查询命令会尽可能地在本地的节点上执行。(比如，从本地节点运行一个客户端连接或者发送一个查询命令到节点上)</li>
<li>\_only\_node:wJq0kPSHTHCovjuCsVK0-A: </li>
<li>\_prefer\_node:</li>
<li>\_shards:</li>
<li>custom</li>
</ul>
最后还有一点没有提到，就是ElasticSearch的默认操作。默认情况下，ElasticSearch会将相关操作随机分发到分片或者分片副本上。如果往集群中发送大量的查询命令，最终每个分片和分片副本上执行的查询命令数量会大致相同。
</p>
</div>
