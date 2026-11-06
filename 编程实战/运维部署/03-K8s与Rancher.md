# 03 Kubernetes 与 Rancher

## 面试口述（40 秒）

K8s 用**声明式 YAML**管容器集群：你说「要 3 个副本」，控制器去对齐现状。核心对象：

- **Pod**：最小调度单元，一组共享网络/存储的容器  
- **Deployment**：管无状态应用的副本与滚动更新  
- **Service**：稳定虚拟 IP / DNS，把流量转到 Pod  
- **Ingress**：七层入口规则（域名/路径 → Service），需要匹配的控制器实现  

**Rancher** 是多集群管理平台（UI + 权限 + 应用商店），集群底层仍是标准 Kubernetes，不是另一套编排引擎。

---

## 对象关系（心智图）

```
用户/浏览器
    ↓
Ingress (域名 /api → svc)
    ↓
Service (ClusterIP / NodePort / LoadBalancer)
    ↓
Pod × N  ← Deployment 保证副本数
```

### 最小 Deployment + Service 示意

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
spec:
  replicas: 2
  selector:
    matchLabels: { app: web }
  template:
    metadata:
      labels: { app: web }
    spec:
      containers:
        - name: web
          image: registry.example.com/web:1.0.0
          ports: [{ containerPort: 8080 }]
---
apiVersion: v1
kind: Service
metadata:
  name: web
spec:
  selector: { app: web }
  ports:
    - port: 80
      targetPort: 8080
```

Ingress 概念：把 `app.example.com/` 转到 `Service/web:80`；TLS 可在入口层终结。规则对象不会自己转发流量，必须有对应控制器；新选型同时评估 Gateway API。

### 常用 kubectl

```bash
kubectl get pods,svc,deploy -A
kubectl describe pod web-xxxx
kubectl logs -f deploy/web
kubectl exec -it deploy/web -- sh
kubectl apply -f app.yaml
kubectl rollout status deploy/web
kubectl rollout undo deploy/web
```

---

## Rancher 是什么、和 K8s 的关系

| 问题 | 答案 |
|---|---|
| Rancher 干什么 | 统一导入/创建多集群、项目权限、监控告警、应用商店、流水线 |
| 和 K8s 关系 | **管理面**；工作负载仍跑在 K8s API 上 |
| 什么时候用 | 多环境（开发/测试/生产）、多团队、不想纯命令行管集群 |
| 面试别说错 | 「Rancher 取代了 K8s」——错；「Rancher 管多个 K8s」——对 |

同类生态：Kubernetes Dashboard、Lens、OpenShift（带更多平台能力）。

---

## 可操作清单

- [ ] 能解释 Pod 为何「短命」、Service 为何「稳定」  
- [ ] 能说清滚动更新与回滚  
- [ ] 知道 ConfigMap/Secret 注入配置（Secret 仍要配合权限与加密）  
- [ ] 看过一次 Rancher UI：集群列表 → 项目 → Workload → Ingress  

开源示例：`../开源精选/devops/kubernetes-examples-guestbook-go-README.md`

## 加厚：声明式控制与数据流的职责

Deployment 控制器维护 ReplicaSet/Pod 的期望状态，调度器选节点，kubelet 驱动节点上的容器运行时。Service 用标签选择后端，EndpointSlice 描述端点；实际转发依赖集群网络/代理实现。Rancher 是管理面，不能替代这些数据路径，也不能因 UI 显示正常就断定服务可用。

旧 guestbook 摘录包含早期 ReplicationController/Docker 时代内容，理解拓扑即可。现代集群通常通过 CRI 对接 containerd 等运行时；“跑 Docker 镜像”不等于节点一定装 Docker Engine。

### 入口选型时效提醒

截至本次整理 2026-11-06（Asia/Shanghai），Kubernetes 社区 **Ingress NGINX（ingress-nginx）已按公告于 2026 年 3 月退役，后续不再发布安全修复**，不应作为新项目默认推荐。它与 F5 的 NGINX Ingress Controller 是不同项目，不能因为名字相近就互换结论。Ingress API 本身并未因此被删除，API 已冻结；新方案可评估 Gateway API 或仍受维护的控制器，迁移时验证注解、路由、超时、TLS 与业务行为，不认为可无缝替换。

## 探针与资源：别把重启当万能恢复

| 配置 | 解决的问题 | 常见误配 |
|---|---|---|
| startupProbe | 给慢启动应用留初始化窗口，成功前门控其他探针 | 启动没完成就被 liveness 反复杀死 |
| readinessProbe | 当前是否应接 Service 的正常流量 | 拿它当“失败就重启”开关 |
| livenessProbe | 是否陷入靠重启才能恢复的坏状态 | 把数据库等外部依赖短暂失败当本进程必死 |
| requests | 调度与资源分配依据 | 全不设，放置/争用难预测 |
| limits | CPU/内存资源约束 | CPU 被节流；内存超限可能 OOM，被误判代码逻辑错 |

示意：以下放入前面 `containers[0]`，前提是应用真实实现了相应 HTTP 端点；数值仅示例，需按启动耗时与负载评估，**不是完整生产配置**。

```yaml
resources:
  requests: { cpu: 100m, memory: 128Mi }
  limits: { cpu: 500m, memory: 256Mi }
