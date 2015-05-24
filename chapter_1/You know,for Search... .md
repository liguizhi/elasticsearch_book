Es是一个基于`Apache Lucene`(一个全文检索引擎库)的开源搜索引擎。Lucene毫无疑问是迄今为止最高级，性能最好，功能最强大的搜素引擎库——无论是开源还是私有库。

但是，Lucene只是一个库。要想获得更强大的应用，你得使用java开发并且在应用中直接继承Lucene。更糟的是，想要弄懂Lucene如何工作，你得在信息检索方面有更深的造诣。Lucene非常复杂。

Elasticsearch 也是用Java写的，并且内部的索引和搜索功能都是基于Lucene。不过，它的目的是让全文检索变得非常容易，使用简单相关的RESTful API隐藏了Lucene背后的复杂性。

然而，Elasticsearch又不只是基于Lucene，也不止是全文检索。它还是：
- 分布式实时文档存储系统，所有字段都可以索引和检索
- 拥有实时分析功能的分布式搜索引擎
- 能扩展到数百台服务器，处理PB级别的结构化或非结构化数据

它把各项功能集成到一台独立的服务器，你可以通过RESTful API方式跟它通信，无论是用你习惯的编程语言客户端，还是用命令行方式。

Elasticsearch非常拥有上手，它对初学者隐藏了复杂的搜索理论，把默认配置显性化。开箱即用。只需简单的理解，你就可以用它来开发。

Elasticsearch可以免费下载，使用或者修改。它基于Apache 2许可协议。

随着你的知识的增长，你可以使用更强大的Elasticsearch功能。整个搜索引擎都是可以配置的，而且非常灵活。从那些高级功能中选择你需要的，并应用到你的场景中吧。

# 安装Elasticsearch #
最简单的方式就是用起来，所以，开始吧！

Elasticsearch只需要近期版本的Java，最好是从Java官方下载最新版并安装。

你可以从elasticsearch.org/download中获取最新的Elasticsearch版本。
```
    curl -L -O http://download.elasticsearch.org/PATH/TO/VERSION.zip①
    unzip elasticsearch-$VERSION.zip
    cd elasticsearch-$VERSION
    ①在URL处填elasticsearch.org/download中获取的最新版本
```
# 安装Marvel #
Marvel是一个Elasticsearch的管理和监控工具，免费用于开发。它有一个交互控制台Sense，可以用来从浏览器端之间与Elasticsearch交互。

本书中很多代码样例都带有“View in Sense”字样，如果点击的话，就会打开一个Sense控制台的样例代码。你可以不必安装Marvel，但如果安装的话，你在本地Elasticsearch集群中体验样例代码的时候就很有互动性了。

Marvel可以通过插件方式获取，想要下载和安装，可以运行以下命令：
```
    ./bin/plugin -i elasticsearch/marvel/latest
```
你可能不希望Marvel监控本地集群，可以通过以下设置禁用数据连接
```
    echo 'marvel.agent.enabled:false' >> ./config/elasticsearch.yml
```

# 运行Elasticsearch #

Elasticsearch现在已经可以运行了，你可以通过前台方式运行：
```
    ./bin/elasticsearch
```
如果想要后台运行的话，只需增加参数`-d`

可以通过新的命令行窗口运行以下命令进行测试：
```
    curl 'http://localhost:9200/?pretty'
```
你应该能看到以下响应：
```
    {
        "status":200,
        "name":"Shrunken Bones",
        "version":{
            "number":"1.4.0",
            "lucene_version":"4.10"
        },
        "tagline":"You Know, for Search"
    }
```
这就意味着Elasticsearch集群(cluster)已经运行了，我们可以继续体验。
>集群(clusters)和节点(nodes):一个节点运行一个Elasticsearch实例，一个集群是一组相同名称(cluster.name)的节点。这些节点共享数据以便共同提供故障转移和扩展性。当然，一个单一节点也可以构成一个集群。

你可以把默认的cluster.name改成你熟悉的名字，比如你自己的名字，以免你的节点自动在网络上连接了其他相同名字的节点！

通过修改elasticsearch.yml文件即可完成上述操作，记得要重启Elasticsearch。当Elasticsearch在前台运行的时候，你可以通过CTRL-C来结束运行，或者，也可以通过关闭命令的API来结束：
```
    curl -XPOST 'http://localhost:9200/_shutdown'
```
# 查看Marvel和Sense #
如果你安装了Marvel管理和监控工具，只需要在浏览器中输入以下命令即可查看:`http://localhost:9200/_plugin/marvel/`.

