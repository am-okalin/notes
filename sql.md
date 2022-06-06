<script id="MathJax-script" async src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-mml-chtml.js"></script>

[TOC]
## MYSQL基础
### 逻辑操作符
|    操作符     |           语法          |            描述            |
|---------------|-------------------------|----------------------------|
| NOT或`!`      | NOT a                   | 逻辑非                     |
| AND或`&&`     | a AND b                 | 逻辑与                     |
| OR或`||`      | a OR b                  | 逻辑或                     |
| XOR           | a XOR b                 | 逻辑异或                   |

### 比较运算符
|    操作符     |           语法          |            描述            |
|---------------|-------------------------|----------------------------|
| <=>           | a <=> b                 | 类似=，可用于NULL判断       |
| IS (NOT) NULL | a IS (NOT) NULL         | (非)NULL判断               |
| (NOT) BETWEEN | a (NOT) BETWEEN b AND c | 若a(不)在bc之间为真        |
| (NOT) LIKE    | a (NOT) LIKE b          | 模式匹配，若a(不)匹配b为真 |
| IN            | a IN (b1,b2,...)        | 若a等于b1,b2...中某个为真  |

### 数据类型
#### 数值型
|    类型   | 存储空间 |                     说明                     |
|-----------|----------|----------------------------------------------|
| tinyint   | 1字节    | 微小整数，可表示2<sup>8</sup>-1个数          |
| smallint  | 2字节    | 小整数，可表示2<sup>16</sup>-1个数           |
| mediumint | 3字节    | 中整数，可表示2<sup>24</sup>-1个数          |
| int       | 4字节    | 标准整数，可表示2<sup>32</sup>-1个数         |
| bigint    | 8字节    | 大整数，可表示2<sup>64</sup>-1个数          |
| float     | 4/8字节  | 单精度浮点数                                 |
| double    | 8字节    | 双精度浮点数整数                             |
| decimal   | 自定义   | 字符串表示浮点数，取值范围取值存储空间字节数 |

- 浮点型 `float(6,2)`: 显示长度为6，2表示小数位长度

#### 字符串
|          类型         |  存储空间 |         说明         |     取值范围     |
|-----------------------|-----------|----------------------|------------------|
| char(M)               | M字节     | 定长字符串           | M字节            |
| varchar(M)            | L+1字节   | 可变字符串           | M字节            |
| tinyblob,tinytext     | L+1字节   | 微小blob，微小文本串 | 2<sup>8</sup>-1  |
| blob,text             | L+2字节   | 小blob，小文本串     | 2<sup>16</sup>-1 |
| meduimblob,meduimtext | L+3字节   | 中blob，中文本串     | 2<sup>24</sup>-1 |
| longblob,longtext     | L+4字节   | 大blob，大文本串     | 2<sup>32</sup>-1 |
| enum('v1','v2',...)   | 1/2字节   | 枚举: 可赋予某个成员 | 2<sup>16</sup>-1 |
| set('v1','v2',...)    | 1/2/3/4/8 | 集合: 可赋予多个成员 | 2<sup>6</sup>    |

- `L`: 实际存储长度
- `enun`和`set`是特殊串类型，列值从串值中选择
- `blob`: 二进制大对象, blob与test可存大文件(通常不建议这么做)。blob区分大小写，text不区分大小写

#### 其他
- 时间/日期:  通常使用整形时间戳记录时间
- null:  即空值，0与null判断时都视为false


### 数据字段属性
- `auto_increment` 自增，必须依附主键/唯一索引
- `unsigned` 无符号，用于数值型，存储长度增加1倍
- `zerofill` 只用于数值类型，数值前用0补齐不足的位数
- `null`/`not null` 默认null,若指定`not null`要写入默认值
- `default` 默认值


### 函数
- count 返回记录数，count(*/1)返回包括null的行数，count(列)返回不包括null的行数
- sum 返回总和
- avg 返回平均数
- max/min 返回最大/最小值
- find_in_set(str,strlist) 如`find_in_set(id, '1,4,7')` 返回id为1,4,7的行
- any/some
- all

### 索引
- 定义: 索引是帮助mysql高效获取数据的数据结构。由B+树实现
    + 是一种m叉树，叶子节点由双向链表关联，且仅叶子节点存储数据域
