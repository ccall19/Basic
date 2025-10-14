# TCP 三次握手

用来 向对方 证明双方均 具有 收发能力：  
服务端处于监听状态 

- 客户端 syn 请求发起 连接 —》 cli 向 ser 证明自己 可以 发  
- 服务端返回 ack + syn -〉 ser 向 cli 证明自己 可以 收发  
- 客户端 返回 ack -》 cli 向 ser 证明自己 可以 收  

![alt text](image.png)