# 跟Elasticsearch交互 #
如何跟Elasticsearch交互取决于你是否是用java。

## Java API ##
如果你使用Java开发，那么Elasticsearch提供了两种内置的客户端代码：
*Node client*
节点客户端把本地集群连接成非数据节点。换句话说，它本身不存储数据，但它知道什么数据存储在哪个节点，并且能把请求直接转发到正确节点。
*Transport client*
更轻量级的转发客户端可以用来向远程集群发送请求。它本身不参与集群，只是把请求转发到集群中的节点。

这两种Java客户端都通过9300端口跟集群交互，使用了Elasticsearch本地的传输协议。集群中的节点，也通过9300端口相互通信。如果这个端口没有打开，那么你的节点就无法形成一个集群。
>相应的Java客户端必须跟Elasticsearch节点的版本一致，否则他们可能无法相互理解。

更多Java客户端可以通过查阅手册中的Java API章节。

## RESTful API with JSON over HTTP ##
所有其他语言，都可以使用RESTful API通过9200端口与Elasticsearch通信，用你熟悉的web客户端即可。相信你已经发现，你甚至可以通过命令行的curl命令与Elasticsearch交互。
>Elasticsearch提供了集中语言的官方客户端——Groovy，JavaScript，.NET，PHP，Perl，Python，以及Ruby——并且有很多社区提供的客户端，所有这些都可以在手册中找到。

一个Elasticsearch请求跟任何HTTP请求的组成部分相同。举例来说，如果要统计集群中的文档数量，我们可以这么查：
```
            ①    ②                       ③       ④
    curl -XGET 'http://localhost:9200/_count?pretty' -d '
    {
            ⑤
        "query":{
            "match_all":{}
        }
    }
    '
①   相应的HTTP方法或者动作：GET,POST,PUT,HEAD或者DELETE
②   集群中任意节点的协议，主机名，以及端口
③   请求路径
④   pretty是可选参数，如果设置的话，返回结果会以JSON格式排版，便于阅读
⑤   JSON格式化的请求主题（如果需要的话）
```
Elasticsearch返回HTTP状态码比如200 OK以及(除HEAD请求外)JSON编码的相应主体。上面的curl请求将会返回类似的JSON主体：
```
    {
        "count":0,
        "_shards":{
            "total":5,
            "sussessful":5,
            "failed":0
        }    
    }
```
我们没看到HTTP头信息，因为我们没有要求curl返回头信息。要看头信息，只需在curl命令中增加`-i`选项：
```
    curl -i -XGET 'localhost:9200/'
```

本身其他部分，我们将用简写格式处理curl样例，把请求中相同的信息，比如主机名，端口号，以及curl命令本身，都去掉。替换前的请求样例如下：
```
    curl -XGET 'localhost：9200/_count?pretty' -d'
    {
        "query":{
            "match_all":{}
        }
    }
    '
```
简写格式如下
```
    GET /_count
    {
        "query":{
            "match_all":{}
        }
    }
```

事实上，这个用Sense控制台使用的格式是一样的。你可以通过连接“View in Sense”查看本代码样例。

## 面向文档 ##
程序中对象往往不止是键值对列表，通常他们是复杂的数据结构，包含日期，地理位置信息，对象，或者数值组。

很快你会在数据库中存储这些对象。想通过传统数据库中的行和列来存储这些信息，无异于想把含义丰富的对象挤压到一个巨大的电子表格中：你必须把对象做扁平化处理以便适应表格的格式——通过是每列一个字段——并且每次检索的时候都得还原回来。

Elasticsearch是面向文档的，意味着它存储整个对象文档。不只是存储，它还索引文档的内容，以便用于搜索。在Elasticsearch中，你索引，检索，排序，或者过滤文档都不是通过行列式的数据。这是不同于传统数据思考方式的基本方法，正是Elasticsearch能提供复杂全文检索的原因。

## JSON ##

Elasticsearch使用JSON（即JavaScript对象标记法）作为序列号文档的格式。JSON序列化方法被绝大多数编程语言支持，并且已经成为NoSQL运动中的事实标准。它非常简单，清晰，易于阅读。

