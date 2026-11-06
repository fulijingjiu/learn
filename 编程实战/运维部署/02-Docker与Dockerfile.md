# 02 Docker 与 Dockerfile

## 面试口述（30 秒）

**镜像**是只读模板，**容器**是镜像的运行实例。数据持久化用**卷**，容器互通用**网络**。Dockerfile 描述怎么构建镜像；好 Dockerfile 要合理分层、利用缓存和多阶段构建、最终镜像小，且**绝不把密钥写进镜像层**。

---

## 核心概念

| 概念 | 一句话 |
|---|---|
| Image | 分层文件系统 + 元数据 |
| Container | 镜像 + 可写层 + 进程 |
| Volume | 持久数据，容器删了数据还在 |
| Network | bridge / host / none；同网可用服务名互访 |
| Compose | 多容器编排（开发/小项目常用） |

### 常用命令

以下是练习命令，不是本文已执行的操作。旧镜像标签仅演示语法，实际项目选受支持版本、修补安全更新并记录 digest；示例口令不是生产可用默认值。删容器/卷、构建推送与部署均要先确认目标。

```bash
docker pull nginx:1.27-alpine
docker images
docker run -d --name web -p 8080:80 nginx:1.27-alpine
docker ps -a
docker logs -f --tail=200 web
docker exec -it web sh
docker stop web && docker rm web

# 卷与网络
docker volume create app-data
docker network create app-net
docker run -d --name db --network app-net \
  -v app-data:/var/lib/mysql \
  -e MYSQL_ROOT_PASSWORD=change_me \
  mysql:8.0
```

Compose 最小心智模型：`docker compose up -d` / `down` / `logs -f` / `ps`。

---

## Dockerfile：多阶段构建与常见坑

### 推荐模板（多阶段）

```dockerfile
# ===== build =====
FROM golang:1.22-alpine AS builder
WORKDIR /src
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 go build -o /out/app ./cmd/app

# ===== runtime =====
FROM alpine:3.20
RUN adduser -D -H app
WORKDIR /app
COPY --from=builder /out/app .
USER app
EXPOSE 8080
CMD ["./app"]
```

Java / Node 同理：构建与运行分阶段；运行镜像需匹配 libc、CA 证书、时区和动态库等依赖，不盲目把所有程序搬进 alpine/distroless。

### 缓存友好写法

1. 先 COPY 依赖清单（`package.json` / `pom.xml` / `go.mod`），再 `install`  
2. 再 COPY 源码 —— 改代码时不必重下依赖  
3. 合并 `RUN` 时注意可读性；清理缓存（`rm -rf /var/lib/apt/lists/*`）

### 常见坑（面试爱问）

| 坑 | 后果 | 改法 |
|---|---|---|
| `ADD` 乱用 / 远程 tar | 不可控 | 优先 `COPY` |
| 把 `.env`、私钥 COPY 进镜像 | 密钥泄漏（层里永久存在） | BuildKit secret mount；运行时密钥通道注入，不能用 ARG/ENV 当构建密钥通道 |
| 用 `latest` | 不可复现 | 钉版本标签 |
| 以 root 跑业务 | 权限风险 | `USER` 非 root |
| 一个容器塞多个主进程 | 难重启难观测 | 一进程一容器（sidecar 另说） |
| 忽略 `.dockerignore` | 上下文巨大、缓存失效 | 排除 `node_modules`、`.git`、真实 `.env*`/密钥文件（保留无秘密的示例配置） |
| 改一行代码整层重建 | 构建慢 | 依赖与源码分层 |

### 健康检查示例

```dockerfile
HEALTHCHECK --interval=30s --timeout=3s --retries=3 \
  CMD wget -qO- http://127.0.0.1:8080/healthz || exit 1
```

---

## 可操作清单

- [ ] 本地能 `docker build -t myapp:dev .` 并 `run` 通  
- [ ] 数据目录用 volume，不写容器可写层重要数据  
- [ ] 镜像扫描（如 `docker scout` / Trivy）扫高危  
- [ ] CI 里构建并推到私有仓库，部署只拉镜像不现场编译（生产更稳）

## 加厚：隔离、缓存和秘密分别怎么理解？

容器通常共享宿主机内核，通过 namespaces 隔离可见资源、cgroups 约束资源，不是每个容器都有完整独立内核的虚拟机。镜像层可共享，运行时另有可写层；重建容器并不保存该层中的业务数据，持久数据放受管理卷/外部服务并做备份。