- 索引会降低IO的次数(提高查询的效率)
- 索引会减慢DML(增删改)操作，频繁更新的字段不适用
- 索引类型
    + `primarry key` 主键索引: 唯一，不可为空，不可重复，且建议每个表都有主键索引
    + `unique` 唯一索引: 唯一，不可重复
    + `index` 普通索引: 可重复，可创建多个
    + `fulltext` 全文索引
- 联合索引: 即多列索引，包括联合唯一索引 与 联合普通索引
- 辅助索引: 即非主键索引会创建`叶子节点`存储`主键ID`的`B+树`，
- 覆盖索引: 仅通过辅助索引就命中了全部的查询字段，无需回表查询
- 回表查询: 辅助索引不能命中全部查询字段时，会先查询辅助索引树，在用辅助索引命中的ID查询主键索引树获取全部数据
    + 可通过将单列索引升级为联合索引实现覆盖索引避免回表
- 索引扫描做排序: `排序操作`很消耗资源，通过索引扫描可实现排序，达到优化的目的
    + 多列索引时要保证索引顺序与`order by`一致
- `ICP`索引下推: 先根据索引查找记录，再根据条件过滤记录，因此产生多余的回表查询
    + 在ICP优化后，先进行where条件过滤再进行索引查询

## DCL(data control language)
- GRANT 允许`对象创建者`给某组用户或所有用户(PUBLIC)某些权限
- REVOKE 废除 用户/组/所有用户 的访问权限
- `SET TRANSACTION` 设置当前事务状态
- `COMMIT` 提交事务
- `ROLLBACK` 回滚事务

## DDL(data definition language)
### CREATE 创建数据库和数据库的一些对象
```sql
CREATE TABLE user(
    id int unsigned not null auto_increment,
    tel char(11) not null,
    pass char(32) not null,
    name varchar(50) not null index,
    sex enum('男','女') not null default '男',
    primary key(id),
    unique tel_unique(tel),
)engine=InnoDB default charset=UTF8;
```

### ALTER 修改数据表定义及属性
- 语法 `ALTER TABLE table action`
- 修改字段:  `CHANGE`可更改字段名和属性 `MODIFY`只可更改属性
    + `ALTER TABLE table CHANGE field_old field_new varchar(32) not null`
    + `ALTER TABLE table MODIFY field varchar(32) not null`
- 添加字段/索引:  `ADD`
    + `ALTER TABLE table ADD field varchar(32) not null`
    + `ALTER TABLE table ADD index/unique index_name(field1, field2)`
    + `ALTER TABLE table ADD primary key(field)`
- 删除字段/索引:  `DROP`
    + `ALTER TABLE table DROP field`
    + `ALTER TABLE table DROP index/unique index_name`
    + `ALTER TABLE table DROP primary key`
- 更改表名称:  `RENAME AS`
    + `ALTER TABLE table RENAME AS table_new`
- 更改属性auto_increment的值: `ALTER TABLE table auto_increment=1`

### 其他DDL语句
- DROP 删除
    + `DROP TABLE table`删除表
- SHOW 查看
    + `SHOW TABLES` 查询当前库中所有表
    + `SHOW DATABASES` 查询所有库
    + `show indexes from table` 查看table表的索引
- `DESC table` 显示表结构
- `USE database;` 选择库

## DML(data manipulation language)
- 插入数据: `INSERT INTO`
    + `INSERT INTO table[(field1,...,fieldn)] VALUES(v11,...,v1n)[,...]`
- 修改数据: `UPDATE`
    + `UPDATE table SET field=表达式[,...] [where 条件]`
- 删除数据: `DELETE`
    + `DELETE FROM table [where 条件]`

## DQL(data query language)
### 基本语法
- 编写过程: `select..from..join..on..where..group by..having..order by..limit`
- 解析过程: `from..on..join..where..group by..having..select..order by..limit`
- `AS` 字段别名，用于字段之后如`SELECT md5(123) as md5 ..`
- `DISTINCT` 数据去重，用于`SELECT`字句之后，查询字段之前。
    + `DISTINCT` 会占用系统资源，单表内查询建议使用索引
