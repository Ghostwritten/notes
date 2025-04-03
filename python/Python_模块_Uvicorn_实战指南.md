
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/a68198b50a134ba2b5610c0414da5881.png)





Uvicorn 是一个高性能的 ASGI 实现。它是构建和运行 Python 基于 ASGI 的 Web 和实时服务器应用的理想选择。它特别适合于构建高性能、实时的应用，如 WebSocket、HTTP/2、GraphQL API等。本文将通过一些实战例子，教你如何使用 Uvicorn 构建和运行应用。

## 1. 介绍和安装

### 1.1 介绍

Uvicorn 是一个基于 Python 的轻量级 ASGI 服务器，以 uvloop 和 httptools 为核心，以提供极速和高效。

### 1.2 安装

使用 pip 安装：

```bash
pip install uvicorn
```

如果需要最高性能，建议安装以下选项：

```bash
pip install uvicorn[standard]
```

该包含 uvloop 和 httptools，并提供完善的日志和进程以及完整的处理能力。

## 2. 创建基础应用

### 2.1 简单的 HTTP 应用

创建一个文件，名为 `app.py`：

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
def read_root():
    return {"message": "Hello, World!"}
```

使用 Uvicorn 运行该应用：

```bash
uvicorn app:app --host 0.0.0.0 --port 8000 --reload
```

### 2.2 添加日志

实际应用中，日志很重要。使用 `--log-level` 可以控制日志输出：

```bash
uvicorn app:app --log-level debug
```

定制日志格式：

```bash
uvicorn app:app --log-config logging.yaml
```

一个例子 `logging.yaml`：

```yaml
version: 1
formatters:
  default:
    format: "%(asctime)s - %(name)s - %(levelname)s - %(message)s"
handlers:
  console:
    class: logging.StreamHandler
    formatter: default
loggers:
  uvicorn:
    handlers: [console]
    level: INFO
root:
  handlers: [console]
  level: INFO
```

## 3. 实现高级功能

### 3.1 支持 WebSocket

在添加 WebSocket 支持时，可以使用 FastAPI 的高性能功能。

```python
from fastapi import FastAPI, WebSocket

app = FastAPI()

@app.websocket("/ws")
async def websocket_endpoint(websocket: WebSocket):
    await websocket.accept()
    while True:
        data = await websocket.receive_text()
        await websocket.send_text(f"Message text was: {data}")
```

运行该应用，并使用浏览器接入 WebSocket。

### 3.2 优化运行性能

在高流量场景中，使用 `--workers` 可以超现行处理。

```bash
uvicorn app:app --host 0.0.0.0 --port 8000 --workers 4
```

举例：指定服务器使用 uvloop ：

```bash
uvicorn app:app --loop uvloop
```

## 4. Nginx 配置反向代理

在生产环境中，使用 Nginx 作为反向代理，可以提供以下优势：

- SSL 支持
- 更好的负载均衡能力
- 请求过滤与缓存

以下是具体配置步骤：

### 4.1 安装与配置 Nginx

1. **安装 Nginx**

在基于 Debian 的系统中：

```bash
sudo apt update
sudo apt install nginx
```

在基于 Red Hat 的系统中：

```bash
sudo yum install nginx
```

2. **编辑 Nginx 配置文件**

创建或修改一个 Nginx 站点配置文件，例如 `/etc/nginx/sites-available/uvicorn`：

```nginx
server {
    listen 80;
    server_name example.com;

    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }

    error_log /var/log/nginx/uvicorn_error.log;
    access_log /var/log/nginx/uvicorn_access.log;
}
```

3. **启用站点配置**

在基于 Debian 的系统中：

```bash
sudo ln -s /etc/nginx/sites-available/uvicorn /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

在基于 Red Hat 的系统中，直接编辑 `/etc/nginx/nginx.conf` 添加上述配置，然后重新加载 Nginx：

```bash
sudo nginx -t
sudo systemctl reload nginx
```

### 4.2 启用 SSL 支持

使用 Let's Encrypt 为域名启用 HTTPS 支持：

1. 安装 Certbot：

```bash
sudo apt install certbot python3-certbot-nginx
```

2. 自动配置 HTTPS：

```bash
sudo certbot --nginx -d example.com
```

完成后，Nginx 将自动更新为支持 HTTPS 的配置。

3. 定期更新证书：

Certbot 会自动添加定期更新任务。如果需要手动测试：

```bash
sudo certbot renew --dry-run
```

### 4.3 性能优化建议

- **启用 HTTP/2 支持**：

在 `listen` 指令中添加 `http2`：

```nginx
listen 443 ssl http2;
```

- **启用 Gzip 压缩**：

在 Nginx 配置中启用 Gzip 压缩：

```nginx
gzip on;
gzip_types text/plain application/json application/javascript text/css;
```

- **优化缓冲设置**：

调整 `proxy_buffer_size` 和 `proxy_buffers`：

```nginx
proxy_buffer_size 128k;
proxy_buffers 4 256k;
proxy_busy_buffers_size 256k;
```

---

## 5. 常见问题与解决方案

### 5.1 高并发问题

在高并发场景下，Uvicorn 可能会因为 CPU 饱和而表现出瓶颈。解决方案包括：

- 增加 `--workers` 参数以启用多进程。
- 使用负载均衡器（如 Nginx 或 HAProxy）分配请求。

### 5.2 WebSocket 超时

WebSocket 可能在高流量时出现超时问题。可以通过调整 Uvicorn 的 `--timeout-keep-alive` 参数增加连接存活时间：

```bash
uvicorn app:app --timeout-keep-alive 120
```

### 5.3 日志不完整

日志在高负载时可能出现丢失情况。解决方法是使用独立日志管理器（如 ELK 或 Fluentd）集中处理日志。

---

## 6. 总结

Uvicorn 是构建现代高性能 Python 应用的利器。通过灵活的配置和良好的性能优化，它可以轻松应对从开发到生产的各种场景。在这篇文章中，我们涵盖了 Uvicorn 的安装、基础功能、性能优化以及部署技巧。希望这些内容能帮助你更高效地使用 Uvicorn。

如果你有任何问题或建议，请在评论区留言，一起交流学习！

