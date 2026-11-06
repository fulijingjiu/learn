# 04 Nginx 与反向代理

## 面试口述（30 秒）

Nginx 常做四件事：**静态站点**、**反向代理**（对外 80/443，对内转 app）、**负载均衡**、**TLS 证书终结**。前端静态资源可由 Nginx 直接吐；API 用 `proxy_pass` 转到 Node/Java/容器。

---

## 配置骨架

### 1. 静态站

```nginx
server {
  listen 80;
  server_name static.example.com;
  root /var/www/html;
  index index.html;
  location / {
    try_files $uri $uri/ /index.html;  # SPA 常用
  }
}
```

### 2. 反向代理

```nginx
upstream app_upstream {
  server 127.0.0.1:8080;
  # server 10.0.0.11:8080;
  # server 10.0.0.12:8080;
}

server {
  listen 80;
  server_name api.example.com;

  location / {
    proxy_pass http://app_upstream;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
  }
}
```

### 3. 负载均衡策略（概念）

- `round_robin`（默认）  
- `least_conn`  
- `ip_hash`（会话粘滞粗糙版）  
- 健康检查：开源版能力有限，生产常靠上游探活 + K8s readiness  

### 4. 证书概念

| 点 | 说明 |
|---|---|
| TLS 终结 | 客户端 ↔ Nginx 加密，Nginx ↔ 后端可内网明文或再加密 |
| 证书来源 | Let’s Encrypt（acme）、公司内 CA、云厂商证书 |
| 文件 | `ssl_certificate`（全链）、`ssl_certificate_key`（私钥，权限 600） |
| 续期 | certbot / acme.sh 定时任务；忘记续期 → 浏览器大红锁 |

```nginx
listen 443 ssl http2;
ssl_certificate     /etc/nginx/certs/fullchain.pem;
ssl_certificate_key /etc/nginx/certs/privkey.pem;
```

HTTP 跳 HTTPS：`return 301 https://$host$request_uri;`

### 常用运维命令

```bash
nginx -t                 # 测配置
nginx -s reload          # 平滑重载
tail -f /var/log/nginx/access.log
tail -f /var/log/nginx/error.log
```

Docker 官方镜像要点见：`../开源精选/devops/docker-library-nginx-README.md`

---

## 可操作清单

- [ ] 本地或容器跑通静态页 + 反代到本机 8080  
- [ ] 会看 `error.log` 区分 upstream timed out / connect refused  
- [ ] 知道 `client_max_body_size` 影响上传  
- [ ] 证书私钥不进 Git
