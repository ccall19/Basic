# 06｜防火墙放行速查：nftables/iptables + 云安全组

“端口不通”排障时，经常是：服务监听正常，但**本机防火墙**或**云安全组**挡住了。

---

## 1. 30 秒检查清单

1) 服务是否监听：
- `ss -lntp '( sport = :PORT )'`

2) 云上先看安全组/ACL（很多时候不是 Linux 的锅）

3) Linux 本机看防火墙体系：
- 优先尝试：`nft list ruleset`（新系统）
- 或：`iptables -S`（老系统/兼容层）
- RHEL 系可能是 firewalld：`firewall-cmd --state`、`firewall-cmd --list-all`
- Ubuntu 常见 ufw：`ufw status`

---

## 2. 你需要的最小认知

- **入站**：别人打你服务器的端口
- **出站**：你服务器访问别人
- **状态跟踪**：已建立连接的回包通常需要允许 `ESTABLISHED,RELATED`

如果你只是想“让 443 能进来”，一般只需要放行入站 TCP/443。

---

## 3. nftables 快速查看（不改配置先会看）

- `nft list ruleset`

你重点找：
- `table inet filter` 或类似
- `chain input` 里是否有 `tcp dport 443 accept`
- 有没有默认 `drop`（最后一条或策略）

---

## 4. iptables 快速查看（不改配置先会看）

- `iptables -S`
- 看计数（如果有）：`iptables -nvL`

重点看：
- INPUT 链策略是 ACCEPT 还是 DROP
- 是否有针对端口的 DROP

---

## 5. 安全提示（很重要）

- 不建议线上直接“全放开”
- 放行时最好限制来源网段（如果业务允许）
- 变更前记录现状：把 `nft list ruleset` 或 `iptables -S` 输出保存到文件

---

## 6. 一句话心法

- 端口不通：先 `ss` 确认监听，再确认云安全组，最后看本机 nft/iptables。
