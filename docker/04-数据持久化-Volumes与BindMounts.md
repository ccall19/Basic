# Docker 数据持久化：Volumes vs Bind Mounts

学习目标（读完你应该能做到）：

- 分清 volume / bind mount / tmpfs 的适用场景
- 能为数据库容器正确挂载 volume，避免“删容器数据就没了”
- 知道权限与性能（尤其 Docker Desktop）相关的常见坑

## 1. 为什么需要持久化

容器的可写层（container writable layer）是临时的：

- 删除容器数据就没了
- 升级/滚动重启会丢失状态

持久化通常通过：

- **Volume**（推荐用于数据库/持久化数据）
- **Bind Mount**（推荐用于开发态挂载源码/配置）

一句话记忆：

- **Volume**：把数据交给 Docker 管（你只需要记住 volume 名称）
- **Bind Mount**：把宿主机的某个真实路径“映射进容器”（路径由你指定）

快速对比：

| 项 | Volume | Bind Mount |
|---|---|---|
| 你需要记住的东西 | volume 名称 | 宿主机路径 |
| 推荐用途 | 数据库/长期数据 | 开发挂源码/配置 |
| 常见坑 | 较少（仍可能有 UID/GID） | 权限/性能更常见（尤其 Docker Desktop） |

---

## 1.1 从写法上如何区分（避免被相对路径迷惑）

你经常会看到两种写法都长这样：

```bash
-v SOURCE:TARGET
```

区分关键：**只看冒号左边 `SOURCE` 到底是“路径”还是“名字”**。

- **Bind Mount**：`SOURCE` 是宿主机路径（绝对/相对都算路径）
  - 绝对路径：`/Users/...`、`/data/...`
  - 相对路径：`./`、`../`（就算很短也仍然是 bind）
  - 变量展开：`"${PWD}"`、`$HOME/...`

  示例：

  ```bash
  -v ./nginx.conf:/etc/nginx/nginx.conf
  -v "${PWD}":/work
  ```

- **Volume**：左边是一个“名字”（例如 `mysql_data`），不是路径

  ```bash
  -v mysql_data:/var/lib/mysql
  ```

如果你担心混淆，**推荐用更清晰的 `--mount`**（一眼能看到 type）：

```bash
# bind
--mount type=bind,source="${PWD}",target=/work

# volume
--mount type=volume,source=mysql_data,target=/var/lib/mysql
```

不确定时还可以用 `docker inspect <container>` 看 `Mounts` 里的 `Type`（bind/volume）来“验尸”。

---

## 2. Volume（推荐）

### 2.1 特点

- Docker 管理路径（你不需要关心具体在哪）
- 迁移/备份相对清晰
- 权限问题更少（但仍可能遇到 UID/GID）

### 2.2 常用命令

```bash
docker volume ls
docker volume create myvol
docker volume inspect myvol
docker volume rm myvol
```

### 2.3 使用示例（MySQL）

```bash
docker run -d --name mysql \
  -e MYSQL_ROOT_PASSWORD=pass \
  -v mysql_data:/var/lib/mysql \
  mysql:8
```

---

## 3. Bind Mount（开发常用）

```bash
docker run -it --rm \
  -v "${PWD}":/work \
  -w /work \
  python:3.12-slim \
  python -V
```

### 常见坑

- 宿主机文件权限 / 容器内用户权限不一致
- macOS/Windows 下性能可能不如 Linux（大量小文件时更明显）

---

## 4. tmpfs（更快但不持久）

适合缓存、临时文件：

```bash
docker run -d --tmpfs /tmp:rw,size=256m IMAGE
```

---

## 5. 备份与迁移（思路）

- Volume 本质也是目录：可以用临时容器把数据打包出来
- 或者使用数据库自带备份工具（更可靠）

（后续你想的话，我可以补一个“通用 volume 备份/恢复脚本模板”。）
