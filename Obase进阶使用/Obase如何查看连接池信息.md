Obase连接池内连接的统计信息可以通过如下的方式获取,这些信息可以帮助调用方分析连接池的状态:

## dotNet

可以通过ObaseConnectionPool的属性访问器Current获取到当前应用程序域的连接池,此属性上有Statistics和StatisticsFully属性访问器提供简要和完整的分析.

调用代码如下:

```
//获取当前连接池的信息
var statistics = ObaseConnectionPool.Current.Statistics;
//获取当前连接池的完整信息
var statisticsFully = ObaseConnectionPool.Current.StatisticsFully;
```

典型的简要信息输出如下:
```
MySql ConnectionPool / Pool: 5/5, Get wait: 0, GetAsync wait: 0
```
其中每个部分的含义是:

- MySql ConnectionPool 连接池的名称
- Pool: 5/5 池内可用连接数/当前池大小
- Get wait: 0:同步获取等待队列长度
- GetAsync wait: 0:异步取等待队列长度

典型的完整信息输出如下:
```
MySql ConnectionPool / Pool: 5/5, Get wait: 0, GetAsync wait: 0

MySql.Data.MySqlClient.MySqlConnection, Times: 1, ThreadId(R/G): 17/17, Time(R/G): 2025-07-15 14:50:06:506/2025-07-15 14:50:05:505, 
MySql.Data.MySqlClient.MySqlConnection, Times: 1, ThreadId(R/G): 17/19, Time(R/G): 2025-07-15 14:50:06:506/2025-07-15 14:50:06:506, 
MySql.Data.MySqlClient.MySqlConnection, Times: 1, ThreadId(R/G): 17/8, Time(R/G): 2025-07-15 14:50:06:506/2025-07-15 14:50:06:506, 
MySql.Data.MySqlClient.MySqlConnection, Times: 493, ThreadId(R/G): 17/17, Time(R/G): 2025-07-15 14:52:38:5238/2025-07-15 14:52:38:5238, 
MySql.Data.MySqlClient.MySqlConnection, Times: 1, ThreadId(R/G): 17/20, Time(R/G): 2025-07-15 14:50:06:506/2025-07-15 14:50:06:506, 
```
其中每个部分的含义是:

第一行与简要分析相同,从第二行开始每行为一个连接的具体信息.

- MySql.Data.MySqlClient.MySqlConnection: 连接对象的类型.
- Times: 1:从池中被获取过的次数.
- ThreadId(R/G): 17/17:最后归还此连接的线程ID/最后获取此连接的线程ID.
- Time(R/G): 2025-07-15 14:50:06:506/2025-07-15 14:50:05:505:最后归还此连接的时间/最后获取此连接的时间.

## Java

java版待补充