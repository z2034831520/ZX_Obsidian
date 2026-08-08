---
title: 非root用户和最小权限
aliases: [非 root 用户和最小权限]
type: knowledge
tags: [Docker, 安全, 最小权限]
status: draft
created: 2026-08-08
updated: 2026-08-08
---

# 非 root 用户和最小权限

容器隔离不是完整安全边界。进程默认以 root 运行时，一旦发生逃逸或错误挂载，影响可能更大。镜像应创建专用用户并通过 `USER` 切换：

```dockerfile
RUN addgroup --system app && adduser --system --ingroup app app
COPY --chown=app:app . /app
USER app
```

运行时还应：

- 只授予必需的 Linux capabilities，尽量 `--cap-drop=ALL`。
- 避免 `--privileged`。
- 不挂载 Docker socket。
- 根文件系统可行时设为只读，并单独提供可写目录。
- 限制设备、进程数、CPU 和内存。
- 使用 seccomp、AppArmor、SELinux 或同类机制。
- 不把宿主敏感目录绑定进容器。
- Secret 以最小范围提供，并定期轮换。

非 root 运行需要同步处理文件所有权、监听低端口和卷权限。不能为了“能启动”就退回 root，而应修正权限模型。

## 参考资料

- [Docker security](https://docs.docker.com/engine/security/)
