## 解析你的文本

<div>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;到这里问题就变得稍微复杂了一些。传入到Document中的数据是如何转变成倒排索引的？查询语句是如何分解成一个个Term使搜索变得可行？这种数据转变的过程就称为文本解析</div>
<br/>
<div>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;文本解析是由<b>analyzer</b>组件负责。analyzer由一个分词器(tokenizer)和0个或者多个过滤器(filter)组成,也可能会有0个或者多个字符mappers组成。</div>
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
<br/>
<div>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;  。所以通过Filter可以让analyzer有几乎无限的处理能力，新的需求添加新的Filter就可以了。</div>

## 索引和查询


