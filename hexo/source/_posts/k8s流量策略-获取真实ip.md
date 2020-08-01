---
title: k8s流量策略-获取真实ip
date: 2020-07-11 14:24:57
tags:
- k8s
categories: k8s
---

## externalTrafficPolicy 简介

如果服务需要将外部流量路由到 本地节点或者集群级别的端点，即service type 为LoadBalancer或NodePort，那么需要指明该参数。存在两种选项：”Cluster”（默认）和 “Local”。 “Cluster” 隐藏源 IP 地址，可能会导致第二跳（second hop）到其他节点，但是全局负载效果较好。”Local” 保留客户端源 IP 地址，避免 LoadBalancer 和 NodePort 类型服务的第二跳，但是可能会导致负载不平衡。

在实际的业务中，诸多业务是需要保留客户端源 IP，所以需要通过将服务的配置文件中的 externalTrafficPolicy 参数设置为 “Local” 来激活这个特性。

```
    {
      "kind": "Service",
      "apiVersion": "v1",
      "metadata": {
        "name": "example-service",
      },
      "spec": {
        "ports": [{
          "port": 8765,
          "targetPort": 9376
        }],
        "selector": {
          "app": "example"
        },
        "type": "LoadBalancer",
        "externalTrafficPolicy": "Local"
      }
    }
```

<!--more-->

### 使用保留源 IP 的警告和限制

新功能中，外部的流量不会按照 pod 平均分配，而是在节点（node）层面平均分配（因为 GCE/AWS 和其他外部负载均衡实现没有能力做节点权重， 而是平均地分配给所有目标节点，忽略每个节点上所拥有的 pod 数量）。

然而，在 pod 数量（NumServicePods） « 节点数（NumNodes）或者 pod 数量（NumServicePods） » 节点数（NumNodes）的情况下，即使没有权重策略，我们也可以看到非常接近公平分发的场景。

内部 pod 对 pod 的流量应该与 ClusterIP 服务类似，流量对于所有 pod 是均分的。

### ps

设置了externalTrafficPolicy:Local以后svc死活都不能访问，后来经过一系列排查iptables和kube-proxy终于发现了解决办法。
在kube-proxy启动参数里面需要设置--hostname-override：

```
- --hostname-override=$(NODE_NAME)
        env:
        - name: NODE_NAME
          valueFrom:
            fieldRef:
              apiVersion: v1
              fieldPath: spec.nodeName
```

## 通过podAntiAffinity 避免pod 流量不均衡

竟然外部的流量不会按照 pod 平均分配，而是在节点（node）层面平均分配 ,那么我们能做的只有保证同一业务的pod调度到不同的node节点上。

podAntiAffinity使用场景：

- 将一个服务的POD分散在不同的主机或者拓扑域中，提高服务本身的稳定性。
- 给POD对于一个节点的独占访问权限来保证资源隔离，保证不会有其它pod来分享节点资源。
- 把可能会相互影响的服务的POD分散在不同的主机上

对于亲和性和反亲和性，每种都有三种规则可以设置：

- RequiredDuringSchedulingRequiredDuringExecution ：在调度期间要求满足亲和性或者反亲和性规则，如果不能满足规则，则POD不能被调度到对应的主机上。在之后的运行过程中，如果因为某些原因（比如修改label）导致规则不能满足，系统会尝试把POD从主机上删除（现在版本还不支持）。
- RequiredDuringSchedulingIgnoredDuringExecution ：在调度期间要求满足亲和性或者反亲和性规则，如果不能满足规则，则POD不能被调度到对应的主机上。在之后的运行过程中，系统不会再检查这些规则是否满足。
- PreferredDuringSchedulingIgnoredDuringExecution ：在调度期间尽量满足亲和性或者反亲和性规则，如果不能满足规则，POD也有可能被调度到对应的主机上。在之后的运行过程中，系统不会再检查这些规则是否满足。

那我们的使用场景只能使用RequiredDuringSchedulingIgnoredDuringExecution，要严格保证同一业务pod调度到不同的主机。当然这样可能出现一种问题：**不满足条件的pod，会pending**。这个时候我们运维会接受到通知，去增加node节点或是驱赶业务不重要的pod。

### 示例解读

```
apiVersion: v1
kind: Pod
metadata:
  name: with-pod-affinity
spec:
  affinity:
    podAntiAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        podAffinityTerm:
          labelSelector:
            matchExpressions:
            - key: name
              operator: In
              values:
              - frontend
          topologyKey: kubernetes.io/hostname
  containers:
  - name: with-pod-affinity
    image: gcr.io/google_containers/pause:2.0
```

使用kubernetes.io/hostname作为拓扑域,查看匹配规则，即同一打有同样标签name=frontend的pod会调度到不同的节点。

### 亲和性/反亲和性调度策略比较

| 调度策略        | 匹配标签 |                 操作符                  | 拓扑域支持 |         调度目标         |
| --------------- | -------: | :-------------------------------------: | ---------: | :----------------------: |
| nodeAffinity    |     主机 | In, NotIn, Exists, DoesNotExist, Gt, Lt |         否 |      pod到指定主机       |
| podAffinity     |      Pod |     In, NotIn, Exists, DoesNotExist     |         是 |  pod与指定pod同一拓扑域  |
| PodAntiAffinity |      Pod |     In, NotIn, Exists, DoesNotExist     |         是 | pod与指定pod非同一拓扑域 |