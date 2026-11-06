# 01 Linux 基础与跳板机 / FinalShell

## 面试口述（30 秒）

Linux 服务器靠 SSH 远程管理。生产环境业务机通常不直接暴露公网：先登录**跳板机（堡垒机）**，再从跳板机进内网。Windows/macOS 可用 **FinalShell** 做多会话、文件上传、隧道。

---

## 可操作清单

### 1. 常用命令（先熟这批）

```bash
# 身份与路径
whoami; pwd; ls -lah; df -h; free -h; uptime

# 进程与端口
ps aux | grep nginx
ss -lntp          # 或 netstat -lntp
top / htop

# 日志
tail -n 200 -f /var/log/syslog
journalctl -u nginx -n 100 --no-pager

# 权限（慎用 chmod 777）
chmod 600 ~/.ssh/id_ed25519
chown -R app:app /opt/app
```

### 2. SSH 基础（勿贴真实密钥）

```bash
# 生成密钥（示例算法）
ssh-keygen -t ed25519 -C "yourname@example.com" -f ~/.ssh/id_ed25519

# 公钥放到服务器（或由运维配）
ssh-copy-id -i ~/.ssh/id_ed25519.pub user@jump.example.com

# 直连
ssh -i ~/.ssh/id_ed25519 user@10.0.0.12
```

`~/.ssh/config` 示例（占位主机名）：

```sshconfig
Host jump
  HostName jump.example.com
  User ops
  IdentityFile ~/.ssh/id_ed25519

Host app1
  HostName 10.0.0.12
  User app
  ProxyJump jump
  IdentityFile ~/.ssh/id_ed25519
```

然后：`ssh app1`

### 3. 跳板机 / 堡垒机是什么

| 概念 | 说明 |
|---|---|
| 跳板机 | 唯一（或少数）可从公网 SSH 的机器，再转到内网 |
| 堡垒机 | 企业级：审计、审批、录像、二次认证；功能更强 |
| 为什么用 | 缩小攻击面、统一入口、便于审计与权限回收 |

**ProxyJump（`-J`）概念**：客户端 → 跳板 → 目标，一跳完成，不必先手动登跳板再 ssh。

```bash
ssh -J ops@jump.example.com app@10.0.0.12
# 等价于 config 里的 ProxyJump
```

**安全要点（口述）**

- 禁用密码登录，只用密钥；跳板开 MFA 更好
- 业务机安全组只放行跳板 IP
- 个人私钥不进仓库、不进镜像、不截图外发

### 4. FinalShell 用法要点（工具向）

FinalShell 是国产跨平台 SSH 客户端，适合「多服务器运维」。

| 能力 | 怎么用 |
|---|---|
| 多服务器 | 左侧主机树分组；区分跳板 / 业务机 |
| 隧道 | 本地/远程端口转发：把内网 DB/Redis 映射到本机端口调试 |
| 上传下载 | 右侧文件管理或拖拽；大文件注意断点/权限 |
| 同步命令 | 可对多 Tab 发同一命令（慎用生产） |
| 编码 | 中文乱码时检查 UTF-8 / locale |

**建议工作流**

1. 在 FinalShell 里先配「跳板」连接  
2. 业务机勾选「跳板机」或填 ProxyJump 等价项  
3. 隧道只开调试期间需要的端口，用完关掉  
4. 上传脚本到 `/tmp` 或发布目录，执行前 `cat` / `bash -n` 检查  

---

## 面试加分

- 说清「跳板机 ≠ VPN」：跳板是登录中转；VPN 是网络层互通。  
- 能画出：笔记本 → 跳板 → app/db。  
- 知道 `ProxyJump` / `ProxyCommand` 比「先登再登」更规范。
