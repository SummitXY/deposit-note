# Designing Data-Intensive Applications

- [Designing Data-Intensive Applications](#designing-data-intensive-applications)
  - [Data System Basic](#data-system-basic)
    - [One case in Twitter](#one-case-in-twitter)
  - [References](#references)

## Data System Basic

现代软件系统大部分是`data-intensive`而不是`compute-intensive`,
而`data-intensive system`有三个重要的特性：Reliability, Scalability, Maintainability

### One case in Twitter

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

## References
1. https://github.com/Vonng/ddia