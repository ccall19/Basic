# 02｜ARP/邻居表：解决“同网段不通”的关键

同网段通信的第一步不是 TCP，而是 **ARP（IPv4）/ND（IPv6）把 IP 解析成 MAC**。

---

## 1. 30 秒检查清单

当你怀疑“同网段不通”时：

1) 看邻居表状态：
- `ip neigh`

2) 抓 ARP 看有没有问、有没有答：
- `tcpdump -ni <网卡> arp`

3) 临时刷新邻居条目（谨慎）：
- `ip neigh flush all`（会短暂影响连接，线上慎用）

---

## 2. 你要看懂 `ip neigh` 的几个状态

- `REACHABLE`：近期通信正常
- `STALE`：很久没用过，下一次会重新确认
- `DELAY/PROBE`：正在确认对端
- `INCOMPLETE`：发了 ARP 请求但还没拿到回应
- `FAILED`：确认失败

经验：
- 看到大量 `INCOMPLETE/FAILED`，基本可以断定二层/安全策略/对端问题。

---

## 3. 最常见的 4 类原因（按出现频率）

### 3.1 目标不在同二层（你以为同网段）
- 典型：子网掩码/前缀配置错、VLAN 不通
- 用 `ip addr` 检查前缀；确认交换/VLAN

### 3.2 云/机房安全策略阻断（ARP/ND 也可能被限制）
- 有些环境对二层广播/邻居发现有约束

### 3.3 IP 冲突
- 现象：时通时不通；抓包看到 ARP reply 的 MAC 在变
- 抓包：`tcpdump -e -ni <网卡> arp`

### 3.4 虚拟化/容器网络路径复杂
- 例如 bridge、veth、bond、overlay
- 先确认你抓的是“正确那张网卡”

---

## 4. 抓包怎么看（只记住最小集合）

你需要看懂两句：
- Who has `X.X.X.X`? Tell `Y.Y.Y.Y`（请求）
- `X.X.X.X` is-at `aa:bb:cc:dd:ee:ff`（应答）

抓包命令：
- `tcpdump -e -n -i <网卡> arp`

---

## 5. 一句话心法

- “同网段不通”优先查 ARP/ND，不要先怀疑 TCP/端口。
