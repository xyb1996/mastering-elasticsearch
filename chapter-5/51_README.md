## 选择正确的directory实现类——存储模块
<div style="text-indent:2em;">
<p>在配置集群时，数据存储模块是众多模块中不需要过多关注的一个模块，但是却是非常重要的一个模块。它允许用户控制索引数据的存储方式：持久化存储(在硬盘上)或者临时存储(在内存中)。ElasticSearch中绝大部分的存储方式都会映射到合适的Apache Lucene Directory类(http://lucene.apache.org/core/4_5_0/core/org/apache/lucene/store/Directory.html )。directory模块用来存取所有索引文件，因此合理配置就显得变尤为重要了。 </p>
<h4>存储类型</h4>
<p>ElasticSearch给用户提供了4种存储类型，下面看看它们提供了什么特性以及用户该如何使用用这些特性。</p>
<h4>简单的文件系统存储</h4>
<p>directory类对外最简单的实现基于文件的随机读写(Java RandomAccessFile: http://docs.oracle.com/javase/7/docs/api/java/io/RandomAccessFile.html )映射到Apache Lucene的SimpleFSDirectory(http://lucene.apache.org/core/4\_5\_0/core/org/apache/lucene/store/SimpleFSDirectory.html )。对于简单的应用来说，这种实现方式足够了。它主要的瓶颈是在文件的多线程存取时性能很差。在ElasticSearch中，通常建议使用基于新IO的系统存储来替代简单的文件系统存储。只是如果用户希望使用简单的文件系统存储，可以设置index.store.type属性值为simplefs。</p>
<h4>新IO文件系统存储</h4>
<p>这种存储类型使用的directory类是基于java.nio包中的FileChannel类(http://docs.oracle.com/javase/7/docs/api/java/nio/channels/FileChannel.html )实现的，该类映射到Apache Lucene的NIOFSDirectory类(http://lucene.apache.org/core/4\_5\_0/core/org/apache/lucene/store/NIOFSDirectory.html ). 这种实现方式使得多个线程同时读写文件时不会出现性能下降的问题。通过设置index.store.type属性值为niofs使用该存储类型。</p>
<!--note structure -->
<div style="height:50px;width:650px;text-indent:0em;">
<div style="float:left;width:13px;height:100%; background:black;">
  <img src="../lm.png" height="40px" width="13px" style="margin-top:5px;"/>
</div>
<div style="float:left;width:50px;height:100%;position:relative;">
	<img src="../note.png" style="position:absolute; top:30%; "/>
</div>
<div style="float:left; width:550px;height:100%;">
	<p style="font-size:13px;margin-top:5px;">需要记住的是由于Microsoft Windows操作上的JVM虚拟机有一些bug，新IO文件系统存储代码运行在Windows上时很有可能会出现一些性能问题。bug相关的信息可以参考(http://bugs.sun.com/bugdatabase/view\_bug.do?bug\_id=6265734. )</p>
</div>
<div style="float:left;width:13px;height:100%;background:black;">
  <img src="../rm.png" height="40px" width="13px" style="margin-top:5px;"/>
</div>
</div> <!-- end of note structure -->

<h4>MMap文件系统存储</h4>
<p>这种存储类型使用Apache Lucene MMapDirectory实现类(http://lucene.apache.org/core/4\_5\_0/core/org/apache/lucene/store/MMapDirectory.html )。它使用mmap系统调用((http://en.wikipedia.org/wiki/Mmap )来读取和随机方式完成写文件操作。在进程中，它将文件映射到相同尺寸的虚拟内存地址空间中。由于没有任何的锁操作，多线程存取索引文件时就程序就具有可伸缩性了(可伸缩性是指当增加计算资源时，程序的吞吐量或者处理能力相应的增加)。当我们使用mmap读取索引文件，在操作系统看来，该文件已经被缓存(文件会被映射到虚拟内存中)。基于这个原因，从Lucene索引中读取一个文件时，文件不必加载到操作系统的缓存中，读取速度就会快一些。这基本上就是允许Lucene，也就是ElasticSearch直接操作I/O缓存，索引文件的存取当然会快很多。值得一提的事，MMap文件系统存储最好应用在64-位操作系统环境中，如果要用在32-位的操作系统环境中，必须确保索引足够小，而且虚拟内存空间足够。用户可以通过设置index.store.type属性值为mmapfs来使用该存储类型。 </p>

<h4>内存存储</h4>
<p>这种存储类型是几种类型中唯一不基于Apache Lucene directory实现的(当然也可以用Lucene的RAMDirectory类来实现)。内存存储类型允许用户直接把索引数据存储到内存中，所以硬盘上不会存储索引数据。记住这一点至关重要，因为这意味着数据并没有持久化：只要整个集群重启，数据就会丢失。然而，如果你的应用需要一个微型的、存取快速的，能有多个片分和分片副本的而且重建过程很快的索引，内存存储类型可能是你需要的。把index.store.type属性值设置为memeory即可使用该存储类型。</p>
<!--note structure -->
<div style="height:50px;width:650px;text-indent:0em;">
<div style="float:left;width:13px;height:100%; background:black;">
  <img src="../lm.png" height="40px" width="13px" style="margin-top:5px;"/>
</div>
<div style="float:left;width:50px;height:100%;position:relative;">
	<img src="../note.png" style="position:absolute; top:30%; "/>
</div>
<div style="float:left; width:550px;height:100%;">
	<p style="font-size:13px;margin-top:5px;">存储在内存中的索引数据，与其它存储类型相似，也会在允许存数据的节点上保留分片副本。</p>
</div>
<div style="float:left;width:13px;height:100%;background:black;">
  <img src="../rm.png" height="40px" width="13px" style="margin-top:5px;"/>
</div>
</div> <!-- end of note structure -->

<h4>其它的属性</h4>
<p></p>

<h4>默认存储类型</h4>
<p></p>
</div>
