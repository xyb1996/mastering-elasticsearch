## 解析你的文本

<div>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;问题到这里就变得稍微复杂了一些。传入到Document中的数据是如何转变成倒排索引的？查询语句是如何转换成一个个Term使高效率文本搜索变得可行？这种转换数据的过程就称为解析</div>
<br/>
<div>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;文本解析工作由<b>analyzer</b>组件负责。analyzer由一个分词器(tokenizer)和0个或者多个过滤器(filter)组成,也可能会有0个或者多个字符mappers组成。</div>
<div>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Lucene中的<b>tokenizer</b>用来把文本拆分成一个个的Token。Token包含了一些额外的信息，比如Term在文本的中的位置及Term原始文本，以及Term的长度。文本经过<b>tokenizer</b>处理后的结果称为token stream。token stream其实就是一个个Token的顺序排列。token stream将等待着filter来处理。</div>
<br/>
<div>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;除了<b>tokenizer</b>外，Lucene的另一个重要组成部分就是filter链，filter链将用来处理Token Stream中的每一个token。这些处理方式包括删除Token,改变Token，甚至添加新的Token。Lucene中内置了许多filter，读者也可以轻松地自己实现一个filter。有如下内置的filter：
<ul>
<li><b>Lowercase filter</b>：把所有token中的字符都变成小写</li>
<li><b>ASCII folding filter</b>：去除tonken中非ASCII码的部分</li>
<li><b>Synonyms filter</b>：根据同义词替换规则替换相应的token</li>
<li><b>Multiple language-stemming filters</b>：把Token(实际上是Token的文本内容)转化成词根或者词干的形式。</li>
</ul>
</div>
<div>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;所以通过Filter可以让analyzer有几乎无限的处理能力：因为新的需求添加新的Filter就可以了。</div>

### 索引和查询

<br/>
<div>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在我们用Lucene实现搜索功能时，也许会有读者不明觉历：上述的原理是如何对索引过程和搜索过程产生影响？</div>
<br/>
<div>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;索引过程：Lucene用用户指定好的analyzer解析用户添加的Document。当然Document中不同的Field可以指定不同的analyzer。如果用户的Document中有<span style="font-family:COURIER">title</span>和<span style="font-family:COURIER">description</span>两个Field，那么这两个Field可以指定不同的analyzer。</div>
<div>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;搜索过程：用户的输入查询语句将被选定的查询解析器(query parser)所解析,生成多个Query对象。当然用户也可以选择不解析查询语句，使查询语句保留原始的状态。在ElasticSearch中，有的Query对象会被解析(analyzed)，有的不会，比如：前缀查询(prefix query)就不会被解析，精确匹配查询(match query)就会被解析。对用户来说，理解这一点至关重要。</div>
<div>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;对于索引过程和搜索过程的数据解析这一环节，我们需要把握的重点在于：倒排索引中词应该和查询语句中的词正确匹配。如果无法匹配，那么Lucene也不会返回我们喜闻乐见的结果。举个例子：如果在索引阶段对文本进行了转小写(lowercasing)和转变成词根形式(stemming)处理，那么查询语句也必须进行相同的处理，不然就搜索结果就会是竹篮打水——一场空。</div>