- `LIMIT [start,]len` 限定结果行数。start表示开始条数，可省默认0；len表示长度
- `ORDER BY` 对最终查询结果排序(子查询中不能有`ORDER BY`) `ASC`升序，`DESC`降序。
- `GROUP BY` 对查询结果分组
    + `GROUP BY`字句中不支持字段别名，也不支持任何使用了统计函数的集合列
    + 在完成数据结果分组查询和统计后，可用`HAVING`字句对结果进行进一步筛选
    + 例`SELECT uid,sum(score) AS total FROM mark GROUB BY (uid)`
    + 例`SELECT uid,xk,min(score) AS min FROM mark GROUB BY (uid) HAVING min<70`

### 连接查询
- `a表 jion_type b表 using(同名字段)`using中的字段用于连接两表的同名字段。
- `SQL语句 UNION [ALL/DISTINCT] SQL语句` union将两条语句查询结果组合到一个结果集中
- 多表连接会先连接ab表，再与c表连接获取最终结果集
    + `SELECT a.*,c.* FROM a,b,c WHERE a.id=b.a_id AND b.c_id=c.id`

#### 自身连接
- 自身连接用于单表连接查询，通常要为一张表起两个别名
- 例: 一张表内每个id都有一个父id，求每一个id的间接父id(即父id的父id)
- 解: `SELECT a.id,b.pid FROM table a,table b WHERE a.pid=b.id`

#### 内连接
- 交叉连接`cross join` 也称为笛卡尔积连接，没有where条件。
    + `SELECT a.*,b.* FROM a,b`    
    + `SELECT a.*,b.* FROM a CROSS JOIN b`
- 自然连接 是特殊的等值连接。将等值连接中的重复字段去掉就是自然连接(常与using连用)
    + `SELECT 两表中不重复字段 from a,b where a.id=b.id`
    + `SELECT * FROM a NATRUAL JOIN b`
- 等值连接 where条件中用于连接两个表的比较运算符为`=`
    + `SELECT a.*,b.* FROM a,b WHERE a.id=b.id`
    + `SELECT a.*,b.* FROM a INNER JOIN b ON a.id=b.id`
- 非等值连接 where条件中用于连接两个表的比较运算符不为`=`
    + `SELECT a.*,b.* FROM a,b WHERE a.age BETWEEN b.age_i AND b.age_o`
    + `SELECT a.*,b.* FROM a INNER JOIN b ON a.age BETWEEN b.age_i AND b.age_o`
- 嵌套循环连接算法 是内连接使用的匹配算法
    + 以`SELECT a.*,b.* FROM a,b WHERE a.id = b.id`为例
    + 取第一行`a.id`与每个`b.id`比较，符合就列为结果行中的一行，再取第二行..直到结束。
    + 如果`b.id`建立了索引就不必全表扫描了

#### 外连接
- 左外连接 把左表中元组全部选出，尽管有些行没有左表的对应数据
    + `SELECT a.*,b.* FROM a LEFT JION b ON a.id=b.id`
- 右外连接 类比左外连接。若只选出右边那些没有对应数据的行SQL如下: 
    + `SELECT a.*,b.* FROM a RIGHT JION b ON a.id=b.id WHERE a.id IS NULL`
- 全外连接`FULL JION` 将左右两表中所有行选出，尽管有些行没有对应数据
    + `SELECT a.*,b.* FROM a FULL JOIN b ON a.id=b.id`
- MySQL没有全连接`FULL OUTER`语法。想实现查询ab表中有对应数据的所有行sql如下: 
    + `SELECT a.*,b.* FROM a LEFT JOIN b ON a.id = B.id IS NULL UNION SELECT a.*,b.* FROM a RIGHT JOIN b ON A.Key = B.Key IS NULL`
    + 将`IS NULL`去掉就是一个用左右外连接加集合查询实现的全连接


### 嵌套查询(子查询)
- 按`子查询的查询条件`与`父查询的关系`分类: 子查询的查询条件(不)依赖于父查询 称为(不)相关子查询。
    + `select a.* from a where a.id in (select b.id from b where b.key=a.key)`
    + `select a.* from a where a.id in (select b.id from b where b.key=3)`
    + 相关子查询反复从父查询拿匹配字段到子查询匹配，是效率较低的查询
