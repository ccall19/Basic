# Dockerfile 必备知识速查

学习目标（读完你应该能做到）：

- 写出一个可运行的 Dockerfile（WORKDIR/COPY/RUN/ENV/EXPOSE/CMD）
- 知道如何“拆分层”来提高缓存命中、加速构建
- 了解多阶段构建的用途，并能把运行镜像做得更小

## 1. Dockerfile 的基本目标

- **可复现**：同一份源码 + 同一份 Dockerfile，在任意机器构建出一致镜像
- **可维护**：更少的“魔法步骤”，分层更清晰
- **更快**：提高构建缓存命中率
- **更小**：减少无用层与无用文件

---

## 2. 常用指令（按使用频率）

### 2.1 FROM

```dockerfile
FROM python:3.12-slim
```

- 多阶段构建：`FROM ... AS builder`

### 2.2 WORKDIR

```dockerfile
WORKDIR /app
```

### 2.3 COPY / ADD（优先 COPY）

```dockerfile
COPY . .
COPY pyproject.toml ./
```

- `ADD` 支持自动解压与远程 URL，一般不需要；优先 `COPY`

### 2.4 RUN

```dockerfile
RUN apt-get update && apt-get install -y --no-install-recommends curl \
  && rm -rf /var/lib/apt/lists/*
```

- 合并同类操作减少层数
- 包管理器最后清理缓存

### 2.5 ENV

```dockerfile
ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1
```

### 2.6 EXPOSE（仅声明/文档化）

```dockerfile
EXPOSE 8000
```

### 2.7 USER（避免 root 运行）

```dockerfile
RUN useradd -m appuser
USER appuser
```

- 生产建议尽量不要 root 运行
- 注意：一些基础镜像（如 `alpine`）没有 `useradd`，要用 `adduser`/`addgroup`

### 2.8 ENTRYPOINT / CMD

```dockerfile
ENTRYPOINT ["python", "-m", "yourapp"]
CMD ["--help"]
```

- `ENTRYPOINT`：固定主程序
- `CMD`：默认参数（可被 `docker run ... <args>` 覆盖）

---

## 3. 缓存与构建加速（非常重要）

### 3.1 让“依赖安装层”尽量稳定

核心原则：**先复制依赖清单 → 安装依赖 → 再复制源码**。

以 `requirements.txt` 为例：

```dockerfile
WORKDIR /app
COPY requirements.txt ./
RUN pip install -U pip && pip install -r requirements.txt
COPY . .
```

> 如果你用的是 `pyproject.toml`（Poetry/uv/pdm 等），也保持一致：复制对应 lock 文件并用对应工具安装依赖，不要把 `uv.lock`/`poetry.lock` 和 `requirements.txt` 混在同一个示例里。

（进阶）BuildKit 下可以用缓存挂载加速 pip：

```dockerfile
RUN --mount=type=cache,target=/root/.cache/pip \
    pip install -U pip && pip install -r requirements.txt
```

### 3.2 .dockerignore

一定要写 `.dockerignore`，避免把 `node_modules/`、`.git/`、日志、数据文件复制进镜像，减少构建上下文、提高速度。

---

## 4. 多阶段构建（减小体积）

典型场景：前端构建（Node）等。

```dockerfile
FROM node:20 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
```

---

## 5. 常见坑（学习阶段最容易踩）

- 端口映射后访问不到：服务监听了 `127.0.0.1`，应监听 `0.0.0.0`
- 基础镜像太瘦：可能缺 CA 证书或 tzdata
- 依赖安装慢：优化缓存层、减少上下文、写 `.dockerignore`
- bind mount 权限：宿主机 UID/GID 与容器内用户不一致

---

## 6. 推荐练习方式

- 同一项目写出“开发镜像”和“生产镜像”（多阶段）
- 观察 `docker history <image>`，把镜像体积明显压下来（目标导向最快）
