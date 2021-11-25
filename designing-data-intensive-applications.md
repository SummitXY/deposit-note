# Designing Data-Intensive Applications

- [Designing Data-Intensive Applications](#designing-data-intensive-applications)
  - [Data System Basic](#data-system-basic)
    - [1. One case in Twitter](#1-one-case-in-twitter)
    - [2. 应对负载增加的方法](#2-应对负载增加的方法)
    - [3. Data Model](#3-data-model)
    - [4. 声明式语言 vs 命令式语言](#4-声明式语言-vs-命令式语言)
    - [MapReduce Query](#mapreduce-query)
  - [References](#references)

## Data System Basic

现代软件系统大部分是`data-intensive`而不是`compute-intensive`,
而`data-intensive system`有三个重要的特性：Reliability, Scalability, Maintainability

### 1. One case in Twitter

考虑一个推特核心场景：
1. 发推文：平均 4.6k请求/秒，峰值超过 12k请求/秒
2. 看时间线：300k请求/秒 (用户关注的所有博主的按时间倒序发布的文章)

数据结构上有两种实现方式：
> 数据结构可参考localhost : designing_data_intensive_app

**The first solution**
![](img/fig1-2.png)

The query for reading timeline:
```sql
    SELECT tweets.sender_id, tweets.text, tweets.`timestamp`
    FROM users
    JOIN follows ON users.id = follows.follower_id
    JOIN tweets ON follows.followee_id = tweets.sender_id
    WHERE users.screen_name = "Bob"
    ORDER BY tweets.`timestamp` DESC
```

By using this way, the QPS of writing into table `tweets` is 4.6k(the highest is 12k), and the QPS of reading from table`users follows tweets` is 300k. And the delay time is high.

**Another solution**
![](img/fig1-3.png)

这个实现是在「读」timeline的时候加个cache(比如redis)，当一个用户发推时，会更新每个「followers」的时间线，这样会大大缩短「读」耗时

But the QPS of writing into cache would be very high. Every user has 75 followers in average, so the QPS would be `4.6k x 75 = 345k`.

**Easter Eggs**

最终推特采取了二合一的策略：
1. 普通用户发推，会采用method 2，实时更新其关注者的timeline
2. 大V(拥有很多粉丝)发推，采用method 1, 不实时更新cache，而是等着followers查看自己的timeline时再请求DataBase

### 2. 应对负载增加的方法

1. 垂直伸缩 Vertical Scaling 使用性能更强大的机器
2. 水平伸缩 Horizontal Scaling 将负载分摊在多个小机器上

### 3. Data Model

面向关系的数据库：Mysql PostgreSql

面向文档的数据库：[MongoDB](mongodb.md) RethinkDB CouchDB 使用JSON来表示存储结构

**Sql VS NoSql(文档数据库)**
1. 文档数据库更灵活，可以随意更新“列”字段


### 4. 声明式语言 vs 命令式语言

在特定场景下，声明式语言比命令式语言有很大的优势，比如Web:

```html
<ul>
    <li class="selected">
        <p>Sharks</p>
        <ul>
            <li>Great White Shark</li>
            <li>Tiger Shark</li>
            <li>Hammerhead Shark</li>
        </ul>
    </li>
    <li><p>Whales</p>
        <ul>
            <li>Blue Whale</li>
            <li>Humpback Whale</li>
            <li>Fin Whale</li>
        </ul>
    </li>
</ul>

```

如果像让选中的列表有一个蓝色的背景，可以用`CSS`这么写：
```css
li.selected > p {
	background-color: blue;
}

```

而如果使用`Javascript`来写则是这样的：
```js
var liElements = document.getElementsByTagName("li");
for (var i = 0; i < liElements.length; i++) {
    if (liElements[i].className === "selected") {
        var children = liElements[i].childNodes;
        for (var j = 0; j < children.length; j++) {
            var child = children[j];
            if (child.nodeType === Node.ELEMENT_NODE && child.tagName === "P") {
                child.setAttribute("style", "background-color: blue");
            }
        }
    }
}

```

Meanwhile, there are other advantages belowing:
1. 调用者只需要写出想要的数据类型、组合方式等信息，隐藏具体实现，方便后续在不更改接口的前提的情况下优化


### MapReduce Query

MapReduce is a coding model(编程模型), which process huge amount of data in multi machines

比如使用`MapReduce`来查询`mongodb`数据：

For example, two `documents`:
```json
    {
  observationTimestamp: Date.parse(  "Mon, 25 Dec 1995 12:34:56 GMT"),
  family: "Sharks",
  species: "Carcharodon carcharias",
  numAnimals: 3
}
{
  observationTimestamp: Date.parse("Tue, 12 Dec 1995 16:17:18 GMT"),
  family: "Sharks",
  species:    "Carcharias taurus",
  numAnimals: 4
}

```

使用JS的mapreduce方法来获得每个月观察到的鲨鱼的数量：
```js
db.observations.mapReduce(function map() {
        var year = this.observationTimestamp.getFullYear();
        var month = this.observationTimestamp.getMonth() + 1;
        emit(year + "-" + month, this.numAnimals);
    },
    function reduce(key, values) {
        return Array.sum(values);
    },
    {
        query: {
          family: "Sharks"
        },
        out: "monthlySharkReport"
    });

```

The specific steps:
1. 通过`query`来筛符合条件的`document`
2. 每个`document`跑一遍`map()`，上述两个document跑完得到的结果`emit("1995-12",3)`和`emit("1995-12",4)`
3. 到`reduce`这一步，参数`key`为`"1995-12"`，`values`为`[3,4]`，输出到document`monthlySharkReport`=7

## References
1. https://github.com/Vonng/ddia