---
来源: CSDN博客
领域: 后端
关键词: Nginx, 反向代理, 负载均衡, 限流, HTTPS, 生产环境, 部署
质量: ⭐⭐⭐⭐⭐
提炼状态: 待提炼
原始链接: https://blog.csdn.net/jj1245_/article/details/151934007
---

# Nginx 生产部署避坑指南 + 最佳实践

## 一、Nginx 是什么？

Nginx 是一个高性能的「反向代理服务器」：
- 外部请求先经过它，再转发到后端服务（反向代理）
- 能同时处理上万个并发连接（高并发处理）
- 处理静态文件、压缩数据、防恶意攻击

## 二、反向代理 & 负载均衡

### 配置目标
让 Nginx 把请求均匀转发到多台后端服务器，隐藏真实 IP，自动剔除故障节点。

```nginx
# 全局配置
user  nginx;
worker_processes  1;
error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;

# 负载均衡配置
upstream backend_servers {
    server 192.168.1.10:8080;
    server 192.168.1.11:8080;
    server 192.168.1.12:8080;

    # 进阶配置
    least_conn;  # 最小连接数策略
    keepalive 32;  # 保持32个长连接
    proxy_next_upstream error timeout http_500;  # 故障重试
}

server {
    listen       80;
    server_name  www.yourdomain.com;

    location /api/ {
        proxy_pass http://backend_servers/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

        proxy_connect_timeout 30s;
        proxy_read_timeout 60s;
        proxy_send_timeout 60s;
    }
}
```

**关键点：**
- 后端服务器 IP 全藏在 Nginx 里
- 流量均匀分散到多台服务器

## 三、静态资源处理 & 动静分离

### 配置目标
让 Nginx 直接处理图片、CSS、JS 等静态文件，减轻后端压力。

```nginx
server {
    listen       80;
    server_name  www.yourdomain.com;

    # 静态资源路径
    location /static/ {
        root /data/;
        autoindex off;
        expires 30d;  # 浏览器缓存30天
        gzip on;
        gzip_types text/css application/javascript image/png;
    }

    location /images/ {
        root /data/;
        # 防盗链
        valid_referers none blocked www.yourdomain.com;
        if ($invalid_referer) {
            return 403;
        }
    }

    # 动态请求转发
    location /api/ {
        proxy_pass http://backend_servers/;
    }
}
```

**关键点：**
- 静态文件直接由 Nginx 返回，速度比后端处理快 10 倍以上
- 浏览器缓存 + 压缩，用户第二次访问秒加载

## 四、限流防刷 & IP 黑白名单

### 配置目标
限制单个 IP 的并发连接数和请求频率，拉黑恶意 IP。

```nginx
http {
    # 并发连接限制
    limit_conn_zone $binary_remote_addr zone=ip_conn:10m;
    # 请求频率限制
    limit_req_zone $binary_remote_addr zone=ip_req:10m rate=5r/s;
}

server {
    listen 80;
    server_name www.yourdomain.com;

    location /api/login {
        # 并发连接限制：每个IP最多10个
        limit_conn ip_conn 10;
        # 请求频率限制：突发最多排队10个
        limit_req zone=ip_req burst=10 nodelay;

        # IP黑白名单
        allow 192.168.1.0/24;
        deny 10.0.0.1;

        proxy_pass http://backend_servers/;
    }
}
```

**关键点：**
- 恶意 IP 频繁刷接口直接返回 403
- 登录接口限流防止 CC 攻击

## 五、HTTPS 配置

### 配置目标
启用 HTTPS，让数据加密传输。

```nginx
server {
    listen       443 ssl;
    server_name  www.yourdomain.com;

    ssl_certificate      /etc/nginx/ssl/yourdomain.crt;
    ssl_certificate_key  /etc/nginx/ssl/yourdomain.key;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES256-GCM-SHA384;
    ssl_prefer_server_ciphers on;

    # HTTP重定向到HTTPS
    rewrite ^(.*)$ https://$host$1 permanent;

    location / {
        proxy_pass http://backend_servers/;
    }
}
```

## 六、常用运维命令

```bash
# 安装
# Linux: yum install nginx 或 apt-get install nginx
# Windows: 官网下载解压，双击nginx.exe

# 启动/重启
sudo systemctl start nginx
sudo systemctl restart nginx

# 检查配置
nginx -t
```

## 总结：Nginx 生产部署清单

| 功能 | 配置要点 |
|------|----------|
| 反向代理 | 藏好后端 IP，proxy_set_header 传递客户端信息 |
| 负载均衡 | upstream 定义服务器组，least_conn 均衡策略 |
| 静态资源 | root 指定路径，expires 设置缓存，gzip 压缩 |
| 限流防刷 | limit_conn + limit_req，黑白名单 |
| HTTPS | ssl_certificate 配置证书，rewrite 强制 HTTPS |

**核心原则**：代码可以慢慢写，Nginx 必须稳如狗；配置写对了，摸鱼才安心！
