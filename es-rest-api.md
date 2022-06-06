[TOC]
## 基础概念
- [接口文档](https://www.elastic.co/guide/en/elasticsearch/reference/7.13/rest-apis.html) 
- 常见错误返回
    + 连接无法关闭：节点出错
    + 429：集群过于繁忙(重试或增加节点)
    + 4xx：请求体格式错误
    + 500：集群内部错误

## 分词
```shell
# 在settings中自定义analyzer
# todo::自定义分词

#指定analyzer进行测试，
GET _analyze
{
    "analyzer": "english",
    "text": "Mastering Elasticsearch, elasticsearch in Action"
}

#指定索引字段进行测试
POST index_name/_analyze
{
    "field": "comment",
    "text": "Mastering Elasticsearch"
}

#自定义分词器进行测试
POST _analyze
{
    "tokenizer": "standard",
    "filter": ["lowercase", "stop"],
    "text": "Mastering in Elasticsearch"
}
```

- `analysis`文本分析/分词：把全文本转换一系列单词`term`/`token`的过程。
- `analyzer`分词器：专门处理分词的组件，可用es内置分词器或自定义分词器。
- 除了数据写入时使用`analyzer`转换词条，匹配query语句时也需要使用相同的`analyzer`对查询语句进行分析
- `analyzer`的组成与分词过程
    + `character filters` 原始文本的字符过滤，如去除html
    + `tokenizer` 按照一定规则切分单词
    + `token filters` 将切分单词进行加工：小写、删除stopwords、增加同义词
- `elasticsearch`内置的analyzers
    + `standard` 默认，按词切分，小写处理
    + `simple` 按照非字母切分(数字符号被过滤)，小写处理
    + `stop` 小写处理，`stopwords`过滤(如the, a, is等)
    + `whitespace` 按空格切分，不转小写
    + `keyword` 不分词(不做倒排索引)，直接将输入作为输出。
    + `patter` 正则表达式，默认\w+(非字符分隔)
- `language analyzer` 提供了30多钟语言的分词器。它的`analyzer`字段直接指定为语言字段如`english`、`thai`
- `customer analyzer` 自定义分词器即指定`analyzer`的组成部分
- 中文分词要用`elasticsearch-plugin install pulgin_name`安装插件
    + `icu_analyzer` 安装`analyzer-icu`插件使用
    + [IK](https://github.com/medcl/elasticsearch-analysis-ik)：支持定义词库，热更新分词词典
    + [THULAC](https://github.com/microbun/elasticsearch-thulac-plugin)：清华大学自然语言处理和社会人文计算实验室的中文分词器

### character filters
- 在tokenizer前对文本进行处理。可配置多个`character filters`,会影响`tokenizer`的`position`和`offset`信息
- es自带`character filters`
    + HTML strip 去除html标签
    + Mapping 字符串替换
    + Pattern replace 正则匹配替换

### Tokenizer
- 将原始文本按照一定规则，切分为词(term or token)
- Elasticsearch 内置的 Tokenizers
    + whitespace 空格分词
    + standard 标准分词
    + uax_url_email url或emailfenci
    + pattern 正则分词
    + keyword 不做分词
    + path hierarchy 文件路径分词
- 可用java开发插件，自己实现tokenizer

### token filters
- 将tokenizer输出的term 进行增加，修改，删除
- 自带的token filters
    + lowercase
    + stop
    + synonym 添加近义词

## CURD&批量操作
- 为解决重新建立网络开销损耗性能的问题，开放了`_bulk`、`_mget`、`index_name/_msearch`接口在1次api调用中执行多次操作
- `_bulk`可在URI中指定index，也可在请求`payload`中指定index。它能在一次API调用中对不同索引做`index`/`create`/`update`/`delete`等操作。返回结果包含每条执行的结果，单条操作失败并不影响其他操作。

```shell
GET _cluster/health
GET _cluster/health?level=shards
GET _cluster/health/index_name
GET _nodes/es01,es02

GET _cat/shards
GET _cat/nodes
GET _cat/nodes?v
GET _cat/nodes?v&h=id,ip,port,v,m
GET _cat/indices?v

PUT index_name
GET index_name
DELETE index_name
GET index_name/_count
GET index_name/_mapping
#创建文档：PUT方式可指定创建ID
POST index_name/_doc
{"user":"mike", "comment":"you know, for search"}
PUT index_name/_doc/1?op_type=create
{"user":"mike", "comment":"you know, for search"}
PUT index_name/_create/1 
{"user":"mike", "comment":"you know, for search"}
#索引文档：文档不存在就保存文档；否则删除旧文档后保存新文档，版本号+1
PUT index_name/_doc/1?op_type=index //op_type默认为index，可省略
{"user":"mike", "comment":"you know, for search"}
#更新文档：对文档进行数据更新，`payload`修改内容需包含在`doc`字段中
POST index_name/_update/1
{"doc":{"user":"mike", "comment":"you know, for search"}}
GET index_name/_doc/1
DELETE index_name/_doc/1

POST _bulk
{"index": {"_index": "index_name", "_id": 1}}
{"user":"mike", "comment":"you know, for search"}
{"delete": {"_index": "index_name", "_id": 1}}
{"create": {"_index": "index_name", "_id": 1}}
{"user":"mike", "comment":"you know, for search"}
{"update": {"_index": "index_name", "_id": 1}}
{"doc":{"user":"mike", "comment":"you know, for search"}}

GET _mget
{"docs":[{"_index":"index_name", "_id":1}, {"_index":"test", "_id":1}]}

GET index_name/_msearch
{}
{"query": {"match_all":{}}, "from":0, "size":10}
{"index": "test"}
{"query": {"match_all":{}}}
```


## Mapping
### Dynamic Mapping
- 在写入文档时如果索引不存在，mapping会把json文档映射成lucene所需的扁平格式后自动创建索引。
- `mappings.dynamic` 的值，在新增字段时产生的影响
    + true 会自动更新mapping
    + false mapping不更新，新增字段数据无法被索引，但信息会出现在`_source`中
    + strict 文档索引失败
    + 对于已有字段的maping一旦生成无法被修改。想要修改必须使用reindex api

### 字段类型
- Text/Keyword
- Date
- Integer/Floating
- Boolean
- IPv4 & IPv6
- 对象类型/嵌套类型
- geo_point & geo_shape / percolator
- es中不提供数组类型，但任何字段都可包含多个相同类型的数值

|           ES类型           |    JSON类型    |
|----------------------------|----------------|
| Date                       | 日期格式字符串 |
| float/long                 | 数字           |
| Text(带keyword子字段)      | 其他字符串     |
| boolean                    | 布尔值         |
| float                      | 浮点数         |
| long                       | 整数           |
| Object                     | 对象           |
| 由第一个非空数值的类型决定 | 数组           |
| 忽略                       | 空值           |

### mapping常见参数
- `mappings.properties.fieldName.index` 控制字段是否被索引
    + 默认位true
    + false 时该字段不可被搜索
- `mappings.properties.fieldName.index_options` 控制倒牌索引记录的内容
    + `docs` 记录`doc id`
    + `freqs` 记录`doc id`/`term frequencies`
    + `positions` 记录`doc id`/`term frequencies`/`term position`
    + `offsets` 记录`doc id`/`term frequencies`/`term position`/`character offects`
    + 字段类型为`Text`的 默认记录`postions`，其他默认为`docs`
    + 记录内容越多，占用存储空间越大
- `mappings.properties.fieldName.null_value` 需要对`null`值实现搜索
    + 只有`keyword`类型支持设定`null_value`
- `mappings.properties.fieldName.copy_to` 将字段的数值拷贝到目标字段
    + `_all`在7中被`copy_to`替代
    + 满足一些特定的搜索需求
    + `copy_to`的目标字段不出现在`_source`中
- `mappings.date_detection` 将日期转为date类型(默认true)
- `mappings.numeric_detection` 将日期转为数值类型(默认false)

## search API
- `_search`API有`uri search`和`request body search`(推荐)两种查询方式
- `_search`的语法与查询范围：
    + `_search` 指查询集群上的所有索引
    + `index1/_search` 指定范围为`index1` 
    + `index1,index2/_search` 指定范围为`index1`和`index2` 
    + `index*/_search` 使用通配符表示范围为以`index`开头的索引
- `_search`的`response`中的结果集按相关性进行排序
    + `took` 花费的时间
    + `hits.total` 返回结果数
    + `hits.hits` 结果集，默认前10个`doc`
- `information retrieval` 衡量相关性的指标
    + `precision`查准率：尽量返回较少的无关文档。值为`true positive`/`positive`
    + `recall`查全率：尽量返回较多的相关文档。值为`true negatives`/`negatives`
    + `ranking`：是否能按相关度进行排序
- 根据`precision`&`recall`对搜索数据进行分类
    + `true positive` 被搜索到的正确数据
    + `false positive` 被搜索到的错误数据
    + `true negatives` 没被搜索到的错误数据
    + `false negatives` 没被搜索到的正确数据

### 请求体参数
```json
POST index_name/_doc
{"user":"mike", "comment":"you know, for search", "sort":4}

GET /index_name/_search
{
    "_source": ["user", "comment", "sort"],
    "from": 0,
    "size": 10,
    "profile": false,
    "sort": [{"sort":"desc"}],
    "script_fields": {
        "new_field": {
            "script": {
                "lang": "painless",
                "source": "doc['sort'].value * params['str']",
                "params": {
                    "str": 2
                }
            }
        }
    },
    "query": {
        "match_all":{}
    }
}

```

- `_source` 对返回结果`field`进行过滤
- `from` 页码
- `size` 页长
- `profile` 是否查看查询执行情况
- `profile.shards.searches.query.type`表示查询类型
    + `term query`: `querystr`为`A B`表示A or B
    + `phrase query`: `querystr`为`"A B"`表示A and B，并且前后顺序一致
    + `disjunction max query`: 泛查询时对所有字段都做了匹配
    + `boolean query`
- `script_fields`脚本字段使用es中的脚本算出结果
    + `lang` 指定使用的脚本语言
    + `source` 脚本内容
    + `params` 定义变量
- `query.match` 定义字段匹配规则
    + `query` 查询字符串
    + `operator` 操作符 默认为`OR`
- `query.match_phrase` 定义字段连词匹配规则
    + `query` 查询字符串，每个单词按顺序出现(空格分割)
    + `slop` 中间插入指定数量的 `term`


## es聚合搜索`aggregation`
- `aggregation` 除搜索外，提供对ES数据进行统计分析的功能。它具有`高实时性`和`高性能`的特点。在`kibana`中的`可视化报表`都是使用了`aggregation`
    + `高实时性`: 解决T+1
    + `高性能`: 只要一条语句，就能从es得到分析结构。无需客户端自己实现分析逻辑
- 通过聚合，我们会得到数据概览，是分析和总结全套数据，而不是寻找单个文档。如
    + 尖沙咀和香港岛的客房数量
    + 不同价格区间，可预定的经济型酒店和五星级酒店数量
- `aggregation`集合的分类
    + `Bucket Aggrregation` 一些列满足特定条件的文档和集合
    + `Metric Aggregation` 一些数学运算，可对文档字段进行统计分析
    + `Pipeline Aggregation` 对其他的聚合结果进行二次聚合
    + `Matrix Aggregration` 支持对多个字段的操作并提供一个结果矩阵

### `term level query`
- `term level query` 针对输入不做分词，在倒排索引中查找准确词项，并使用相关度算分公式为每个文档进行相关度算分。它的分类有
    + `term query`  term查询
    + `exists query` 查找空值
    + `prefix query` 前缀查询
    + `wildcard query` 通配符查询
    + `range query` 范围查询，日期range范围字符为`y M w(周) d H/h m s`
- 可通过`constant score`将`query`转为`filter`, 避免相关性算分的开销，并利用缓存提高性能(filter可有效利用缓存)

```json
POST index_name/_search 
{
    "profile": "true", 
    "explain": true,
    "query": {
        "term": {
            "field1": true
        },
        "range": {
            "price": {
                "gte": 20,
                "lte": 30
            },
            "date": {
                "gte": "now-1y"
            }
        }
    }
}

POST index_name/_search 
{
    "profile": "true", 
    "explain": true,
    "query": {
        "constant_score": {
            "filter": {
                "term": {
                    "field1": true
                }
            }
        }
    }
}
```

### 全文查询
- 全文查询包括
    + `match query`
    + `match phrase query`
    + `query string query`
- 索引和搜索时都会进行分词，查询字符串先传递到一个合适的分词器，然后生成一个供查询的词项列表
- 查询时会先对输入查询进行分词，然后每个词项逐个进行底层查询，最终将结果进行合并。并为每个文档生成一个算分。
    + 例如`matrix reloaded`会查到包括matrix获取reload的所有结果
    + 可通过`operator`控制搜索字符串的逻辑关系

### 结构化搜索`structured search`
- bool,时间，日期，数字这类结构化数据：有精确格式，可对格式进行逻辑操作，包括比较数字或时间范围，或判定两个值的大小
- 结构话的文本可以做精确匹配或部分匹配，如`term query`或`perfix query`
- 结构化结构只有是否两种值，可根据场景需要据欸的那个结构化搜索是否需要打分


## `aggregation`语法
```json
"aggregations": {
    "<aggregation_name>": {
        "<aggregation_type>": {
            <aggregation_body>
        }
        [,"mete": {[<meta_data_body>]}]?
        [,"mete": {[<sub_aggregation>]}]?
    }
    [, "<aggregation_name_2>": {...}]*
}
```

- `aggregation`属于`search`的一部分，一般情况下建议将其size指定为0,即不返回文档只返回聚合结果
- `aggregations` 和`query`同级的关键字
- `aggregation_name` 自定义聚合名字
- `aggregation_body` 聚合的定义，不同的Type+Body
- `sub_aggregation` 子聚合查询

## metric aggregation 的 aggregation_type
- 单值分析：只输出一个分析结果
    + min, max, avg, sum
    + cardinality (类似distinct count)
- 多值分析：输出多个分析结果
    + stats, extended stats
    + percentilem, percentile rank 求百分位数
    + top hits(排在最前面的示例)


## bucket aggregation
- 按照一定规则，将文档分配到不同的桶中，从而达到分类的目的。支持嵌套
- es提供一些常见的bucket aggregation
    + terms
    + 数字类型 range / data range
    + 数字类型 histogram / date histogram

### terms aggregation
- 字段需要打开fielddata, 才能`terms aggregation`
    + keyword默认支持doc_values
    + text需要在mapping中`fielddata: true`
- 假设job为text类型，job.keyword表示不会分词


## `reindex`
- 新建`{index_name}_new` 
- 将`{index_name}`reindex至`{index_name}_new`
- 将`{index_alias}`别名绑定为`{index_name}_new`
- 删除`{index_name}`