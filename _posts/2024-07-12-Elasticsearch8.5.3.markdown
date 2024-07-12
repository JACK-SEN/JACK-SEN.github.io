---
layout: post
title:  "Elasticsearch8.5.3"
datetime:   2024-01-08 10:27:26
categories: Elasticsearch8.5.3
---
# Elasticsearch8.5.3

# 配置文件修改

**elasticsearch.yml**

```properties
cluster.name: my-application
node.name: node-1
path.data: E:\\es\\elasticsearch-8.5.3\\data
path.logs: E:\\es\\elasticsearch-8.5.3\\logs
bootstrap.memory_lock: true
network.host: 0.0.0.0
http.port: 9200
#discovery.seed_hosts: ["host1", "host2"]
cluster.initial_master_nodes: ["node-1"]
#readiness.port: 9399
#action.destructive_requires_name: false
xpack.security.enabled: false
xpack.security.enrollment.enabled: true
http.cors.enabled: true
http.cors.allow-origin: "*"
ingest.geoip.downloader.enabled: false
```

**jvm.options**

```properties
-Dfile.encoding=GBK
```

# 启动

**windows**

运行elasticsearch.bat

**linux**

运行elasticsearch



# 开发环境

jdk版本 >=11

springboot版本 >=2.6

# SpringBoot集成

### 配置解释

1. **spring.elasticsearch.uris**
   - 用途：设置Elasticsearch节点的URI。
   - 示例值：`http://127.0.0.1:9200`
   - 推荐设置：根据实际部署情况设置，可以是单个节点或多个节点，多个节点用逗号分隔。
2. **spring.elasticsearch.username**
   - 用途：设置Elasticsearch的用户名。
   - 示例值：`your_username`
   - 推荐设置：根据实际用户名配置。
3. **spring.elasticsearch.password**
   - 用途：设置Elasticsearch的密码。
   - 示例值：`your_password`
   - 推荐设置：根据实际密码配置。
4. **spring.elasticsearch.connection-timeout**
   - 用途：设置与Elasticsearch建立连接的超时时间。
   - 示例值：`15s`
   - 推荐设置：默认设置为15秒，根据网络延迟和服务器响应情况调整。
5. **spring.elasticsearch.socket-timeout**
   - 用途：设置与Elasticsearch通信的套接字超时时间。
   - 示例值：`60s`
   - 推荐设置：默认设置为60秒，根据实际需求调整，如果查询操作比较耗时，可以适当增加。
6. **spring.elasticsearch.path-prefix**
   - 用途：设置Elasticsearch的路径前缀。
   - 示例值：``
   - 推荐设置：默认值为空，如果Elasticsearch配置了路径前缀，则需要相应配置。
7. **spring.elasticsearch.restclient.sniffer.delay-after-failure**
   - 用途：设置RestClient失败后的延迟时间。
   - 示例值：`1m`
   - 推荐设置：默认设置为1分钟，根据实际需求调整。
8. **spring.elasticsearch.restclient.sniffer.interval**
   - 用途：设置RestClient嗅探的间隔时间。
   - 示例值：`5m`
   - 推荐设置：默认设置为5分钟，根据集群节点变化频率调整。
9. **spring.elasticsearch.webclient.max-in-memory-size**
   - 用途：设置WebClient最大内存大小。
   - 示例值：``
   - 推荐设置：根据实际应用需求设置，防止内存溢出。例如，`10MB`。

### 推荐设置示例

以下是推荐的一组配置，根据实际情况可以进行调整：

```
spring.elasticsearch.uris=http://127.0.0.1:9200,http://127.0.0.2:9200
spring.elasticsearch.username=your_username
spring.elasticsearch.password=your_password
spring.elasticsearch.connection-timeout=10s
spring.elasticsearch.socket-timeout=60s
spring.elasticsearch.path-prefix=/elasticsearch
spring.elasticsearch.restclient.sniffer.delay-after-failure=1m
spring.elasticsearch.restclient.sniffer.interval=5m
spring.elasticsearch.webclient.max-in-memory-size=10MB
```

### 配置优化说明

1. **Elasticsearch URI**
   - 如果有多个Elasticsearch节点，建议配置多个URI，提升高可用性。
2. **连接和套接字超时**
   - 根据网络环境和查询复杂度调整连接和套接字超时时间。
3. **RestClient嗅探**
   - 调整嗅探间隔和失败后的延迟时间，以便及时更新节点状态。
4. **WebClient内存**
   - 根据实际应用场景设置WebClient的最大内存大小，避免大数据量查询时出现内存溢出。

通过以上配置，可以更好地管理Elasticsearch连接，提升应用的稳定性和性能。
