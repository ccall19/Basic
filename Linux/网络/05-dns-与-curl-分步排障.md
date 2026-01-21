# 05｜DNS + curl：把“访问失败”拆成 5 个步骤

访问一个 HTTPS URL，通常会经历：
1) DNS 解析
2) TCP 建连
3) TLS 握手
4) HTTP 请求/响应
5) 应用内容/业务逻辑

你的目标是：**快速定位卡在哪一步**。

---

## 1. 30 秒检查清单（最实用）

1) 先看 DNS 配置来源：
- `cat /etc/resolv.conf`
- systemd 系：`resolvectl status`（如果存在）

2) 用 `dig` 只验证解析：
- `dig +short A example.com`
- `dig +short AAAA example.com`

3) 用 `curl -v` 看每一步：
- `curl -v https://example.com/`

4) 跳过 DNS 直接打 IP（分离 DNS 与网络/服务）：
- `curl -v https://<ip>/ -H 'Host: example.com'`

---

## 2. `curl -v` 输出你要盯的几行

- `*   Trying <ip>:443...`：开始 TCP
- `* Connected to ...`：TCP 成功
- `* TLSv1.3 ...` / `SSL connection using ...`：TLS 成功
- `> GET / ...`：开始 HTTP
- `< HTTP/1.1 200`：HTTP 成功

快速判断：
- 卡在 `Trying`：网络/路由/防火墙/端口
- TCP 成功但 TLS 失败：证书/SNI/协议版本/中间代理
- TLS 成功但 HTTP 非 2xx：应用层/网关策略

---

## 3. DNS 常见坑（够用版）

### 3.1 解析慢/偶发失败
- DNS 服务器不可达/超时
- 多个 nameserver 顺序重试导致慢

### 3.2 IPv6 优先导致“看似随机不通”
- AAAA 有记录，但 IPv6 路由/安全组没打通

验证方式：
- 强制 IPv4：`curl -4 -v https://example.com/`
- 强制 IPv6：`curl -6 -v https://example.com/`

---

## 4. 一条命令把 DNS 与 TCP 拆开

- 只测 TCP：`nc -vz <ip> 443`（如果有 `nc`）
- 或：`curl -v https://<ip>/ -H 'Host: example.com'`

---

## 5. 一句话心法

- 访问失败不要一句话归因“网络问题”，先用 `dig` 与 `curl -v` 把步骤拆开。