- 比较运算符常用IN取匹配集合。例:取出每个栏目(cut)下的最新产品: 
    + `SELECT cat_id,goods_id,goods_name FORM goods WHERE goods_id IN (SELECT max(goods_id) FROM goods GROUP BY cut_id)`
- 带有ANY(SOME)/ALL函数的子查询
    + `select a.* from a where a.id >any(select b.id from b);`

#### 带有EXISTS谓词的函数
- EXISTS代表存在量词`∃`表达`(∃x)P`。带有EXISTS谓词的子查询不反回数据，只产生逻辑真假。
- EXISTS通常与不相关子查询连用。如查询哪些栏目(category)下有商品(goods)
    + `select cate_id from category where exists(select * from goods where goods.cate_id=category.cate_id);`
- SQL中没有全称量词`∀`无法表达`(∀x)P`,要等同表达为`┐(∃x(┐P))`。
- 如查询选修了全部课程的学生，转为: 没有一门课是他不选修的。
```sql
SELECT * FORM student where not exists(
    select * from course where not exists(
        select * from sc where s_id=student.id and c_id=course.id
    )
);
```

### 集合查询
- `UNION`并集，会自动取出重复元素，若要保留使用`UNION ALL`
- `INTERSECT`交集
- `EXCEPT`差集
- `SELECT a.*,b.* FROM a LEFT JOIN b ON a.id = B.id IS NULL UNION SELECT a.*,b.* FROM a RIGHT JOIN b ON A.Key = B.Key IS NULL`

### 基于派生表的查询
- from型子查询:把内层查询结果供外层再次查询
- 例 查出挂科两门及以上的同学平均成绩
    + `1` `SELECT stu_id,count(*) as gk FROM scores WHERE score<60 GROUP BY stu_id HAVING gk>=2` 
    + `2` `SELECT stu_id from (1) as t`
    + `3` `SELECT stu_id,avg(score) form stu where stu_id in (2) group by stu_id`
- from子查询相当于查询一个临时派生表，在`2`中使用的就是from型子查询，实际上这个例子不强行使用from子查询会更简单一点
d

## 关系模式数据表设计
- `relation<U,F>` `U`表示属性 `F=R(U)`表示关系。
- `Sname=f(Sno)` 表示Sno函数决定Sname,Sname函数依赖Sno
- 例 学生关系模式`student<U,F>`，其中`Mname`表示系主任
    + `U={Sno,Sdept,Mname,Cno,Grade}`
    + `F=R(U)={Sno→Sdept,Sdept→Mname,(Sno,Cno)→Grade}`

### 函数依赖
- `X→Y`表示`X函数确定Y`或`Y函数依赖X`。X称为这个函数依赖的`决定属性组`或`决定因素`。若Y不函数依赖X 则记作$$X\cancel{\rightarrow}Y$$
- 若`X→Y`且`Y⊆X`，则称`X→Y`是`平凡函数依赖` 如`(s_id,c_id)→s_id`
- 若`X→Y`且`Y⊈X`，则称`X→Y`是`非平凡函数依赖` 如`(s_id,c_id)→grade`
- 若`X→Y`且`Y→X`，则记作`X←→Y` 
- 完全函数依赖 如成绩`grade`完全函数依赖`(s_id,c_id)`
$$X \rightarrow Y \quad \forall X' \subset X \quad X' \cancel{\rightarrow} Y \quad 则X\stackrel{F}{\longrightarrow}Y \quad如(s,c)\stackrel{F}{\longrightarrow}grade$$
- 部分函数依赖 如系`sdept`部分函数依赖`(s_id,c_id)`
$$X \rightarrow Y \quad X\cancel{\stackrel{F}{\longrightarrow}}Y \quad 则X\stackrel{P}{\longrightarrow}Y \quad 如(s,c)\stackrel{P}{\longrightarrow}sdept $$
- 传递函数依赖 如`s→Sdept`且`Sdept->Mname`则`Mname`传递函数依赖`s`
$$X\stackrel{非平凡}{\longrightarrow}Y\quad Y\cancel{\rightarrow}X\quad Y\stackrel{非平凡}{\longrightarrow}Z\quad 则X\stackrel{传递}{\longrightarrow}Z$$
- `(非)平凡`与`完全/部分`可看作是函数依赖两个维度可以共存

#### 多值依赖
- 对于函数依赖的优化顶多到BCNF，想要优化到4NF就要理解`多值依赖`的概念。多值依赖涉及到

