# 查看Redis配置
CONFIG GET 命令的基本语法:
````sh
redis 127.0.0.1:6379> CONFIG GET CONFIG_SETTING_NAME
````
查看日志等级的配置
````sh
redis 127.0.0.1:6379> CONFIG GET loglevel
````
# 编辑修改Redis配置
可以直接编辑 redis.conf 文件，也可以通过 CONFIG set 命令更新配置。  
CONFIG SET 命令的基本语法:
````sh
redis 127.0.0.1:6379> CONFIG SET CONFIG_SETTING_NAME NEW_CONFIG_VALUE
````
比如修改日志级别：
````sh
127.0.0.1:6379> CONFIG SET loglevel "verbose"
````
## Redis 的日志级别有以下四种：
- debug：会打印出很多信息，适用于开发和测试阶段。
- verbose（冗长的）：包含很多不太有用的信息，但比debug简化一些。
- notice：适用于生产模式。
- warning : 警告信息。

Redis 默认设置为 verbose，开发测试阶段可以用 debug，生产模式一般选用 notice。

# Redis 常用配置参数说明
https://redis.com.cn/redis-configuration.html