把这个JSON文档想象成一个用户对象：
```
    {
        "email":"john@smith.com",
        "first_name":"John",
        "last_name":"Smith",
        "info":{
            "bio":"Eco-warior and defender of the weak",
            "age":25,
            "interests":["dolphins","whales"]
        },
        "join_date":"2014/05/01"
    }
```
尽管原始的用户对象非常复杂，但是在这个JSON版本中，对象的结构和含义都包含进来了。把对象转化成JSON以便索引，对Elasticsearch来说，比处理扁平化表结构更容易。
>几乎所有的语言都有把无序的数据结构或者对象转化成JSON格式的模块，只是不同语言的实现细节有别。具体查阅处理JSON序列化模块。Elasticsearch官方客户端中，已经自动提供了JSON格式转化。

## Finding your feet ##
我们来通过一个简单的教程来给你更直观的印象，以及阐述Elasticsearch多么易用，其中包含了基础的概念，比如索引，检索和聚合。

我们将持续介绍一些新的名词和基本概念，但即使你不能马上理解的话也问题不大。本书的其余部分中，我们会继续深入介绍此处涉及的概念。

所以，坐好了，来一次感受Elasticsearch能力的旋风之旅吧。

## Let's build an employee directory ##

我们正巧为大公司打工，作为HR部门新活动*We love our drones!*的第一步，我们被分派的任务是建立雇员目录。这个目录需要能增强员工的认同感，提供实时的协同动态合作，所以，它有这几项业务需求：
>数据能存储多值的标签，数组，以及全文文本
>能检索任何雇员的全部信息
>允许结构化搜索，比如查询年龄大于30岁的雇员
>允许简单的全文搜索和复杂的短语搜索
>从匹配的文档中提供高亮的文本摘要
>能使管理部门基于数据建立分析面板

## 索引雇员文档 ##
业务的第一步就是存储雇员数据。需要约定一个雇员文档的格式，一个雇员文档代表一个雇员信息。在Elasticsearch中存储数据的过程就叫建索引（indexing），当然，建索引之前我们得明确把数据存储在什么地方。

在Elasticsearch中，一个文档归属于一种类型（type），并且不同的类型（types）可以存在于一个索引（index）中。可以大致跟传统关系数据库做个类比：
>    Relation DB => Databases => Tables => Rows => Columns
>    Elasticsearch => Indices => Types  => Documents => Fields

一个Elasticsearch集群可以包含多个索引(indices,数据库）,索引中又可以包含多种类型(types,类似tables).这些类型包含了多个文档(documents,类似于rows),每个文档包含多个字段(field,类似于column)。
> ## Index vs Index vs Index ##
> 你可能已经注意到，index在Elasticsearch的语境中有不同的含义，此处有必要做个澄清：
> 
- Index(名词):如上所述，一个索引就类似传统数据库中的数据库。是存储相关文档的地方。复数是indices或indexes
- Index(动词):建立文档索引，即把文档存储到索引中以便检索和查询。类似于SQL中的insert操作，不同之处在于，如果文档已经存在的话，新文档会替换老文档。
- Inverted index：反向索引。关系型数据库为了增加列查询的速度，会增加数据库索引，类似于B-Tree索引。Elasticsearch和Lucene为了达到同样目的，用的是被称作反向索引的结构。默认情况下，文档的每个字段都被索引（具有反向索引）并且能被搜索。一个不具备反向索引的字段是无法被搜索的。我们会在后续反向索引章节中进行深入讨论。

所以，对于我们的雇员目录来说，我们需要进行以下工作：
- 为每个雇员建立索引文档，文档中包含单个雇员的全部信息
- 每个文档都是employee类型（type）
- 这个类型在megacorp索引（index）下
- 索引存储在我们的Elasticsearch集群中

实践中，这个过程非常简单（虽然看上去像是很多步）。我们只需要执行一条命令完成上述步骤：
```
    PUT /megacorp/employee/1
    {
        "first_name":"John",
        "last_name":"Smith",
        "age":25,
        "about":"I love to go rock climbing",
        "interests":["sports","music"]
    }
```
注意路径中的/megacorp/employee/1包含了三部分信息：
**megacorp**是索引(index)名
**employee**是类型(type)名
**1**是这个雇员的ID