### 键(码)与属性
- 定义: 设`K`为关系模式`R<U,F>`中的属性集
$$若K\stackrel{P}{\longrightarrow}U \text{ 则K为R的超码(surper key);}$$
$$若K\stackrel{F}{\longrightarrow}U \text{ 则K为R的候选码(candidate key);}$$
- 候选码是最小的超码，即K的任意一个真子集都不是候选码
- 若`R<U,F>`中候选码多于一个，则选其中一个为`主码(primary key)`。通常情况下`主码`/`候选码`都被简称为`码`
- 若`R<U,F>`中`U`(全部属性)是候选码，则U被称为`全码(all-key)`
- 若`R<U,F>`中`X`是`另一关系模式`的码且不是`R`的码，则称`X`是`R`的`外码(foreign key)`
- `候选码`中的属性被称为`主属性(prime attribute)`；不包含在任何候选码中的属性称为`非主属性(nonprime attribute)`；

### 范式优化
- 范式是某一级别关系模式的集合。集合关系: `5NF`⊂`4NF`⊂`BCNF`⊂`3NF`⊂`2NF`⊂`1NF`
- `低一级范式`的`关系模式`通过`模式分解`可以转化为若干个`高一级范式`的`关系模式`的集合。这个过程叫`规范化`
- 范式优化解决了DML异常/复杂的问题
- 反范式优化: 为了提升查询性能而做出的反范式优化的表设计。

#### 2NF
- 定义: 若R∈1NF,且每个`非主属性`完全函数依赖于任何一个`候选码`，则R∈2NF $$R\langle{U,F}\rangle \in{1NF}\quad K\stackrel{F}{\longrightarrow}U\quad a\in{(U-K)}\quad \forall{a}\stackrel{F}{\longrightarrow}\forall{K}\quad 则R\in2NF$$
- 例: `SLC<(Sno,Sdept,Sloc,Cno,Grade),F>`中`码`为`(Sno,Cno)`求F并优化到2范式。其中Sloc是学生住处，且每个系的学生住在一个地方。F如下
$$Sno\rightarrow Sdept\quad Sdept\rightarrow Sloc\quad Sno\rightarrow Sdept$$
$$(Sno,Cno)\stackrel{P}{\longrightarrow}Sdept\quad (Sno,Cno)\stackrel{P}{\longrightarrow}Sloc\quad (Sno,Cno)\stackrel{F}{\longrightarrow}Grade\quad$$
- 模式分解: 去除`非主属性`对`候选码`的部分函数依赖。将部分函数依赖`候选码`的`非主属性`与其完全函数依赖的候选码拿出来生成新的关系模式。得`SC<(Sno,Sdept,Sloc),F>`与`SC<(Sno,Cno,Grade),F>`
- 1NF到2NF解决了什么弊端
    + 插入异常: 如插入一个没有选课的学生，但缺少主属性`Cno`。
    + 删除异常: 删除一个学生的选课信息，就会缺少主属性`Cno`。
    + 数据冗余: Sdept，Sloc存储相同数据多次

#### 3NF
- 定义: R∈2NF，R中不存在`非主属性`对`码`的传递依赖，则R∈3NF
$$R\langle{U,F}\rangle \in{2NF}\quad K\stackrel{F}{\longrightarrow}U\quad Y,Z\subset{(U-K)}\quad Z\cancel{\subseteq}Y\quad K\cancel{\stackrel{Y传递}{\longrightarrow}}Z\quad 则R\in{3NF}$$
- 继续2NF的例子，通过模式分解将2NF优化到3NF。`SC<(Sno,Sdept,Sloc),F>`需要优化
- 模式分解: 去除`非主属性`对`码`的传递依赖。分解为`SD<(Sno,Sdept),F>`和`DL<(Sdept,Sloc),F>`
- 2NF到3NF解决了什么弊端
    + 数据冗余: Sloc列是多余的数据存储
    + 修改复杂: 修改Sdept就要连Sloc一起修改；若Sdept与Sloc对应关系改变，就要找出所有的Sdept行进行修改。

