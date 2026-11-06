# 02 Docker 与 Dockerfile

## 面试口述（30 秒）

**镜像**是只读模板，**容器**是镜像的运行实例。数据持久化用**卷**，容器互通用**网络**。Dockerfile 描述怎么构建镜像；好 Dockerfile 要多层、利用缓存、最终镜像小，且**绝不把密钥写进镜像层**。

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

## Dockerfile：多层构建与常见坑

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

Java / Node 同理：编译阶段用大镜像，运行阶段用 JRE/alpine/distroless。

### 缓存友好写法

1. 先 COPY 依赖清单（`package.json` / `pom.xml` / `go.mod`），再 `install`  
2. 再 COPY 源码 —— 改代码时不必重下依赖  
3. 合并 `RUN` 时注意可读性；清理缓存（`rm -rf /var/lib/apt/lists/*`）

### 常见坑（面试爱问）

| 坑 | 后果 | 改法 |
|---|---|---|
| `ADD` 乱用 / 远程 tar | 不可控 | 优先 `COPY` |
| 把 `.env`、私钥 COPY 进镜像 | 密钥泄漏（层里永久存在） | build-arg/secret、运行时注入 |
| 用 `latest` | 不可复现 | 钉版本标签 |
| 以 root 跑业务 | 权限风险 | `USER` 非 root |
| 一个容器塞多个主进程 | 难重启难观测 | 一进程一容器（sidecar 另说） |
| 忽略 `.dockerignore` | 上下文巨大、缓存失效 | 排除 `node_modules`、`.git` |
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
