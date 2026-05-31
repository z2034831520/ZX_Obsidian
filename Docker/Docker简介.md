## 简介
Docker 是一个开源的**容器化平台**，让开发者可以将应用程序及其所有依赖打包成一个标准化的单元——**容器（Container）**，从而实现"一次构建，到处运行"。其常常用来与虚拟机进行对比，[Docker VS 虚拟机](Docker%20VS%20虚拟机.md)。

## 核心概念

**镜像（Image）** 只读的模板，包含运行应用所需的代码、运行时、库和配置。类比为"安装光盘"。

**容器（Container）** 镜像的运行实例，相互隔离、轻量。类比为"正在运行的程序"。

**Dockerfile** 用于构建镜像的脚本文件，描述如何一步步组装环境。

**Registry（镜像仓库）** 存储和分发镜像的地方，最常用的是 [Docker Hub](https://hub.docker.com)。

## 常用命令速查

```bash
# 拉取镜像
docker pull nginx

# 运行容器
docker run -d -p 8080:80 --name my-nginx nginx

# 查看运行中的容器
docker ps

# 进入容器
docker exec -it my-nginx bash

# 停止 / 删除容器
docker stop my-nginx
docker rm my-nginx

# 查看所有镜像
docker images
```

## Dockerfile 示例

```dockerfile
FROM node:18-alpine        # 基础镜像
WORKDIR /app               # 设置工作目录
COPY package*.json ./      # 复制依赖文件
RUN npm install            # 安装依赖
COPY . .                   # 复制源码
EXPOSE 3000                # 声明端口
CMD ["node", "index.js"]   # 启动命令
```
## Docker 的核心优势

- **环境一致性**：开发、测试、生产环境完全一致，彻底告别"在我机器上能跑"
- **快速启动**：秒级启动，远快于虚拟机的分钟级
- **轻量隔离**：共享宿主内核，比 VM 节省大量内存和磁盘
- **易于分发**：镜像推送到 Registry，团队成员一条命令即可运行
- **微服务友好**：天然适合将应用拆分为多个独立容器