#### BCNF
- 定义: R∈3NF，R中不存在`主属性`对`码`的部分和传递依赖，则R∈BCNF
$$R\langle{U,F}\rangle \in{3NF}\quad K\stackrel{F}{\longrightarrow}U\quad a\in{K}\quad K\cancel{\stackrel{部分}{\longrightarrow}}(a)\quad K\cancel{\stackrel{传递}{\longrightarrow}}(a)\quad 则R\in{BCNF}$$
- 3NF的例子是符合BCNF的，这里举一个属于3NF但不属于BCNF的例子
    + 例 `STC((S,T,C),[(S,C)→T,(S,T)→C,T→C])`其中S学生，T老师，C课程。
    + 解 因为`T→C` 所以`(S,T)→C`是部分函数依赖；且`S,T,C`都是主属性；得此关系模式中存在主属性`C`对码`(S,T)`的部分函数依赖。
- 模式分解: 去除`主属性`对`码`的部分和传递函数依赖。将`STC`分解为`ST`与`TC`两个模式
- 3NF到BCNF解决了什么弊端
    + 插入异常: 插入老师与课程的关系，但此课程还没被选课就不存在Sno
    + 删除异常: 删除一个老师，会连学生一起删除
    + 修改复杂: 若老师与课程的映射关系改变，要修改大量数据

### 数据表设计
- 区分`不可变配置表`, `可变配置表`, `数据表`，这三者间若有关系则从前至后是`一对多`的
- `固定配置表`:  `id`做唯一标识，不做后台功能，不可自由增加，若增加则要改代码
- `可变配置表`:  `key`做唯一标识，要做后台功能，可由产品自定义增加
- `数据表`:  不可穷举的数据，要做后台功能

### 映射
- `f:A->B`: 两个非空集合A与B间存在着对应关系f，而且对于A中的每一个元素a，B中总有唯一的一个元素b与它对应，这种对应称为从A到B的映射，记作`f:A->B`
- `b=f(a)`: `f:A->B`中的一个对应关系，其中`a`被称为`原像`；`b`被称为`像`
- `原像集`: 指`A`，也被称为`定义域`
- `像集`: 指`B`，B也被称为`值域`，可记作`f(A)`
- `满射`: B中每个元素在A中都有原像。强调f(A)=B
- `单射`: A中不同的元素在B中都有不同的像。A中任意两个不同元素x1≠x2,它们的像f(x1)≠f(x2)
- `双射` = `单射` + `满射`，也被称为f是A到B上的`一一映射`。
- 满射: `A={a1,a2}`,`B={b1,b2,b3}`,`F{f(a1)=b1, f(a2)=b2}`
- 单射: `A={a1,a2,a3}`,`B={b1,b3}`,`F{f(a1)=b1, f(a2)=b1,f(a3)=b3}`
- 双射: `A={a1,a2,a3}`,`B={b1,b2,b3}`,`F{f(a1)=b1, f(a2)=b2,f(a3)=b3}`
- 普通映射: `A={a1,a2,a3}`,`B={b1,b2,b3}`,`F{f(a1)=b1, f(a2)=b1,f(a3)=b3}`

## 并发事务
- `读-读` 不会有任何问题，无需并发控制
- `写-写` 可能导致`第一类更新丢失`、`第二类更新丢失`等问题，通过乐观/悲观锁解决
- `读-写` 可能导致`脏读`、`幻读`、`不可重复读`等事务隔离问题，通过`MVCC`或提高`事务隔离级别`解决

### 相关问题
- 脏读，不可重复读，更新丢失都是同一行的问题(使用`可重复读`级别就能解决)，幻读是对表求count(\*)时的问题(要`串行化`才能解决)。
- `dirty reads`(脏读): `事务A`读取了`事务B`更新的数据，然后`B`回滚操作，那么`A`读取到的数据是脏数据
- `non-repeatable reads`(不可重复读): `事务A`多次读取同一数据，`事务B`在`事务A`多次读取的过程中，对数据作了修改并提交，导致`事务A`多次读取不一致。
- `phantom reads`(幻读): 与`不可重复读`类似。它发生在一个`事务A`读取了几行数据，接着另一个并发`事务B`插入/删除了数据后结束时。在随后A的查询中，事务A就会发现多了一些原本不存在的记录，就好像发生了幻觉一样，所以称为幻读。
- `lost update`(更新丢失): `事务A`和`事务B`选择同一行，然后基于最初选定的值更新该行时，由于两个事务都不知道彼此的存在，就会发生丢失更新问题
    + 第一类更新丢失`回滚`： A读取了B未提交的row1，A回滚row1把B的提交覆盖了
    + 第二类更新丢失`提交`： A提交row1把B提交的row1覆盖了

