### docker相关
___

1. 创建网络

```docker
docker network create --subnet=192.168.10.0/24 redis
```

2. 查看网络详情

```docker
docker network inspect redis
```

3. 新建容器

redis(主)
```docker
docker run -itd --privileged --name master-redis --net redis --ip 192.168.10.2 -p 6379:6379 -v $HOME/Codes/docker/redis/master/conf/redis.conf:/usr/local/etc/redis/redis.conf -v $HOME/Codes/docker/redis/master/data:/data -v $HOME/Codes/docker/redis/master/log:/usr/local/etc/redis/log redis:4.0.11-alpine redis-server /usr/local/etc/redis/redis.conf --appendonly yes
```
redis(从)
```docker
docker run -itd --privileged --name slave1-redis --net redis --ip 192.168.10.3 -p 6380:6379 -v $HOME/Codes/docker/redis/slave1/conf/redis.conf:/usr/local/etc/redis/redis.conf -v $HOME/Codes/docker/redis/slave1/data:/data -v $HOME/Codes/docker/redis/slave1/log:/usr/local/etc/redis/log redis:4.0.11-alpine redis-server /usr/local/etc/redis/redis.conf --appendonly yes
```

```docker
docker run -itd --privileged --name slave2-redis --net redis --ip 192.168.10.4 -p 6381:6379 -v $HOME/Codes/docker/redis/slave2/conf/redis.conf:/usr/local/etc/redis/redis.conf -v $HOME/Codes/docker/redis/slave2/data:/data -v $HOME/Codes/docker/redis/slave2/log:/usr/local/etc/redis/log redis:4.0.11-alpine redis-server /usr/local/etc/redis/redis.conf --appendonly yes
```

php
```docker
docker run -itd --name sl1 --net redis --ip 192.168.10.10 -p 80:80 -v $HOME/Codes/php/htdocs/redis_optimize:/var/www/html sl sh
```

4. 启动
```docker
docker exec -it master-redis redis-cli
docker exec -it master-redis sh
docker exec -it sl1 sh
```

### 主从复制原理
___

* 3个阶段
1. 连接建立阶段（准备阶段）
2. 数据同步阶段
3. 命令传播阶段

* 6个过程
1. 保存主节点信息
2. 主从建立socket连接
3. 发送ping命令
4. 权限验证
5. 同步数据集
6. 命令持续复制

### 主从复制常用相关配置
___

* 从库配置
1. slaveof
2. masterauth
3. slave-read-only 默认为yes.
4. slave-serve-stale-data 主从断开连接, slave是否继续应答client请求. 默认为yes.
5. slave-priority 默认为100, master宕机, 哨兵将此值最小的slave提升为master. 值为0, 不会被提升为master.

* 主库配置
1. repl-disable-tcp-nodelay 默认为no, 降低同步延迟
2. repl-ping-slave-period 10
3. repl-backlog-size 1mb
4. repl-backlog-ttl 3600
5. min-slaves-to-write 3
6. min-slaves-max-lag 10



### tc模拟网络延迟和丢包
___

* alpine中安装iproute2
> apk add iproute2

* 配置延迟5s
```shell
tc qdisc add dev eth0 root netem delay 5000ms
```

* 删除配置
```shell
tc qdisc del dev eth0 root netem delay 5000ms
```
