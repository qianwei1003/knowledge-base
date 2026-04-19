---
来源: Docker 官方文档
原始链接: https://docs.docker.com/build/building/best-practices/
---

# Docker 构建最佳实践（官方文档）

> 来源：Docker 官方文档 — Building Best Practices
> 整理时间：2026-04-19

## 1. 使用多阶段构建

多阶段构建通过在构建阶段和最终输出之间建立清晰的分离，来减少最终镜像大小。将 Dockerfile 指令拆分为不同阶段，确保最终输出只包含运行应用所需的文件。

多个阶段还可以通过并行执行构建步骤来提高效率。

### 创建可复用的阶段

如果有多个镜像有大量共同部分，考虑创建一个可复用的阶段，包含共享组件。Docker 只需构建一次公共阶段，衍生镜像在 Docker 主机上更高效地使用内存并更快加载。

## 2. 选择正确的基础镜像

实现安全镜像的第一步是选择正确的基础镜像：

- **Docker Official Images**：经过策展的集合，有清晰的文档、推广最佳实践、定期更新
- **Verified Publisher**：由与 Docker 合作的发布者维护的高质量镜像
- **Docker-Sponsored Open Source**：Docker 赞助的开源项目发布的镜像

选择基础镜像时注意：
- 选择与需求匹配的**最小基础镜像**
- 较小的基础镜像提供可移植性和快速下载
- 考虑使用两种基础镜像：一种用于构建/测试，另一种（更精简）用于生产

## 3. 定期重建镜像

Docker 镜像是不可变的。为保持镜像更新和安全，应定期使用更新的依赖重建镜像。

```bash
# 获取最新基础镜像
docker build --pull -t my-image:my-tag .

# 完全干净的构建（最新基础镜像 + 无缓存）
docker build --pull --no-cache -t my-image:my-tag .
```

## 4. 使用 .dockerignore 排除无关文件

使用 `.dockerignore` 文件排除与构建无关的文件，支持类似 `.gitignore` 的排除模式。

```plaintext
*.md
.git
node_modules
```

## 5. 创建临时容器

镜像定义的容器应尽可能临时（ephemeral）。这意味着容器可以随时停止和销毁，然后用最少的设置和配置重建和替换。

## 6. 不要安装不必要的包

避免安装额外的包。减少复杂性、依赖、文件大小和构建时间。

## 7. 解耦应用

每个容器应只有一个关注点。将应用解耦到多个容器中，使水平扩展和复用更容易。

- Web 应用栈可能由三个独立容器组成：Web 应用、数据库、内存缓存
- 每个容器限制一个进程是好的经验法则
- 如果容器相互依赖，使用 Docker 容器网络进行通信

## 8. 排序多行参数

尽可能按字母顺序排序多行参数，避免包重复，使列表更易更新，PR 也更容易阅读和审查。

```dockerfile
RUN apt-get update && apt-get install -y --no-install-recommends \
  bzr \
  cvs \
  git \
  mercurial \
  subversion \
  && rm -rf /var/lib/apt/lists/*
```

## 9. 利用构建缓存

理解构建缓存的工作原理和缓存失效机制，对于确保更快的构建至关重要。

## 10. 固定基础镜像版本

镜像标签是可变的。要完全保护供应链完整性，可以将镜像版本固定到特定摘要（digest）：

```dockerfile
FROM alpine:3.21@sha256:a8560b36e8b8210634f77d9f7f9efd7ffa463e380b75e2e74aff4511df3ef88c
```

Docker Scout 提供自动修复工作流，当有新的镜像摘要可用时，可以自动创建 PR 更新 Dockerfiles。

## 11. 在 CI 中构建和测试镜像

将代码推送到源代码控制或创建 PR 时，使用 GitHub Actions 或其他 CI/CD 管道自动构建、标记和测试 Docker 镜像。

## Dockerfile 指令最佳实践

### FROM

尽可能使用当前官方镜像作为基础。Docker 推荐 **Alpine 镜像**（不到 6 MB）作为紧凑的基础。

### LABEL

使用标签来组织镜像、记录许可信息、辅助自动化：

```dockerfile
LABEL vendor=ACME\ Incorporated \
      com.example.version="0.0.1-beta" \
      com.example.release-date="2026-04-19"
```

### RUN

将长或复杂的 `RUN` 语句拆分为多行，使用 `&&` 链接命令：

```dockerfile
RUN apt-get update && apt-get install -y --no-install-recommends \
    package-bar \
    package-baz \
    && rm -rf /var/lib/apt/lists/*
```

**apt-get 注意事项**：
- 始终将 `RUN apt-get update` 与 `apt-get install` 合并在同一个 `RUN` 语句中
- 使用 `--no-install-recommends` 避免安装不必要的推荐包
- 清理 apt 缓存（`rm -rf /var/lib/apt/lists/*`）减少镜像大小

**管道使用**：如果管道中的某个命令失败，使用 `set -o pipefail &&` 确保构建不会意外成功：

```dockerfile
RUN set -o pipefail && wget -O - https://some.site | wc -l > /number
```

### CMD

`CMD` 应几乎总是使用 `CMD ["executable", "param1", "param2"]` 形式。对于服务类镜像，如 `CMD ["apache2","-DFOREGROUND"]`。

### ENTRYPOINT

最佳用法是设置镜像的主命令，然后用 `CMD` 作为默认参数：

```dockerfile
ENTRYPOINT ["s3cmd"]
CMD ["--help"]
```

### USER

如果服务可以无特权运行，使用 `USER` 切换到非 root 用户：

```dockerfile
RUN groupadd -r postgres && useradd --no-log-init -r -g postgres postgres
USER postgres
```

### VOLUME

使用 `VOLUME` 暴露数据库存储区域、配置存储或容器创建的文件。

### EXPOSE

使用应用的常用传统端口。

### ENV

使用 `ENV` 更新 `PATH` 环境变量，或设置服务所需的环境变量：

```dockerfile
ENV PATH=/usr/local/nginx/bin:$PATH
ENV PG_MAJOR=9.3
ENV PG_VERSION=9.3.4
```

### ADD 或 COPY

- 优先使用 `COPY` 进行文件复制
- `ADD` 适用于需要从远程 URL 下载文件或自动解压 tar 文件的场景
- 在多阶段构建中，`COPY --from=<stage>` 从构建阶段复制文件非常高效
- 可考虑使用 bind mount（`RUN --mount=type=bind`）替代临时 COPY，效率更高

## 关键原则总结

| 原则 | 说明 |
|------|------|
| 最小化镜像 | 使用 Alpine/distroless 等精简基础镜像 |
| 多阶段构建 | 分离构建依赖和运行时依赖 |
| 层缓存优化 | 将变化频率低的指令放在前面 |
| 安全性 | 使用非 root 用户、固定版本、定期更新 |
| 可维护性 | 排序参数、合并 RUN 指令、使用 ENV 管理版本 |
| 可复现性 | 固定基础镜像 digest、版本锁定 |