### `ACID`事务基本要素
- `Atomicity`(原子性): 事务不存在`中间态`，只有`完成`和`不完`成两种状态。在执行过程中发生错误会被`Rollback`到事务开始前的状态。
- `Isolation`(隔离性): 事务内部的操作的数据对其它并发事务是隔离的，并发执行的各个事务之间不能互相干扰
- `Durability`(持久性): 在事务完成以后，该事务所对数据库所作的更改便持久的保存在数据库之中，并不会被回滚
- `Consistency`(一致性): 在事务`开始前`和`结束后`，数据库的`完整性约束`没被破坏

### 事务隔离级别(由低至高)
- `READ-UNCOMMITTED`： 允许并发事务读取尚未提交的数据变更，可能会导致脏读、不可重复读、幻读、更新丢失。
- `READ-COMMITTED`： A读写row1提交后，B才能读取row1。阻止了脏读、第一类更新丢失。
- `REPEATABLE-READ`(默认)： A读写row1提交后，B才能读写row1。阻止了不可重复读、第二类更新丢失
- `SERIALIZABLE`： 最高的隔离级别，完全服从ACID的隔离级别。所有的事务串行化执行，这样事务之间就完全不可能产生干扰。但可能导致超时或锁竞争

### MVCC(`Multi-Version Concurrency Control`)
- `MVCC`多版本并发控制 它为`事务`分配单向增长的`时间戳`。并为每次`数据修改`保存一个与`事务时间戳`相关联的`版本`。
- `快照读` 不直接读取数据，而是读取`某版本`事务开始前的数据库快照
    + 无锁的解决了`读-写`问题，提高了性能
- `当前读`是悲观锁的一种操作: 读取最新记录时会数据进行加锁，防止其他事务修改数据
    + select lock in share mode (共享锁)
    + select for update (排他锁)
    + update (排他锁)
    + insert (排他锁)
    + delete (排他锁)
    + 串行化事务隔离级别
- 表中的`隐藏字段`: `db_trx_id`、`db_roll_pointer`、`db_row_id`
    + `db_trx_id` 最后一次`事务ID`
    + `db_roll_pointer` 指向这行记录的上个版本的`回滚指针`
    + `db_row_id`若表没有主键则根据`db_row_id`生成`聚簇索引`
- `undo log`会记录每一行在修改之前的数据。主要
    + 当事务进行回滚的时候可以用`undo log`的数据进行恢复。
    + 提供`快照读`的数据，通过读取`undo log`的`历史版本`可实现不同`事务版本`都拥有自己独立的`数据快照`。 
- `版本链`: 通过`undo log`中记录的`db_roll_pointer`字段组成

### `read view`
- `read view`是事务进行`快照读`操作前生成的当前数据库快照。
- `read view`的部分属性
    + `trx_ids`: 当前系统活跃(未提交)的事务版本号集合
    + `low_limit_id`: 创建`read view`时系统最大事务版本号+1
    + `up_limit_id`: 创建`read view`时系统正处于活跃事务最小版本号
    + `creator_trx_id`: 创建`read view`的事务版本号
- `read view`可见性判断条件, 仅当`db_trx_id < up_limit_id || db_trx_id == creator_trx_id`时才显示。否则对下一个版本进行判断
- `READ-COMMITTED`级别下，每个`快照读`都会生成最新的`read view`
    + `其他事务提交`对当前事务来说总是可见的
- `REPEATABLE-READ`级别下，仅同个事务的首次`快照读`才会创建`read view`， 之后的`快照读`获取的都是同一个`read view`
    + 晚于`read view`的`其他事务提交`对当前`read view`是不可见的

## mysql逻辑分层
- ![mysql架构图](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c2716016da394a9db397a8b4e978fec9~tplv-k3u1fbpfcp-zoom-1.image)
- 连接层: 提供授权认证、连接处理等功能。
    + 在该层上也可以实现基于SSL的安全链接
    + 该层会为通过授权认证的客户端请求分配相应的权限
    + 在该层上引入了线程池的概念，为通过安全认证的客户端请求提供线程
