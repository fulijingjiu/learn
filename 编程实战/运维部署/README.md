# 运维部署学习资料

> 笔误对照：Linxi→**Linux**，finalshel→**FinalShell**，nigex→**Nginx**。
> 路径：`/workspace/编程实战/运维部署/`

## 学习顺序

1. [01-Linux基础与跳板机](./01-Linux基础与跳板机.md) — SSH、跳板机/堡垒机、FinalShell
2. [02-Docker与Dockerfile](./02-Docker与Dockerfile.md) — 镜像/容器/卷/网络、多阶段构建、密钥通道与只读排障
3. [03-K8s与Rancher](./03-K8s与Rancher.md) — 对象链路、探针/资源、Rancher、逐层排障与入口选型时效
4. [04-Nginx与反向代理](./04-Nginx与反向代理.md) — 静态站、反代、负载均衡、证书
5. [05-部署实战技巧](./05-部署实战技巧.md) — 从本机到服务器的常见流水线
6. [06-排障与Debug](./06-排障与Debug.md) — 连不上、502/504、容器起不来、磁盘满

开源摘录见：`../开源精选/devops/`

## 面试一句话地图

| 主题 | 口述 |
|---|---|
| Linux + 跳板机 | 业务机不直接暴露公网，先登跳板机再 ProxyJump 进内网 |
| Docker | 镜像是模板，容器是运行实例；数据用卷，互通用网络 |
| Dockerfile | 合理分层/多阶段、缓存友好、小镜像；ARG/ENV 也不是构建密钥通道 |
| K8s | 声明式编排：Deployment 管副本，Service 发现，Ingress 进流量 |
| Rancher | 多集群可视化管理平台，底下还是标准 K8s |
| Nginx | 静态资源 + 反向代理 + 负载均衡 + TLS 终结 |

## 补强与操作边界

Docker 02、K8s 03 各补了完整面试口述、症状对照表与只读排障检查清单；先取证，再审批变更，不靠清卷、删 Pod 或开放特权碰运气。K8s 03 注明社区 ingress-nginx 在 2026 年 3 月退役与 Gateway API 选型背景；不要混淆其他同名近似产品。原始 devops 开源摘录保留历史语境。
