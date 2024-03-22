elastic search启动问题处理

测试版本8.12.1，

访问地址，使用https访问，使用localhost访问不行

https://127.0.0.1:9200/

输入账号密码

账号:elastic 

密码:elastic123

1、压缩包解压之后修改启动的集群和节点名称

```properties
配置顺序在前
# Use a descriptive name for your cluster:

cluster.name: ydzb-es
#
# ------------------------------------ Node ------------------------------------
#
# Use a descriptive name for the node:
#
#node.name: node-1
node.name: master
```

2、启动之后修改自定义密码

在elastic search的bin目录下执行命令，eg：E:\es\ydzb-master\bin>，根据提示输入两次密码

```properties
elasticsearch-reset-password --username elastic -i
```

3、

