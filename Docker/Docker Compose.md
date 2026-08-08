---
title: Docker Compose
type: knowledge
tags: [Docker, Compose, 编排]
status: draft
created: 2026-08-08
updated: 2026-08-08
---

# Docker Compose

Compose 使用 YAML 描述多容器应用的服务、网络、卷和配置。

```yaml
services:
  web:
    build: .
    ports:
      - "8080:8000"
    environment:
      DB_HOST: db
    depends_on:
      db:
        condition: service_healthy
  db:
    image: postgres:17
    volumes:
      - dbdata:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 5s
      timeout: 3s
      retries: 5

volumes:
  dbdata:
```

常用命令：

```bash
docker compose config
docker compose up -d --build
docker compose ps
docker compose logs -f
docker compose exec web sh
docker compose down
```

`depends_on` 表达启动依赖；如果要等待应用真正就绪，应结合健康检查。配置中的密码不要直接提交，可使用环境文件、Secret 或外部密钥系统，并明确哪些文件进入版本控制。

## 参考资料

- [Docker Compose quickstart](https://docs.docker.com/compose/gettingstarted/)
