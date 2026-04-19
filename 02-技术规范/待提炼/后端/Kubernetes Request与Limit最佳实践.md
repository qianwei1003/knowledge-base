---
来源: imroc.cc Kubernetes实践指南
领域: 后端
关键词: Kubernetes, Request, Limit, 资源配额, 容器调度, 生产环境
质量: ⭐⭐⭐⭐⭐
提炼状态: 待提炼
原始链接: https://imroc.cc/kubernetes/best-practices/request-limit
---

# Kubernetes Request 与 Limit 最佳实践

## 所有容器都应该设置 request

request 的值并不是指给容器实际分配的资源大小，它仅仅是给调度器看的。调度器会"观察"每个节点可以用于分配的资源有多少，也知道每个节点已经被分配了多少资源。

被分配资源的大小就是节点上所有 Pod 中定义的容器 request 之和，它可以计算出节点剩余多少资源可以被分配(可分配资源减去已分配的 request 之和)。

如果发现节点剩余可分配资源大小比当前要被调度的 Pod 的 request 还小，那么就不会考虑调度到这个节点，反之，才可能调度。

**建议**：给所有容器都设置 request，让调度器感知节点有多少资源被分配了，以便做出合理的调度决策，让集群节点的资源能够被合理的分配使用。

## CPU request 与 limit 的一般性建议

- 如果不确定应用最佳的 CPU 限制，**可以不设置 CPU limit**

- 如果要设置 CPU request，大多可以设置到不大于 1 核，除非是 CPU 密集型应用

## 老是忘记设置怎么办？

可以使用 **LimitRange** 来设置 namespace 的默认 request 与 limit 值，同时它也可以用来限制最小和最大的 request 与 limit。

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: mem-limit-range
  namespace: test
spec:
  limits:
  - default: # 此处定义默认限制值
    memory: 512Mi
    cpu: 500m
    defaultRequest: # 此处定义默认请求值
    memory: 256Mi
    cpu: 100m
    max: # max 和 min 定义限制范围
    memory: 1Gi
    cpu: "1"
    min:
    memory: 128Mi
    cpu: 50m
    type: Container
```

## 重要的线上应用该如何设置

节点资源不足时，会触发自动驱逐，将一些低优先级的 Pod 删除掉以释放资源让节点自愈。

**优先级排序**（从低到高）：
1. 没有设置 request/limit 的 Pod — 优先级最低，容易被驱逐
2. request 不等于 limit 的 Pod — 优先级其次
3. request 等于 limit 的 Pod — 优先级较高，不容易被驱逐

**建议**：重要的线上应用，将 request 和 limit 设成一致，以避免在节点故障时被驱逐。

## 怎样设置才能提高资源利用率？

如果给应用设置较高的 request 值，而实际占用资源长期远小于它的 request 值，会导致节点整体的资源利用率较低。

**建议**：
- 对非核心、资源不长期占用的应用，可以适当减少 request 以提高资源利用率
- 如果服务支持水平扩容，单副本的 request 值一般可以设置到不大于 1 核（CPU 密集型应用除外）
- 例如 coredns，设置到 0.1 核（即 100m）即可

## 尽量避免使用过大的 request 与 limit

风险：
- 使用单副本或少量副本，给很大的 request 与 limit
- 某个副本故障对业务影响较大
- request 较大时，节点挂了难以漂移

**建议**：
- 尽量减小 request 与 limit
- 通过增加副本的方式进行水平扩容
- 让系统更加灵活可靠

## 避免测试 namespace 消耗过多资源

若生产集群有用于测试的 namespace，如果不加以限制，可能导致集群负载过高，影响生产业务。

使用 **ResourceQuota** 来限制测试 namespace 的 request 与 limit 的总大小：

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: quota-test
  namespace: test
spec:
  hard:
    requests.cpu: "1"
    requests.memory: 1Gi
    limits.cpu: "2"
    limits.memory: 2Gi
```

## FAQ

### 为什么 CPU 利用率远不到 limit 还会被 throttle？

CPU 限流是因为内核使用 CFS 调度算法，对于微突发场景，在一个 CPU 调度周期内（100ms）所占用的时间超过了 limit 还没执行完，就会强制"抢走" CPU 使用权（throttle），等待下一个周期再执行。

时间拉长一点，进程使用 CPU 所占用的时间比例却很低，监控上看不出 CPU 有突增，但实际上已被 throttle。

## 总结

| 场景 | 建议 |
|------|------|
| 所有容器 | 都设置 request |
| 重要线上应用 | request = limit |
| 提高资源利用率 | 减小 request，增加副本 |
| 测试环境 | 使用 ResourceQuota 限制总资源 |
| 忘记设置 | 使用 LimitRange 设置默认值 |
