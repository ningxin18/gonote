grpc

bingo

docker

云计算公司

## **etcd介绍与使用**

概念：高可用的分布式key-value存储，可以用于配置共享和服务发现。 
类似项目：zookeeper和consul 
开发语言：Go 
接口：提供restful的http接口，使用简单 
实现算法：基于raft算法的强一致性、高可用的服务存储目录

应用场景：

```
服务发现和服务注册
配置中心
分布式锁
master选举
```

## etcd搭建 

a. 下载etcd release版本：<https://github.com/coreos/etcd/releases/> 
b. ./bin/etcd即可以启动etcd 
c. 使用etcdctl工具更改配置

https://github.com/coreos/etcd/releases/download/v3.3.0/etcd-v3.3.0-linux-amd64.tar.gz

```go
[root@greg02 etcd3.3]#ls
default.etcd  Documentation  etcd  etcdctl  README-etcdctl.md  README.md  READMEv2-etcdctl.md
[root@greg02 etcd3.3]#etcdctl set test asdf
asdf
[root@greg02 etcd3.3]#etcdctl get test
asdf
```

context使用介绍

如何控制goroutine的超时

如何保存上下文数据

使用context处理超时：

```go
ctx, cancel := context.WithCancel(context.Background(),2*time.Second)
```

