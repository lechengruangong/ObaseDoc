在java平台的Obase,使用的是HikariCP连接池来增强性能.

Obase连接池内连接的统计信息可以通过如下的方式获取,这些信息可以帮助调用方分析连接池的状态:

可以通过ObaseConnectionPool的方法getInstance获取到当前应用程序域的连接池,返回值上有getStatistics和getFullStatistics方法提供简要和完整的分析.

调用代码如下:

```
//获取当前连接池的信息
String statistics = ObaseConnectionPool.getInstance().getStatistics();
//获取当前连接池的完整信息
String statisticsFully = ObaseConnectionPool.getInstance().getFullStatistics();
```

典型的简要信息输出如下:
```
Obase ConnectionPool / totalConnections:1, activeConnections:0, idleConnections:1, awaitingConnections:0
```
其中每个部分的含义是:

- Obase ConnectionPool 连接池的名称.
- totalConnections:1 池内可用连接数.
- activeConnections:0 激活的连接数.
- idleConnections:1 空闲的连接数.
- awaitingConnections:0 等待的连接数.

典型的完整信息输出如下:
```
Obase ConnectionPool / maxPoolSize:10, connectionTimeout:30000 ms, idleTimeout:600000 ms, maxLifetime:1800000 ms / totalConnections:1, activeConnections:0, idleConnections:1, awaitingConnections:0
```
其中每个部分的含义是:

- Obase ConnectionPool 连接池的名称.
- maxPoolSize:10 最大池大小.
- connectionTimeout:30000 ms 连接超时时间.
- idleTimeout:600000 ms 空闲超时时间.
- maxLifetime:1800000 ms 连接最大的生命周期.

其余与简要信息相同.