# 实战示例：Docker Compose 模板（可复制改造）

学习目标（读完你应该能做到）：

- 看懂一个最小 `compose.yml`，理解 services/ports/volumes/depends_on
- 会用 `docker compose up/down/logs/exec` 操作多容器项目
- 知道把模板“生产化”时通常还要补哪些项（healthcheck、资源限制、日志等）

> 目标：给常见“Web + DB + Cache + 反向代理”一个可落地的起步模板。

## 1. 最小 compose.yml（示意）

以下为“示意模板”（字段你可以按项目改）：

- nginx：对外入口，转发到 app
- app：你的业务服务
- mysql：数据库（使用 volume）
- redis：缓存

```yaml
services:
  nginx:
    image: nginx:alpine
    ports:
      - "8080:80"
    depends_on:
      - app

  app:
    image: your-app:latest
    environment:
      - DATABASE_URL=mysql://root:pass@mysql:3306/app
      - REDIS_HOST=redis
    depends_on:
      - mysql
      - redis

  mysql:
    image: mysql:8
    environment:
      - MYSQL_ROOT_PASSWORD=pass
      - MYSQL_DATABASE=app
    volumes:
      - mysql_data:/var/lib/mysql

  redis:
    image: redis:7

volumes:
  mysql_data:
```

> 提示：你后续可以把 app 改成 `build:`，直接用 Dockerfile 构建。

## 2. 常用操作

```bash
docker compose up -d
docker compose ps
docker compose logs -f

docker compose exec app sh

docker compose down
```

## 3. 生产化通常还会补的点

- healthcheck（服务就绪检查）
- 限制资源（cpu/memory）
- 日志与轮转策略（避免日志撑爆磁盘）
- secrets 管理（不要把密码写进仓库）