- 服务层: 这一层实现了很多核心功能，像查询解析、分析、优化、缓存、以及内置函数的实现等，所有跨存储引擎的功能也都在这一层实现，像触发器、存储过程、视图等。
- 引擎层: 存储引擎负责的是MySQL中数据的存储和提取，服务器通过API与存储引擎进行通信。索引就在引擎层
- 存储层: 存储数据在文件系统上
- 例： MySQL 的查询流程具体是？or 一条 SQL 语句在 MySQL 中如何执行的？
- 解1：连接层提供连接，服务层进行优化，引擎层选择存储引擎，最后又存储层存储
- 解2：客户端请求->连接层数据校验&线程分配->查询缓存->分析器->优化器->执行器->引擎层取数据

### 存储引擎
- 一个数据库中多个表可以使用不同引擎以满足各种性能和实际需求，MySQL服务器使用可插拔的存储引擎体系结构，可以从运行中的 MySQL 服务器加载或卸载存储引擎。
- `show engines` 查看存储引擎
- `show variables like '%storage_engine%'` 查看默认存储引擎
- `show table status from database where name="tablename"` 查看数据状态(包含引擎)
- 表在数据库目录下都有对应表的`.frm`文件，`.frm`文件保存每个表的元数据`meta`信息，包括表结构的定义等，与数据库存储引擎无关。任何存储引擎的数据表都必须有`table.frm`。使用`datadir` 可查看MySQL物理数据保存在哪里

### 对数据操作的粒度`Lock granularity`分类的锁
- 表锁：开销小，加锁快；不会出现死锁；锁定粒度大，发生锁冲突的概率最高，并发度最低（MyISAM 和 MEMORY 存储引擎采用的是表级锁）；
- 行锁：开销大，加锁慢；会出现死锁；锁定粒度最小，发生锁冲突的概率最低，并发度也最高（InnoDB 存储引擎既支持行级锁也支持表级锁，但默认情况下是采用行级锁）
- 页锁：开销和加锁时间界于表锁和行锁之间；会出现死锁；锁定粒度界于表锁和行锁之间，并发度一般。(BDB)
- 适用：从锁的角度来说，表级锁更适合于以查询为主，只有少量按索引条件更新数据的应用，如Web应用；而行级锁则更适合于有大量按索引条件并发更新少量不同数据，同时又有并发查询的应用，如一些在线事务处理（OLTP）系统。

### MyISAM
- 特点：空间数据索引，性能更好，表空间小
- 表锁：`DML`会锁住整张表，阻塞`DML`和`DQL`
- 缓存：只缓存索引，不缓存真实数据
- 聚簇索引上数据文件是分离的，索引保存的是数据文件的指针。
- `select count(*) from table`无需全表扫描，MyISAM内部维护了一个计数器可直接调取。
- 逻辑文件存储结构：
    + `.frm`：与表相关的元数据信息都存放在frm文件，包括表结构的定义信息等
    + `.MYD (MYData)` ：MyISAM 存储引擎专用，用于存储MyISAM 表的数据
    + `.MYI (MYIndex)`：MyISAM 存储引擎专用，用于存储MyISAM 表的索引相关信息

### InnoDB
- 特点：热备份，事务，主外键，表空间大
- 行锁：适合高并发，修改未指定索引会降级为表锁
- 缓存：不仅缓存索引还要缓存真实数据，对内存要求较高，而且内存大小对性能有决定性的影响
- 聚簇索引上文件挂在主键索引的叶子节点上，因此`InnoDB`必须要有主键，通过主键索引效率很高。
- 辅助索引：在聚簇索引之上创建的索引，其叶子节点是聚簇索引主键值，因此主键不应该过大，因为主键太大其他索引也都会很大。。辅助索引访问数据会`回表查询`(需要二次查找)如复合索引，唯一索引。
- `select count(*) from table`需要全表扫描
- - 物理文件存储结构：
    + `.frm`: 与表相关的元数据信息都存放在frm文件，包括表结构的定义信息等
    + `.ibd`: 独享表空间，每表一个`.ibd`文件
    + `.ibdata`: 共享表空间，多个表共用一个`.ibdata`文件
