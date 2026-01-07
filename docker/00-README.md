# Docker 学习与使用笔记（本目录）

这份目录用于沉淀 Docker 的核心概念 + 日常命令速查 + 可靠的排障方法。

## 文件导航

- `01-命令.md`：Docker/Compose 常用命令速查（日常使用优先看这个）
- `02-Dockerfile.md`：Dockerfile 指令、最佳实践、构建与缓存
- `03-镜像原理与优化.md`：分层、tag、缓存、瘦身、构建策略
- `04-数据持久化-Volumes与BindMounts.md`：volume/bind/tmpfs、权限/备份/迁移
- `05-网络与端口.md`：网络模型、端口映射、DNS、排障 checklist
- `06-日志与轮转.md`：docker logs、日志驱动、json-file 轮转、Compose 配置
- `07-排障手册.md`：从“起不来/连不上/慢/磁盘爆了”到定位的标准流程
- `08-安全与权限.md`：最小权限、非 root、capabilities、secrets（以及可扩展的扫描/加固思路）
- `09-实战示例-compose.md`：可复制的 compose 模板（nginx+app+mysql+redis）

> 后续你如果希望加 Kubernetes/Containerd/OCI 体系，也可以在本目录再拆一层 `k8s/`。