startupProbe:
  httpGet: { path: /startupz, port: 8080 }
  periodSeconds: 5
  failureThreshold: 30
readinessProbe:
  httpGet: { path: /readyz, port: 8080 }
  periodSeconds: 5
livenessProbe:
  httpGet: { path: /livez, port: 8080 }
  periodSeconds: 10
```

Readiness 失败通常使端点不再被当作 ready 流量目标，不会因此自动重启；已有连接和特殊发布设置仍需理解。startup/liveness 持续失败按 kubelet 和重启策略处理。探针应轻量，不泄露内部敏感信息。

## 排障检查清单（只读，先确认 context / namespace）

```bash
kubectl config current-context
kubectl get pods,deploy,svc -n demo -o wide
kubectl describe pod web-xxxx -n demo
kubectl logs web-xxxx -n demo -c web --since=10m
kubectl logs web-xxxx -n demo -c web --previous
kubectl get events -n demo --sort-by=.metadata.creationTimestamp
kubectl get endpointslices -n demo -l kubernetes.io/service-name=web
kubectl rollout status deploy/web -n demo --timeout=30s
```

| 现象 | 优先取证 |
|---|---|
| Pending | 调度事件、requests 与节点容量、亲和/污点、PVC 是否绑定 |
| ImagePullBackOff | 镜像名/digest、仓库可达、拉取凭据权限、平台架构；不要把凭据贴日志 |
| CrashLoopBackOff | 当前/上次日志、退出码、配置、启动命令、探针、OOM；它是重启退避状态，不是根因 |
| Running 但不 Ready | readiness/startup 结果、端点实现、端口、依赖状态 |
| Service 无流量 | selector 是否匹配 label、EndpointSlice 是否有 ready 地址、targetPort、网络策略 |
| 外部 502/503 | 入口控制器状态/规则、后端端点、连接超时、应用日志；不是只看 DNS |
| 升级卡住 | rollout 状态、maxSurge/maxUnavailable、配额、探针、资源和旧副本排空 |

生产 `exec`、`apply`、`rollout undo`、删除 Pod 都不是只读取证。先确定根因与影响，遵守审批与变更流程；回滚 Deployment 只回退工作负载模板，不会自动回滚数据库或外部副作用。

## 常见坑

- containerPort 只声明容器端口，不负责自动让进程监听，也不是公网映射。
- Secret 的 base64 不是加密，仍需 RBAC、静态加密、外部密钥管理和日志脱敏。
- 多副本不等于高可用：若全在同一节点/故障域，节点坏了可能一起中断。
- 滚动发布不保证零停机：需要 readiness、容量、优雅关闭、连接排空与兼容式数据库变更。
- Rancher 权限与底层 Kubernetes 权限要对齐；不为排障临时授予无限 cluster-admin。

## 面试口述（机制 + 排障）

> Kubernetes 用控制器把实际状态持续对齐期望状态，Deployment 管副本，Service 提供稳定访问，入口规则要靠控制器执行，Rancher 是多集群管理面。探针中 readiness 决定接流量，liveness 处理进程坏状态，startup 给慢启动留时间，不能把三者混用。故障先确认集群和命名空间，再看事件、上次日志、退出原因、端点和资源。升级与回滚还要验证连接排空和数据兼容，而不是删 Pod 碰运气。

## 延伸阅读

- 已读开源：[旧 guestbook 示例](../开源精选/devops/kubernetes-examples-guestbook-go-README.md)，仅作历史拓扑入门。
- 官方核验：[探针](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/)、[Ingress NGINX 退役声明](https://kubernetes.io/blog/2026/01/29/ingress-nginx-statement/)、[入口控制器与 Gateway 建议](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/)。
