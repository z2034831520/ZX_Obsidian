---
title: Dockerfile指令
aliases: [Dockerfile 指令]
type: knowledge
tags: [Docker, Dockerfile, 镜像]
status: draft
created: 2026-08-08
updated: 2026-08-08
---

# Dockerfile 指令

Dockerfile 按指令构建镜像层。常用指令：

| 指令 | 作用 |
|---|---|
| `FROM` | 指定基础镜像并开始构建阶段 |
| `WORKDIR` | 设置后续指令工作目录 |
| `COPY` | 从构建上下文复制文件 |
| `RUN` | 构建时执行命令 |
| `ENV` | 设置镜像运行环境变量 |
| `ARG` | 设置构建期变量 |
| `USER` | 指定后续构建或运行用户 |
| `EXPOSE` | 声明应用监听端口，不负责发布端口 |
| `ENTRYPOINT` | 定义主要可执行程序 |
| `CMD` | 提供默认命令或默认参数 |
| `HEALTHCHECK` | 定义容器健康检测 |

推荐使用 JSON exec 形式：

```dockerfile
FROM python:3.14-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
USER 10001
CMD ["python", "main.py"]
```

`ADD` 支持额外行为，普通本地复制优先使用语义更清晰的 `COPY`。构建上下文应配合 `.dockerignore` 排除密钥、缓存和无关大文件。

## 参考资料

- [Dockerfile reference](https://docs.docker.com/reference/dockerfile/)
