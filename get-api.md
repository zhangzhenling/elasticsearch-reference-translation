>* 文章名称：Elasticsearch Reference[2.2]
>* 原文地址：[https://www.elastic.co/guide/en/elasticsearch/reference/2.2/api-conventions.html](https://www.elastic.co/guide/en/elasticsearch/reference/2.2/api-conventions.html)
>* 译者：[code4j](https://github.com/rpgmakervx)

# Get API

get api 支持通过 ID 返回一个 JSON 格式的文档。下面的例子是从 twitter 索引下的 tweet type 中获取 ID 为1的文档：

> curl -XGET 'http://localhost:9200/twitter/tweet/1'

返回结果如下：

```json
{
    "_index" : "twitter",
    "_type" : "tweet",
    "_id" : "1",
    "_version" : 1,
    "found": true,
    "_source" : {
        "user" : "kimchy",
        "postDate" : "2009-11-15T14:12:12",
        "message" : "trying out Elasticsearch"
    }
}
```

上面的结果中包含了文档的 `_index`,`_type`,`_id`,`_version` 等我们希望获取的字段，包括 `_source`字段，如果文档能被找到的话（通过返回结果中的 `found` 字段表明）。

这个 API 还能使用 HTTP 的 `HEAD` 请求,例如：

> curl -XHEAD -i 'http://localhost:9200/twitter/tweet/1'

## 实时性

默认情况下，get API 是实时的，而且不会受索引刷新频率的影响（文档在索引中是否能被检索到）。（**译者批注：目前没找到有说明 get api 详细流程的地方，译者认为既然和刷新频率无关，get api 很有可能是先走内存的**）

如果你想取消实时性 GET 的话，一个办法是请求时设置 `realtime` 参数为 `false`,或者通过节点级全局配置参数 `action.get.realtime` 设置为 `false`。

当获取一个文档时，可以设置获取确切的属性 `fields`。一般来说这些字段可能是被存储的（mapping 中设置字段为 stored），当使用实时的 GET 操作的时候，没有stored字段的概念（一段时间内，基本上一次flush的时间），而是直接从source字段中提取这些字段（即使source被禁用）（**译者批注：这句话有一定的迷惑性，译者开始以为原文的意思是 即使禁用了也能从source取出，而作者的意思是不管你的 store 是不是 true，都去source 字段里面读，也就是说如果你source字段禁用了，get 是取不到任何数据的。**）。实时 GET 的时候比较好的做法是从source拿字段，不管属性有没有被存储。（**实际上如果字段被存储了，就不会走source了，直接去 Lucene 的倒排索引拿，字段多的时候IO开销比较厉害，所以应该是读者的建议，默认 store 即可，除非你的字段很大，但是不多**）

## 可选的类型

Get API 允许使用 `_type` 字段，如果指定 `_all` 可以查询所有的类型。
（**译者批注：感觉用处不大，5.0以后开始淡化 type 的概念了，现在基本上一个索引一个 type **）

## source过滤

默认情况下，Get 操作会返回 source 的全部内容，除非你指定了 `fields` 参数或者 `_source` 字段被禁用。你可通过设置 `_source` 字段禁止返回source中的内容：

> curl -XGET 'http://localhost:9200/twitter/tweet/1?_source=false'

如果你只需要 `_source` 中一两个字段，你可以使用 `_source_include` 或者 `_source_exclude` 参数设置包含或者过滤你需要的字段。这个在文档很大的时候可以节省网络开销。这两个参数都可以通过都好分割或使用通配符，例如：

> curl -XGET 'http://localhost:9200/twitter/tweet/1?_source_include=*.id&_source_exclude=entities'

如果你只想指定包含字段的花，可以使用更短的声明：

> curl -XGET 'http://localhost:9200/twitter/tweet/1?_source=*.id,retweeted'

## 字段

Get 操作可以通过参数 `fileds` 指定返回一组设置存储(`store`)的字段。例如：

> curl -XGET 'http://localhost:9200/twitter/tweet/1?fields=title,content'

为了向后兼容，如果请求中的字段并没有被存储，他们将从 `_source` 字段获取（解析并获取），所以这个功能被 source 过滤取代了。

字段值总是以数组的形式返回，原数组字段例如 `_routing` 就不会用数组格式返回。
（**这个原因大概是因为，elasticsearch 本身就支持多值字段，这种情况下 多值字段中只有一个值的时候它并不知道这个到底是不是多值的，所以干脆都给你数组**）

当然只有叶子节点的字段能通过 `field` 字段返回，所以对象类型的字段不可以返回并且这样的请求会报错。





