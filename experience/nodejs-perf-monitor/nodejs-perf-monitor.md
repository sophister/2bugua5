# Node.js线上监控


## 监控指标

目前监控代码，是 **直接运行在node业务进程** 里，定时将统计到的相关数据，输出到日志里，因此，主要统计的是 **单个进程** 的数据，主要包含以下字段：

* 内存占用
* `v8` 堆使用
* cpu使用
* `eventloop` 延迟
* 当前并发连接数
* `gc`情况

## 具体统计数据

1. 内存使用

从  `process.memoryUsage()` 拿到当前进程的内存、堆使用数据：`rss` `heapTotal` `heapUsed`

从 `v8.getHeapStatistics()` 获取 `v8` 堆使用情况：`total_heap_size` `used_heap_size` `heap_size_limit`

2. cpu使用

`os.loadavg()` 读取CPU负载

3. `eventloop`延迟

使用 [loopbench](https://github.com/mcollina/loopbench) 测试事件循环的延迟

4. 并发连接数

可以直接从  `server.getConnections()` 获取。


## 相关资料

* [node服务的监控预警系统架构](http://web.jobbole.com/89739/)
* [server.getConnections](https://nodejs.org/dist/latest-v10.x/docs/api/net.html#net_server_getconnections_callback)
* [ELK配置性能日志可视化](https://deors.wordpress.com/2017/03/24/build-your-own-application-performance-monitoring-apm-solution-with-elk-and-jmx-part-2/)
