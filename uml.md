## 类图
|    关系   |   A    |  线条  |     B     |            代码具现           |
|-----------|--------|--------|-----------|-------------------------------|
| 继承 泛化 | 子类   | `-->`  | 父类      | A继承B                        |
| 实现      | 实现类 | `..>`  | 接口      | A实现B                        |
| 组合      | 部分类 | `<--*` | 整体类    | A用成员变量保存B的单/多个实例 |
| 聚合      | 部分类 | `<--o` | 整体类    | A用成员变量保存B的单/多个实例 |
| 关联      | 引用类 | `-->`  | 被引用类  | A用成员变量保存B的单/多个实例 |
| 依赖      | use类  | `..>`  | be used类 | A的方法使用B的静态方法/实例   |

- 类图是结构图中的一种，用于描述一个系统的静态结构。
- 类图以反映类结构和类之间关系为目的。
- 类图六种关系由强到弱：继承→实现→组合→聚合→关联→依赖

### 依赖
- 假设B类的变化引起了A类的变化，则说明A类依赖于B类。可视为`use-a`的关系
- 依赖关系是具有偶然性的、临时性的、非常弱的。
- 生命周期：若A类依赖B类，那么只有当A类对象调用到相应方法时，B类对象才被临时创建，方法执行结束，B类对象即被回收(临时性)

### 关联
- 关联具有方向：单向关联，双向关联
- 关联的多重性：一对一关联，一对多关联，//todo::映射关系表示
- `关联`相对于`依赖`表现出一种`长期性`。
- 通常用成员变量保存实例，但并非绝对，如当根据`接口`创建类时
，可表示为 `返回B的方法`
- 组合和聚合都是一种特殊的关联关系，其强调的是`整体`与`部分`，而关联强调的是`对等`

### 聚合
- 聚合是关联关系的一种特例，他体现的是`整体`拥有`部分`的关系，即`has-a`的关系。
- 整体与部分之间是可分离的，部分可以属于多个整体对象，也可以为多个整体对象共享
- 生命周期：整件不会拥有部件的生命周期，所以整件删除时，部件不会被删除。
- 信息的封装性：客户端可以同时实例`整体类`和`部分类`，因为他们都是独立的。
- 代码：通常以构造方法参数对`部分`属性进行对象传递
- 聚合代码层面的表现和关联关系是一致的，只能从语义级别来区分。
    + 关联的对象间是平等的：你是我的朋友。
    + 聚合则一般是不平等的：计算机与CPU；公司与员工。

### 组合
- 组合也是关联关系的一种特例，他体现的是`部分`组成`整体`的关系，即`contains-a`的关系，这种关系比聚合更强，也称为强聚合。
- 整体与部分不可分离，多个整件不可以同时间共享同一个部件
- 生命周期：整体拥有部件的生命周期，整体删除时，部件也会被删除
- 信息的封装性：客户端只可实例化`整体类`，`部分类`在客户端中不可见
- 代码：在内部实例化`部分`对象并赋值给属性
- 组合和聚合的使用要视`问题域`而定，例如在关心汽车的领域里，轮胎是一定要组合在汽车类中的，因为它离开了汽车就没有意义了。但是在卖轮胎的店铺业务里，就算轮胎离开了汽车，它也是有意义的，这就可以用聚合了。



## ER图
```
@startuml
hide circle
skinparam linetype ortho

entity "Entity01" as e01 {
  *e1_id : number <<generated>>
  --
  *name : text
  description : text
}

entity "Entity02" as e02 {
  *e2_id : number <<generated>>
  --
  *e1_id : number <<FK>>
  other_details : text
}

entity "Entity03" as e03 {
  *e3_id : number <<generated>>
  --
  e1_id : number <<FK>>
  other_details : text
}

e01 ||..o{ e02
e01 |o..o{ e03
@enduml
```

### 关键字
- `'autonumber 10 1 "<b>[000]"` 自动序号
- `hide circle` 隐藏标题中多余的图标
- `hide footbox` 移除脚注
- `skinparam linetype ortho` 使用直角线条
- `left to right direction` 从左至右的树型结构
- `*` 候选码
- `**` 加粗
- `--` 分隔符
- `<FK>` 外键描述


### 关联关系
- (O)ption 可选参与
- (M)andatory 强制参与
- `|o--` O 0..1
- `||--` M 1
- `}|--` M 1..n
- `}o--` O 0..n

#### 例子: 共有10种可能
- `人 |o--|{ 车`
- `人 ||--|| 手`
- `人 |o--|| 身份证`



### 属性
identifying_attribute
mandatory_attribute
optional_attribut


改动内容文本输出



