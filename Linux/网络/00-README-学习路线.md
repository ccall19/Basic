# Linux 服务器网络：入门但较完整的知识清单（面向运维/排障）

目标：让你在 Linux 服务器上**能看懂网络现象、能定位常见问题、能做出正确配置与优化取舍**。

> 这份目录的“网络知识”以 Linux 为中心：更关注 Linux 上的配置、命令、抓包与内核行为；协议只讲到对排障/性能/安全有用的深度。

---

## 0. 你最终要具备的能力（验收清单）

- 能用 `ip/ss/tcpdump` 说清楚：**本机 IP/路由/邻居表**、**端口监听与连接状态**、**实际收发的报文**。
- 能定位：DNS 问题、路由问题、端口不通、TCP 建连慢、丢包/重传、MTU/分片问题、NAT/转发问题。
- 能配置：静态 IP/网关、路由、DNS、策略路由、iptables/nftables、防火墙放行、转发与 SNAT/DNAT。
- 能解释：为什么 `ping` 通但 `curl` 不通；为什么同网段不通；为什么偶发超时；为什么连接堆积；为什么 TIME_WAIT 多。
- 能把问题边界切开：应用/系统/网络/对端/云安全组/负载均衡。

---

## 1. 网络基础分层（够用即可，但要能映射到 Linux）

### 1.1 OSI/TCP-IP 分层要点
- L2：以太网、VLAN、MAC、ARP/ND（IPv6）
- L3：IPv4/IPv6、路由、ICMP
- L4：TCP/UDP
- L7：DNS、HTTP、TLS、SSH 等

### 1.2 Linux 里的对应物
- 接口/地址：`ip link`、`ip addr`
- 路由：`ip route`、`ip -6 route`
- 邻居表：`ip neigh`（ARP/ND）
- 套接字：`ss -lntup`、`ss -antp`

---

## 2. IP 与子网：地址规划与常见坑

需要掌握：
- CIDR：`/24`、`/16` 的含义，网络地址/广播地址
- 默认网关与同网段判断（为什么“以为同网段其实不是”）
- 多 IP/多网卡/多默认路由的风险
- IPv4 vs IPv6：双栈基本认知、优先级、排障思路

Linux 上建议掌握的命令：
- `ip addr show`：看 IP、前缀、是否 secondary
- `ip route`：看默认路由与直连路由

---

## 3. 二层与邻居发现：ARP/ND、交换/VLAN（够排障）

必须理解：
- ARP 解析失败会导致“同网段不通”
- ARP 缓存/老化/冲突，虚拟化/容器环境下更常见
- VLAN/Trunk/Access 的概念（云上或机房都常见）

Linux 侧：
- `ip neigh` / `arp -n`（旧工具）：看邻居条目是否 `FAILED/INCOMPLETE`
- `tcpdump -e -n arp`：抓 ARP 交互

---

## 4. 三层路由：让包“走对路”

### 4.1 基础路由
- 直连路由、默认路由、静态路由
- 路由选择（最长前缀匹配）
- 反向路径过滤 rp_filter（常导致“来得了回不去”/“多网卡诡异丢包”）

### 4.2 策略路由（进阶但很实用）
- 多出口：源地址/标记选择不同路由表（`ip rule` + `ip route add table`）

Linux 侧：
- `ip route get <dst>`：查看到某目的地址的实际路由结果
- `ip rule`：查看策略

---

## 5. 传输层：TCP/UDP（排障核心）

### 5.1 TCP 必须懂的
- 三次握手/四次挥手（会画时序图）
- 重传、超时、RTT、拥塞控制（不用背算法名，但要理解现象）
- 常见状态：`SYN-SENT`、`SYN-RECV`、`ESTAB`、`FIN-WAIT-*`、`CLOSE-WAIT`、`TIME-WAIT`
- 半连接队列/全连接队列（服务端 `SYN backlog` 与 `accept backlog`）

### 5.2 UDP 必须懂的
- 无连接、不保证到达；应用层要处理重试/超时
- 常见场景：DNS、日志、监控、游戏、部分 RPC

Linux 侧：
- `ss -antp`：看状态、队列、进程
- `ss -uapn`：看 UDP socket
- `sysctl net.ipv4.tcp_*`：相关内核参数（先会读即可）

---

## 6. 应用层必修：DNS / HTTP / TLS

### 6.1 DNS（服务器最常见问题之一）
- 递归/权威、缓存、TTL
- A/AAAA/CNAME、SRV（部分服务发现）
- `/etc/resolv.conf` 与 systemd-resolved（不同发行版差异）

