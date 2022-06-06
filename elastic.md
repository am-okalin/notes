[TOC]
## elastic套件职能
- kibana 数据可视化
- elasticsearch 数据搜索，分析，存储
    + 功能：基于JSON的开源分布式搜索引擎，大数据近实时分析引擎(具有很强大数据分析能力)
    + 特性：高性能，高可靠，高扩展，管理便捷，实时性(和T+1说不)
- 不同来源，格式的数据采集
    + beats 轻量级采集器平台
    + logstash 动态数据收集管道

## logstash
输入方式：`输入`->`filters`->`output`

## kibana
- devTool开发工具
    + `ctrl+/` 查看API帮助文档
    + `ctrl+option+0`
- 插件安装
```
bin/kibana-plugin install plugin_location
bin/kibana-plugin list
bin/kibana remove
```

## es部署
```shell
docker pull docker.elastic.co/elasticsearch/elasticsearch:7.13.3

vim /etc/sysctl.conf
vm.max_map_count=262144
sysctl -w vm.max_map_count=262144
```


### 数据安全
- `Transport Layer Security` 传输层安全`TLS`是`node`间的通信协议。防止未经授权的`node`访问您的集群。
- `the certificate authority` 证书安全机构`CA`。生成的`CA`包含`公共证书`和`私钥`
    + 集群通过识别证书属于同一`CA`签发对证书进行验证
    + 传输层建议使用不被共享的专用`CA`
- 生成证书
    1. 生成`elastic-stack-ca.p12` 包含`公共证书`和`私钥`
    2. 生成`elastic-certificates.p12` 包含`节点证书`，`节点密钥`，`CA证书`
    3. 将`elastic-certificates.p12` copy到集群每个节点的`ES_PATH_CONF`

```shell
# 生成CA 非生产环境下密码可为空
./bin/elasticsearch-certutil ca
# 生成节点共享证书
./bin/elasticsearch-certutil cert --ca elastic-stack-ca.p12
```

#### config
```shell
# minimal security 即user/pass验证
xpack.security.enabled=true

# basic security 设置以启用节点间通信并提供对节点证书的访问
xpack.security.transport.ssl.enabled: true
xpack.security.transport.ssl.verification_mode: certificate
xpack.security.transport.ssl.client_authentication: required
xpack.security.transport.ssl.keystore.path: elastic-certificates.p12
xpack.security.transport.ssl.truststore.path: elastic-certificates.p12

# https
xpack.security.http.ssl.enabled: true
xpack.security.http.ssl.keystore.path: elastic-certificates.p12
xpack.security.http.ssl.truststore.path: elastic-certificates.p12
```

- `xpack.security.transport.ssl.verification_mode` 证书验证方式
    + `none` 不执行服务器证书的验证，开发环境时可使用
    + `certificate` 验证`证书`是否由受信任的`CA`签发
    + `full`(default) 除了验证`CA`外还会验证`证书`中标识的ip或hostname


## es基本概念
- elasticsearch 用于数据搜索，分析，存储
    + 功能：基于JSON的开源分布式搜索引擎，大数据近实时分析引擎(具有很强大数据分析能力)
    + 特性：高性能，高可靠，高扩展，管理便捷，实时性(对T+1说不)
- `cluster state`集群状态:维护了一个集群中必要的信息。
    + 所有的`node`信息
    + 所有的`index`信息(包括`mapping`和`settings`)
    + `shard`的路由信息

### `document`文档
- 是可搜索的最小的数据单元：日志中的日志项；一部电影的具体信息；一篇文档的具体内容
- `doc`被序列化成`json`对象格式，每个字段都有类型可指定或由`es`推算
- 每个`doc`都有一个`uniqueID`，可以自己指定或通过es生成
- 文档的元数据用于标注文档的相关信息
    + `_index` 文档所属索引名
    + `_type` 文档所属类型名
    + `_id` 文档唯一ID
    + `_source` 文档原始json数据
    + `_version` 文档的版本信息(大量并发时解决文档冲突)
    + `_source` 相关性打分(用于全文检索)

### `index`索引
- `index`是`doc`的容器，是一类`doc`的结合。
- `mapping`用于定义文档字段的类型。表示逻辑空间的概念
- `settings`用于定义数据的物理分布。表示物理空间的概念
- 中文`索引`的不同语义
    + `名词`：指容器的集合
    + `动词`：保存`doc`到`es`的过程也叫`indexing`索引
    + `名词`：特指一个`B树索引`或`倒排索引`
- `_type` 索引的类型，7.x后都为`_doc`

