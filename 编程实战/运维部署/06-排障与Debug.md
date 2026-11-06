# 06 排障与 Debug

## 总原则

1. **先复现范围**：只你不行还是全员不行？内网还是公网？  
2. **沿链路查**：DNS → 证书 → Nginx → 上游进程/容器 → 依赖（DB/Redis）→ 磁盘/内存  
3. **看时间线**：刚发版？证书是否到期？磁盘是否满？  
4. **保留证据**：报错原文、`curl -v`、容器 `logs`、`describe`  

---

## 场景 A：SSH / FinalShell 连不上

| 检查 | 命令/动作 |
|---|---|
| 网络 | `ping` / `nc -vz host 22` |
| 安全组/防火墙 | 云控制台是否放行你的出口 IP |
| 跳板 | 先只连 jump；再 ProxyJump |
| 密钥 | 权限是否 600；是否用了错的 IdentityFile |
| 服务器 sshd | 在控制台 VNC 上看 `sshd` 是否活着 |

FinalShell：试「系统自带终端」对比；关代理；核对字符集与协议（SSH2）。

---

## 场景 B：502 / 504

| 状态 | 常见含义 |
|---|---|
| **502 Bad Gateway** | Nginx 连不上上游（进程挂了、端口错、容器没起来） |
| **504 Gateway Timeout** | 上游太慢或卡住（SQL 慢、死锁、下游超时） |

```bash
# Nginx
tail -n 100 /var/log/nginx/error.log
# 常见：connect() failed (111: Connection refused)
#       upstream timed out

curl -v http://127.0.0.1:8080/healthz   # 绕过 Nginx 直打上游
ss -lntp | grep 8080
```

---

## 场景 C：容器起不来

```bash
docker ps -a
docker logs --tail=200 myapp
docker inspect myapp --format '{{.State.Status}} {{.State.Error}}'

# 常见原因
# - 端口占用
# - 环境变量缺失
# - 镜像架构不对（arm/amd）
# - 健康检查过严 → 一直重启
# - 卷权限导致写失败
```

K8s：

```bash
kubectl describe pod <name>    # Events 最关键
kubectl logs <name> -c <container>
kubectl get events --sort-by=.lastTimestamp
```

看 `ImagePullBackOff` / `CrashLoopBackOff` / `OOMKilled`。

---

## 场景 D：磁盘满

```bash
df -h
du -xh /var | sort -h | tail -20
docker system df
docker system prune -f          # 慎用，先确认
journalctl --disk-usage
```

日志炸弹、未清理镜像层、容器可写层打满 → 先腾空间再谈业务恢复。

---

## 日志在哪（速查）

| 组件 | 常见位置 |
|---|---|
| Nginx | `/var/log/nginx/access.log` `error.log` |
| systemd 服务 | `journalctl -u <service>` |
| Docker 容器 | `docker logs`；json-file 驱动在 `/var/lib/docker/containers/...` |
| K8s | `kubectl logs`；集中式则 ELK/Loki |
| 应用自管 | `/var/log/app` 或挂载卷 |

---

## 可操作排障清单（贴到工单模板）

- [ ] 时间点与变更（发版？配置？证书？）  
- [ ] `curl -v` 结果（状态码/耗时）  
- [ ] Nginx error 最后 50 行  
- [ ] 上游 `ss` + 进程/容器状态  
- [ ] `df -h` 与负载 `uptime`  
- [ ] 是否已回滚 / 临时缓解措施  

## 面试加分

说出「502 是网关侧连不上，504 是上游超时」；能按链路逐层排除，而不是盲目重启。
