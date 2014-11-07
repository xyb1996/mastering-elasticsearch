## Lucene查询语言
<hr>
<div>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ElasticSearch提供的一些查询方式(query types)能够被Lucene的查询解析器(query parser)语法所支持。由于这个原因，我们来深入学习Lucene查询语言，了解其庐山真面目吧。</div>

### 基础语法
<div>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;用户使用Lucene进行查询操作时，输入的查询语句会被分解成一个或者多个Term以及逻辑运算符号。一个Term，在Lucene中可以是一个词，也可以是一个短语(用双引号括引来的多个词)。如果事先设定规则：解析查询语句，那么指定的analyzer就会用来处理查询语句的每个term形成Query对象。</div>
<br/>
<div>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;一个Query对象中会存在多个布尔运算符，这些布尔运算符将多个Term关联起来形成查询子句。布尔运算符号有如下类型：
<hr style="height:4px;"/>
<ul>
    <li><span style="font-family:COURIER;">AND(与)</span>:&nbsp;&nbsp;&nbsp;给定两个Term(左运算对象和右运算对象)，形成一个查询表达式。只有两个Term都匹配成功，查询子句才匹配成功。比如：查询语句"apache AND lucene"的意思是匹配含apache且含lucene的文档。</li>
    <li><span style="font-family:COURIER;">OR(或)</span>:&nbsp;&nbsp;&nbsp;给定的多个Term，只要其中一个匹配成功，其形成的查询表达式就匹配成功。比如查询表达式"apache OR lucene"能够匹配包含“apache”的文档，也能匹配包含"lucene"的文档，还能匹配同时包含这两个Term的文档。</li>
    <li><span style="font-family:COURIER;">NOT(非)</span>:&nbsp;&nbsp;&nbsp; 这意味着对于与查询语句匹配的文档，NOT运算符后面的Term就不能在文档中出现的。例如：查询表达式“lucene NOT elasticsearch”就只能匹配包含lucene但是不含elasticsearch的文档。</li>
</ul>
<br/>
此外，我们也许会用到如下的运算符：
<ul>
    <li><b>+</b>&nbsp;&nbsp;:&nbsp;&nbsp;&nbsp;&nbsp;这个符号表明：如果想要查询语句与文档匹配，那么给定的Term必须出现在文档中。例如：希望搜索到包含关键词lucene,最好能包含关键词apache的文档，可以用如下的查询表达式："+lucene apache"。</li>
    <li><b>-</b>&nbsp;&nbsp;:&nbsp;&nbsp;&nbsp;&nbsp;这个符号表明：如果想要查询语句与文档匹配，那么给定的Term不能出现在文档中。例如：希望搜索到包含关键词lucene,但是不含关键词elasticsearch的文档，可以用如下的查询表达式："+lucene -elasticsearch"。</li>
</ul>
如果在Term前没有指定运算符，那么默认使用OR运算符。<br/>
此外，也是最后一点：查询表达式可以用小括号组合起来，形成复杂的查询表达式。比如：
    <center><span style="color:gray;font-family:COURIER;background-color:#F7F7F7;">elasticsearch AND (mastering OR book)</span></center>
</div>

### 多域查询
<div>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;当然，跟ElasticSearch一样，Lucene中的所有数据都是存储在一个个的Field中，多个Field形成一个Document。如果希望查询指定的Field,就需要在查询表达式中指定Field Name(域的命名)，后面接一个冒号，紧接着一个查询表达式。例如：查询title域中包含关键词elasticsearch的文档，查询表达式如下：
  <center><span style="color:gray;font-family:COURIER;background-color:#F7F7F7;">title:elasticsearch</span></center>
也可以把多个查询表达式用于一个域中。例如：查询title域中含关键词elasticsearch并且含短语“mastering book”的文档，查询表达式如下：
 <center><span style="color:gray;font-family:COURIER;background-color:#F7F7F7;">title:(+elasticsearch +"mastering book")</span></center>
 当然，也可以换一种写法，作用是一样的：
  <center><span style="color:gray;font-family:COURIER;background-color:#F7F7F7;">+title:elasticsearch +title:"mastering book")</span></center>

</div>
##  词语修饰符
<div>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;除了可以应用简单的关键词和查询表达式实现标准的域查询，Lucene还支持往查询表达式中传入修饰符使关键词具有变形能力。最常用的修饰符，也是大家都熟知的，就是通配符。Lucene支持?和\*两种能配符。?可以匹配任意单个字符，而\*能够匹配多个字符。
</div><br/><br/>
<div style="height:57px;margin-left:20px;float:left;"><img src="../tipsL.png"/></div>
<div style="height:52px;width:65%;float:left;word-wrap: break-word;word-break: normal; color:gray;font-family:COURIER;font-size:12px;background-color:#F7F7F7;padding-top:5px;">请注意出于性能考虑，默认的通配符不能是关键词的首字母。</div>
<div style="height:57px;float:left;"><img src="../tipsR.png"/></div>
<br/>
<div style="clear:both;">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;此外，Lucene支持模糊查询(fuzzy query)和邻近查询(proximity query)。语法规则是查询表达式后面接一个~符号，后面紧跟一个整数。如果查询表达式是单独一个Term，这表示我们的搜索关键词是Term的变形。这种搜索方式称为模糊搜索(fuzzy search)。在~符号后面的整数表示最大编辑距离。例如：执行查询表达式 "writer~2"能够搜索到含writer和writers的文档。</div>
<div>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;当~符号用于一个短语时，~后面的整数表示短语中可接收的最大的词编辑距离。举个例子,查询表达式title:"mastering elasticsearch"只能匹配title域中含"mastering elasticsearch"的文档，而无法匹配含"mastering book elasticsearch"的文档。但是如下查询表达式变成title:"mastering elasticsearch"~2,那么两种文档就都能够成功匹配了。</div></br>
<div>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;此外，我们还可以使用加权(boosting)机制来改变关键词的重要程度。加权机制的语法是一个^符号后面接一个数字表示权重。如果权重小于1，就会降低关键词的重要程度。同理，如果权重大于1就会增加关键词的重要程度。默认的加权值为1。可以参考<span style="font-style:oblique">第2章 活用用户查询语言</span>的<span style="font-style:oblique">Lucene默认打分规则详解</span>来了解更多关于加权(boosting)是如何影响打分排序的。</div>