排障工具：
- `dig` / `nslookup` / `resolvectl`（视系统而定）

### 6.2 HTTP 基础
- URL、Header、状态码、代理、Keep-Alive
- HTTP/1.1 vs HTTP/2（连接复用导致的观测差异）

工具：
- `curl -v`：看 DNS/TCP/TLS/HTTP 每步

### 6.3 TLS（“能定位证书/握手失败”即可）
- 证书链、SNI、协议版本、常见错误（过期、域名不匹配、链不完整）

工具：
- `openssl s_client -connect host:443 -servername host`

---

## 7. Linux 网络观测与排障工具（必须会用）

### 7.1 基础观测
- `ip`：接口/地址/路由/邻居
- `ss`：端口监听/连接/队列
- `ping` / `traceroute` / `mtr`：连通与路径
- `ethtool`：链路/速率/协商/丢包统计（物理机更常用）

### 7.2 抓包（最强通用武器）
- `tcpdump`：会筛选、会导出 pcap、会看三次握手/重传
- Wireshark（本地分析 pcap）

### 7.3 日志与内核视角
- `dmesg` / `journalctl -k`：驱动/链路相关
- `conntrack`：NAT/连接跟踪（需要时再深入）

---

## 8. 防火墙与访问控制：iptables/nftables + 云侧安全组

必须理解：
- “进不来”不一定是服务没起，也可能是防火墙/安全组/ACL
- 入站/出站、状态跟踪（ESTABLISHED/RELATED）
- DNAT/SNAT（端口映射、出网地址转换）

Linux 侧（建议至少会看）：
- `nft list ruleset` 或 `iptables -S`（视系统）
- `ufw status` / `firewall-cmd --list-all`（视发行版）

云侧（概念必会）：
- 安全组、NACL、负载均衡健康检查、源地址保留/代理协议

---

## 9. 转发、网桥与虚拟化/容器网络（服务器常见）

需要掌握：
- Linux 转发：`net.ipv4.ip_forward`
- Bridge：`br0`、二层转发
- 容器网络：veth pair、namespace、CNI（理解路径即可，不必背实现）
- 常见现象：容器能出不能入、NodePort/Ingress 访问异常、MTU 不一致

---

## 10. 性能与稳定性（入门版）

要会读现象：
- 延迟上升、抖动（jitter）、吞吐下降、丢包、重传
- 连接数暴涨、端口耗尽（ephemeral ports）、TIME_WAIT 多

Linux 侧：
- `ss -s`：总体 TCP 状态概览
- `sar -n DEV/TCP/ETCP`（如果装了 sysstat）
- 关注队列：`Recv-Q`/`Send-Q`、网卡丢包、软中断

---

## 11. 常见故障“分类诊断法”（建议背下来）

1) 解析失败（DNS）
- `dig domain` 是否正常；`/etc/resolv.conf` 是否正确

2) 路由错误（到不了/回不来）
- `ip route get <dst>`；检查多默认路由/策略路由/rp_filter

3) 端口不通（TCP 连接失败）
- 服务端 `ss -lntp` 是否监听；防火墙/安全组是否放行；`tcpdump` 看 SYN 是否到达

4) 连接慢/偶发超时
- 抓包看重传/窗口；检查 MTU；检查拥塞/丢包；看队列是否堆积

5) “ping 通但 curl 不通”
- ICMP 被允许但 TCP/443 被挡；或 DNS/HTTP/TLS 失败；用 `curl -v` 分步定位

---

## 12. 推荐学习顺序（最短路径）

1. `ip/route/neigh`（地址与路由）
2. `ss`（端口与连接状态）
3. `tcpdump`（抓包定位）
4. DNS（`dig`） + HTTP（`curl -v`）
5. 防火墙（nft/iptables）+ 云安全组
6. 策略路由 + NAT/conntrack
7. 容器网络（veth/bridge/namespace）

---

## 13. 本目录建议的后续笔记拆分（你需要我可以继续帮你生成）

- `01-ip-与-路由.md`
- `02-arp-与-邻居表.md`
- `03-tcp-状态-与-ss-实战.md`
- `04-tcpdump-抓包入门.md`
- `05-dns-排障.md`
- `06-iptables-nft-与-常见放行策略.md`
- `07-策略路由-多出口.md`
- `08-容器网络-最小理解.md`
- `09-常见故障手册.md`
