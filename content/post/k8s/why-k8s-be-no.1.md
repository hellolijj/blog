---
title: "带着问题去学习系列：《深入剖析kuberntes》"
date: 2019-04-25T13:40:47+08:00
draft: false
---

原文：[https://time.geekbang.org/column/article/14252](https://time.geekbang.org/column/article/14252)

## 问题一：为什么kuberntes能打败docker swarm、mesos成为容器编排工具事实标准？

回答这个问题，我们就需要了解 kubernetes、docker swarm、mesos 三种容器编排工具的历史及优缺点。

### docker swarm

docker swarm 是docker公司推出的轻量级容器编排工具。docker-swarm 的使用依赖于docker。

docker swarm 原理：
发送给docker的命令被docker swarm拦截下来，通过具体的调度算法找到合适的 Docker Daemon运行起来

总结下它的优点：

- 轻量级，启动快，操作快

- docker 内置 无需额外安装

缺点

- 只能使用docker容器

- 监控方案只有docker status，比较低级

> 参考文献：[Kubernetes vs. Docker Swarm：完整的比较指南](https://cloud.tencent.com/developer/article/1361203)

### docker-compose


说到docker swarm 就不得不说 docker-compose，docker-compose也会一个docker的编排工具。docker-compose 基于python编写。使用`pip install docker-compose` 就可以安装。

docker-compose 之前叫Fig项目，Fig项目只是两个全职开发和维护的。Fig 项目首次提出了容器编排概念。

> 编排：用户通过工具或者配置来完成一组虚拟机及相关关联资源的定义、配置、创建、删除工作，然后由云计算平台按照这些指令逻辑完成相应的过程。
docker-compose编排也是基于 yaml 文件。例如，某个docker-compose样例如下：

```yaml
neo4j_api:
  image: chvb/docker-apache-php
  container_name: neo4j_front
  restart: always
  volumes:
    - ./mock/:/var/www:rw
  ports:
    - "52000:80"

neo4j_front:
  build: ./html
  container_name: neo4j_front
  restart: always
  volumes:
    - ./html:/usr/src/node:rw
  ports:
    - "52001:80"
```

它的优缺点同 docker swarm

### Mesos

mesos 是 mesosphere公司的一款调度工具。Mesos也是Apache的一个项目，它使您能够以分布式方式运行容器化和非容器化工作负载。它既可以对容器进行编排也可以编排非容器（虚拟机）对应用。对于容器编排，Mesos 推出了一种极好的Marathon“插件”。

2016年中期，Mesosphere 推出了支持的开源项目DC / OS（数据中心操作系统），它进一步简化了Mesos，并支持在几分钟内部署自己的Mesos集群，使用Marathon。

DC / OS更类似于操作系统而不是编排框架。可以在其上运行非容器化的有状态工作负载。集装箱调度由Marathon处理。

所以与 kuberntes 对等的平台就变成了 DC / OS 中的 Marathon. 

与Kubernetes相比，Marathon对API的一般方法很简单。Marathon聚合API并提供相对少量的API资源，而Kubernetes提供更多种类的资源并基于标签选择器。 但是 Marathon 的社区规模没有kubernetes大。

总结：Mesos的优点

- Mesosphere 是企业级应用，更加成熟

Mesos的缺点

- 相对与kuberntes来说笨重
- 开源的Marathon关注人数不多，没有kubernetes 火热

### kubernetes

#### kubernetes 的特性

Kubernetes 满足了生产中运行应用程序的许多常见的需求。例如

- Pod提供了一种复合的应用容器模型

- 挂载外部存储

- Secret管理

- 应用健康检查

- 副本应用实例

- 横向自动扩缩容

- 服务发现

- 负载均衡

- 滚动更新

- 资源监测

- 日志采集和存储

- 支持自检和调试

- 认证和鉴权

#### kubernetes 跟其他容器编排工具的不同

- K8S对容器又做了一层抽象，提出了pod这个概念。

  - kubernetes对容器的所有操作，如 增删查改，包括对容器对监控，动态伸缩，实际上都是对 pod 操作的。

  - 关于kuberntes中pod的概念，参考 kubernetes 为什么抽象 pod 这个概念。

#### kubernetes 提出了声明式api

- 声明式api 与 命令式api 有何异同？

  - 比如要我需要集群中跑10个web服务容器，传统的命令式API是一步步调用命令构建出容器。而使用声明式api，只要告诉K8S我要10个web容器，K8S就会自动将web集群实例数维持在10个，并且，在某个pod出问题退出时，K8S会自动重新拉起新pod，使集群始终保持10个pod实例在跑。这就使得管理集群变得很简单，只要通过配置文件描述出希望的集群状态，而不用去关注中间的实现过程。

  
> 参考博客：[Kubernetes vs. Mesos：选择容器编排工具](https://cloud.tencent.com/developer/article/1367005)

#### kubernetes 提供了Api、便于与其他厂商的合作

kubernetes 之所有能火热起来成为容器编排工具的事实标准，除了它本身提出了其他工具没有的特性之外。还在于它采用社区推动的方式，保证了各方 的利益。

kubernetes 是谷歌贡献给开源社区的。并且还联合其他云计算厂商一起成立CNCF社区。社区以开放的态度吸引了其他云计算厂商和众多开发者。

采用接耦的方法，将功能特性抽象出来，以接口的形式，以便其他开发者定制化开发。例如kubernetes 的CSI、CNI、CRI等接口。

#### 社区化的运营方式  