## ElasticSearch背后的核心理念
<div>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ElasticSearch是构建在极少数的几个概念之上的。ElasticSearch的开发团队希望它能够快速上手，可扩展性强。而且这些核心特性体现在ElasticSearch的各个方面。从架构的角度来看，这些主要特性是：
<ul>
<li>开箱即用。安装好ElasticSearch后，所有参数的默认值都自动进行了比较合理的设置，基本不需要额外的调整。包括内置的发现机制(比如Field类型的自动匹配)和自动化参数配置。</li>
<li>天生集群。ElasticSearch默认工作在集群模式下。节点都将视为集群的一部分，而且在启动的过程中自动连接到集群中。</li>
<li>自动容错。ElasticSearch通过P2P网络进行通信，这种工作方式消除了单点故障。节点自动连接到集群中的其它机器，自动进行数据交换及以节点之间相互监控。索引分片</li>
<li>扩展性强。无论是处理能力和数据容量上都可以通过一种简单的方式实现扩展，即增添新的节点。</li>
<li>近实时搜索和版本控制。由于ElasticSearch天生支持分布式，所以延迟和不同节点上数据的短暂性不一致无可避免。ElasticSearch通过版本控制(versioning)的机制尽量减少问题的出现。</li>
</ul>
</div>

