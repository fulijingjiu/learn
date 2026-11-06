# 03 Kubernetes 与 Rancher

## 面试口述（40 秒）

K8s 用**声明式 YAML**管容器集群：你说「要 3 个副本」，控制器去对齐现状。核心对象：

- **Pod**：最小调度单元，一组共享网络/存储的容器  
- **Deployment**：管无状态应用的副本与滚动更新  
- **Service**：稳定虚拟 IP / DNS，把流量转到 Pod  
- **Ingress**：七层入口（域名/路径 → Service），常配 Nginx Ingress  

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

Ingress 概念：把 `app.example.com/` 转到 `Service/web:80`；TLS 可在 Ingress 终结。

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