Dockerfile 缓存按指令和相关输入复用。把依赖清单和锁文件先 COPY，再安装依赖，最后 COPY 业务源码，能减少仅改源码时的重新下载。多阶段构建的目标是**只把运行需要的产物带入最终镜像**，不是阶段越多越好。

| 配置方式 | 用途 | 是否适合密钥 |
|---|---|---|
| ARG | 构建期非敏感参数 | 否，可能进入历史、元数据或构建证明 |
| ENV | 镜像/运行进程配置 | 不是安全构建密钥通道，运行时也可能被高权限读到 |
| BuildKit secret mount | 构建某一步临时使用凭据 | 更合适，但不能把秘密复制到输出或打印日志 |
| 运行时 secret / vault | 部署时按权限提供秘密 | 需要权限、轮换、审计；不是自动绝密 |

示意（已有 package.json 与锁文件且采用 npm 的构建阶段）：

```dockerfile
RUN --mount=type=secret,id=npmrc,target=/root/.npmrc npm ci
```

由受控构建流程提供该 secret，不在 Dockerfile 写真实凭据。把秘密先 COPY 再在后一层 rm，并不能清除旧层内容；一旦发布过，需按泄漏处理与轮换，而不只是删掉最新文件。

## 排障检查清单（只读优先）

```bash
# 示例名称 web，先确认环境与权限
# 输出/日志也可能含业务信息，外发前脱敏
docker ps -a --filter name=web
docker inspect --format '{{json .State}}' web
docker logs --since 10m --timestamps web
docker stats --no-stream web
docker port web
docker network inspect app-net
```

| 现象 | 优先检查 | 别急着做 |
|---|---|---|
| 容器启动即退出 | 主进程是否前台运行、退出码、启动命令、缺配置 | 无脑加 restart 掩盖启动错误 |
| 137 / OOMKilled | State.OOMKilled、内存限制、宿主机事件、应用内存趋势 | 认定 137 必然是 OOM；SIGKILL 也可能产生该码 |
| 容器内能访问，宿主访问不了 | 程序是否监听 0.0.0.0、端口映射、防火墙 | 把 EXPOSE 当成已经发布端口 |
| Web 连不上数据库 | 是否同网络、服务名/端口、数据库就绪、账号权限 | 在 Web 容器用 localhost 找另一个容器 |
| Permission denied | 挂载权限、运行 UID/GID、只读挂载、安全策略 | 直接 chmod 777 / privileged |
| 改文件没生效 | 镜像 digest、构建上下文、COPY 路径、卷是否覆盖镜像文件 | 每次都 --no-cache 或删所有镜像 |
| unhealthy | 健康命令是否存在、探针地址、启动时间、真实业务健康 | 假设 Docker 一定自动重启 unhealthy 容器 |

- Docker HEALTHCHECK 提供健康状态，普通 Docker 运行模式不会仅因 unhealthy 就必然自动重启；重启策略与编排器行为要另看。
- Compose 的启动顺序不等于依赖可接请求；可配健康条件，但应用仍需有限重连，因为依赖运行中也会失败。
- `docker inspect` 全量输出可能含环境变量；优先限制字段，绝不把完整输出随手发群。
- `compose down -v`、volume prune 等可能永久删数据；不把清卷当通用排障。卷不是备份，恢复能力要演练。
- 镜像 tag 可被覆盖，严格复现需记录 digest；同时保持安全更新策略，不能钉死旧版本永不修补。

## 面试口述（机制 + 排障 4 句）

> Docker 镜像是分层只读模板，容器是隔离运行的进程和可写层，重要数据通过卷或外部存储持久化。Dockerfile 用缓存友好的 COPY 顺序与多阶段构建减小最终产物，不靠无限合并/增加层。构建密钥用 secret mount，不能放 ARG、ENV 或镜像层。容器异常我先看状态、退出码、日志、资源和网络证据，再做受控变更，不直接清卷或开放特权。

## 延伸阅读

- 已读开源：[Compose Nginx + Go + MySQL 示例](../开源精选/devops/docker-awesome-compose-nginx-golang-mysql-README.md)。示例说明结构，不是生产安全基线。
- 官方核验：[Build secrets](https://docs.docker.com/build/building/secrets/)。