### `node`节点
- `node`是一个`es`实例，本质上是一个java进程，生成环境建议一台机器只运行一个实例
- 每个`node`都有一个名字，启动时`-E node.name=node1`指定或者通过配置文件指定
- 每个`node`启动后会被分配一个uid，保存在data目录下
- 每个节点上都保存了集群的状态，每个集群只有一个节点拥有管理`cluster state`的权限被称为`master node`，若`master node`挂掉或集群刚启动则从被第一个指定的`master-eligible node`中选举`master node`。
- [cross-cluster search](https://www.elastic.co/guide/en/elasticsearch/reference/current/xpack-ccr.html)

#### 节点角色
- 一个`node`根据其配置/特性可承担多种角色，通过配置`elasticsearch.yml`的`node.role`对节点角色进行配置。
- `dedicated node`具有单一角色的节点的统称。在生产环境中建议每个节点应为`dedicated node`(性能好，职责明确)
- 客户端有可能请求不同节点上的数据，此时由接受请求的节点负责协调，协调节点将请求转发目标节点，目标节点本地执行后将返回结果发给`coordinating node`协调节点汇总。`coordinating node`需要足够的内存和cpu以便处理聚集阶段。每个节点都是隐式的协调节点。
- 具有单一职责的`coordinating node`被称为`coordinating only node`。其配置为`node.role:[]`
- 单一数据节点，对数据节点进一步特化，不可拥有多个数据节点角色。
    + `data_hot`/`data_warm`/`data_cold` 根据数据读取频率配置不同硬件，降低部署成本
    + `data_content` 内容数据节点适应用户创建的内容。 它们启用CRUD，搜索和聚合等操作
    + `data_frozen` [冻结节点](https://www.elastic.co/guide/en/elasticsearch/reference/current/searchable-snapshots.html#partially-mounted)

|    node.role     | 参数选项 |                     职责                     |
|------------------|----------|----------------------------------------------|
| master-eligible  | master   | 拥有成为`master node`资格的节点              |
| coordinating     | 无参数   | `接受>分发请求到其他节点>汇总返回结果`的节点 |
| data             | data     | 保存数据的节点                               |
| machine learning | ml       | 负责跑机器学习的job，用来做异常检测          |
| ingest           | ingest   | 摄取管道在文档`ingdexing`之前对文档进行处理  |

### `shard`分片
- `primary shard`主分片：用以解决数据水平扩展问题。通过主分片将数据分布到集群内的所有节点上
    + 一个分片是一个运行的lucene实例
    + `number_of_shards`在`index`创建时指定，除非`reindex`否则不允许修改
- `replica shard`副本分片：用以解决数据高可用问题。分片是主分片的拷贝
    + `number_of_replicas`可动态调整
    + 增加`number_of_replicas`可在一定程度上提高服务的可用性(增加读取性能)
- 由于`number_of_shards`不可更改，所以在生产环境中需提前做好分片数的容量规划。从es7.0开始主分片数默认为1。
- 分片数设置过小带来的问题
    + 后续无法增加节点实现水平扩展
    + 每个分片的数据量太大，导致数据重新分配耗时
- 分配数设置过大`over-sharding`
    + 影响搜索结果的相关性打分，影响统计结果的准确性
    + 导致资源浪费，影响性能
- 通过`rest api`查看`cluster`健康状况。status字段以颜色反映集群状态
    + `green` 主分片与副本都正常
    + `yellow` 主分片分配正常，副本分片未能正常分配
    + `red` 有主分片未能分配

### 关系型数据库与es对比
- ES多用于 相关性、高性能全文检索，多用于查询
- RDMS 事务操作，

|     RDBMS      | Elasticsearch |
|----------------|---------------|
| table          | index(type)   |
| row            | document      |
| column         | field         |
| schema(表定义) | mapping       |
| SQL            | DSL           |

### 倒排索引
- `elasticsearch`的`doc`中的每个字段，都有自己的倒排索引。也可以对指定字段不做索引
    + 优点：节省存储空间
    + 缺点：字段无法被搜索
- 搜索引擎中的`正排索引`与`倒排索引`
    + 正排索引：文档ID到单词`term`的索引
    + 倒排索引：`term`单词到文档ID的索引
- `倒排索引`的核心组成：由`term dictionary`单词词典和`posting list`倒排列表组成
    + `term dictionary`记录所有文档的`term`到倒排列表的关联关系
    + `term dictionary`比较大，由`b+数`或`哈希拉链法`实现以满足高性能的CURD
    + `posting list`记录单词和对应文档的关联，由`posting`倒排索引项组成
- `posing`倒排索引项包含
    + `doc_id`文档ID
    + `TF`词频：该单词在文档中出现的次数，用于相关性评分
    + `position`位置：单词在文档中分词的位置，用于语句搜索`phrase query`
    + `offset`偏移：记录单词的开始结束位置，实现高亮显示

正排索引

| 文档ID |         文档内容         |
|--------|--------------------------|
|      1 | mastering elasticsearch  |
|      2 | elasticsearch server     |
|      3 | elasticsearch essentials |

倒排索引

|      term     | count | doc_id:position |
|---------------|-------|-----------------|
| elasticsearch |     3 | 1:1,2:0,3:0     |
| mastering     |     1 | 1:0             |
| server        |     1 | 2:1             |
| essentials    |     1 | 3:1             |

倒排索引中`term`为`elasticsearch`的`posting list`

| doc_id | TF | position |  offset |
|--------|----|----------|---------|
|      1 |  1 |        1 | [10,23] |
|      2 |  1 |        0 | [0,13]  |
|      3 |  1 |        0 | [0,13]  |

