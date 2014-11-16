## 查询重写机制

<div>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;如果你曾经使用过很多不同的查询类型，比如前缀查询和通配符查询，从本质上上，任何的查询都可以视为对多个关键词的查询。可能用户听说过查询重写(query rewrite)，ElasticSearch(实际上是Apache Lucene很明显地)对用户的查询进行了重写，这样做是为了保证性能。整个重写过程是把从Lucene角度认为原始的、开销大的查询对象转变成一系列开销小的查询对象的一个过程。<br/>
<h3>以前缀查询为例</h3>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;展示查询重写内部运作机制的最好方法是通过一个例子，查看例子中用户输入的原查询语句中的term在内部被什么Term所取代。假定我们的索引中有如下的数据：
<div style="margin-left:20px;">
curl<span style="margin-left:1em;"/>-XPUT<i style="margin-left:1em;"/>'localhost:9200/clients/client/1'<i style="margin-left:1em;"/>-d</br>
'{</br>
<i style="margin-left:1em;"/>"id":"1",<i style="margin-left:1em;"/>"name":"Joe"</br>
}'</br>
curl<i style="margin-left:1em;"/>-XPUT<i style="margin-left:1em;"/>'localhost:9200/clients/client/2'<i style="margin-left:1em;"/>-d</br>
'{</br>
<i style="margin-left:1em;"/>"id":"2",<i style="margin-left:1em;"/>"name":"Jane"</br>
}'</br>
curl<i style="margin-left:1em;"/>-XPUT<i style="margin-left:1em;"/>'localhost:9200/clients/client/3'<i style="margin-left:1em;"/>-d</br>
'{</br>
<i style="margin-left:1em;"/>"id":"3",<i style="margin-left:1em;"/>"name":"Jack"</br>
}'</br>
curl<i style="margin-left:1em;"/>-XPUT<i style="margin-left:1em;"/>'localhost:9200/clients/client/4'<i style="margin-left:1em;"/>-d</br>
'{</br>
<i style="margin-left:1em;"/>"id":"4",<i style="margin-left:1em;"/>"name":"Rob"</br>
}'</br>
curl<i style="margin-left:1em;"/>-XPUT<i style="margin-left:1em;"/>'localhost:9200/clients/client/5'<i style="margin-left:1em;"/>-d</br>
'{</br>
<i style="margin-left:1em;"/> "id":"5",<i style="margin-left:1em;"/>"name":"Jannet"</br>
}'</br>
</br>
</div>
</div>
<br/><!--note structure -->
<div style="height:110px;width:650px;">
    <div style="float:left;width:13px;height:100%; background:black;">
        <img src="../lm.png" height="100px" width="13px" style="margin-top:5px;"/>
    </div>
    <div style="float:left;width:50px;height:100%;position:relative;">
	    <img src="../note.png" style="position:absolute; top:30%; "/>
    </div>
<div style="float:left; width:550px;height:100%;">
<b><br/>源码下载</b>
	<p style="font-size:13px;">所有购买了Packt出版的图书的读者都可以用自己的账户从http://www.packtpub.com. 下载源代码文件。如果读者通过其它的途径下载本书，则需要通过http://www.packtpub.com/support 来注册账号，然后网站会把源码通过e-mail发送到你的邮箱。 </p>
</div>
<div style="float:left;width:13px;height:100%;background:black;">
  <img src="../rm.png" height="100px" width="13px" style="margin-top:5px;"/>
</div>
</div> <!-- end of note structure -->
<br/>
<div>我们的目的是找到所有以字符 j 开头的文档。这个需求非常简单，在 client索引上运行如下的查询表达式：<br/>
<p style="">
curl -XGET 'localhost:9200/clients/_search?pretty' -d '{<br/>
<i style="margin-left:1em;"/>"query" : {<br/>
<i style="margin-left:1em;"/> "prefix" : {<br/>
<i style="margin-left:1em;"/>"name" : "j",<br/>
<i style="margin-left:1em;"/>"rewrite" : "constant_score_boolean"<br/>
<i style="margin-left:1em;"/>}<br/>
<i style="margin-left:1em;"/>}<br/>
}'</p>
我们用的是一个简单的前缀查询；前面说过我们的目的是找到符合以下条件的文档：name域中包含以字符 j 开头的Term。我们用rewrite参数来指定重写查询的方法，关于该参数的取值我们稍后讨论。运行前面的查询命令，我们得到的结果如下：<br/>
<div style="margin-left:20px;">
{<br/>
<i style="margin-left:5px;"/> ...<br/>
<i style="margin-left:5px;"/> "hits" : {<br/>
<i style="margin-left:10px;"/> "total" : 4,<br/>
<i style="margin-left:10px;"/> "max_score" : 1.0,<br/>
<i style="margin-left:10px;"/> "hits" : [ {<br/>
<i style="margin-left:20px;"/> "_index" : "clients",<br/>
<i style="margin-left:20px;"/> "_type" : "client",<br/>
<i style="margin-left:20px;"/> "_id" : "5",<br/>
<i style="margin-left:20px;"/> "_score" : 1.0, "_source" : {"id":"5","name":"Jannet"}<br/>
<i style="margin-left:10px;"/>  }, {<br/>
<i style="margin-left:20px;"/>  "_index" : "clients",<br/>
<i style="margin-left:20px;"/>  "_type" : "client",<br/>
<i style="margin-left:20px;"/>  "_id" : "1",<br/>
<i style="margin-left:20px;"/>  "_score" : 1.0, "_source" : {"id":"1", "name":"Joe"}<br/>
<i style="margin-left:10px;"/>  }, {<br/>
<i style="margin-left:20px;"/> "_index" : "clients",<br/>
<i style="margin-left:20px;"/> "_type" : "client",<br/>
<i style="margin-left:20px;"/> "_id" : "2",<br/>
<i style="margin-left:20px;"/> "_score" : 1.0, "_source" : {"id":"2", "name":"Jane"}<br/>
<i style="margin-left:10px;"/> }, {<br/>
<i style="margin-left:20px;"/> "_index" : "clients",<br/>
<i style="margin-left:20px;"/> "_type" : "client",<br/>
<i style="margin-left:20px;"/> "_id" : "3",<br/>
<i style="margin-left:20px;"/> "_score" : 1.0, "_source" : {"id":"3", "name":"Jack"}<br/>
<i style="margin-left:10px;"/> } ]<br/>
<i style="margin-left:5px;"/> }<br/>
}<br/>
</div>


</div>


