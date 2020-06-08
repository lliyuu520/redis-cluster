# 使用Docker构建Redis集群

1. 基于Redis6.0.4构建,从官网下载基础配置文件,链接 https://raw.githubusercontent.com/antirez/redis/6.0/redis.conf 
2. 根据官网推荐,至少3主3从,复制redis.conf为6份,分别为 redis-7001.conf,redis-7002.conf,redis-7003.conf,redis-7004.conf,redis-7005.conf,redis-7006.conf
3. Redis默认采用后台启动,与Docker启动方式冲突,将Redis的后台启动注释
    `````
    #protected-mode yes
    `````

4. 配置连接密码
    ```
    requirepass redis123 #连接密码
    ```
5. 配置集群
    ```` 
    # 集群配置 以下配置按redis.conf分别配置
    masterauth redis123 # 集群连接密码
    cluster-config-file nodes-7001.conf # 配置文件
    cluster-node-timeout 15000 # 节点超时时间
    cluster-announce-ip 192.168.0.134 #docker主机ip
    cluster-announce-port 7001 # 端口
    cluster-announce-bus-port 17001 # 集群bus端口,用配置端口+1000
    ````
6. 编写Dockerfile
    ````
    FROM redis:6.0.4-alpine
    COPY redis-7006.conf /etc/redis/
    ENV TZ Asia/Shanghai
    EXPOSE 6379 16379
    CMD [ "redis-server", "/etc/redis/redis-7006.conf" ]
    ````
 7. 编写docker-compose.yml
    ````
    version: "3.7"
    services:
      redis-7001:
        container_name: redis-7001
        build: redis-7001
        hostname: redis-7001
        privileged: true
        ports:
          - "7001:6379"
          - "17001:16379"
        networks:
          - network
        volumes:
          - redis-7001:/data/
      redis-7002:
        container_name: redis-7002
        build: redis-7002
        hostname: redis-7002
        privileged: true
        ports:
          - "7002:6379"
          - "17002:16379"
        networks:
          - network
        volumes:
          - redis-7002:/data/
      redis-7003:
        container_name: redis-7003
        build: redis-7003
        hostname: redis-7003
        privileged: true
        ports:
          - "7003:6379"
          - "17003:16379"
        networks:
          - network
        volumes:
          - redis-7003:/data/
      redis-7004:
        container_name: redis-7004
        build: redis-7004
        hostname: redis-7004
        privileged: true
        ports:
          - "7004:6379"
          - "17004:16379"
        networks:
          - network
        volumes:
          - redis-7004:/data/
      redis-7005:
        container_name: redis-7005
        build: redis-7005
        hostname: redis-7005
        privileged: true
        ports:
          - "7005:6379"
          - "17005:16379"
        networks:
          - network
        volumes:
          - redis-7005:/data/
      redis-7006:
        container_name: redis-7006
        build: redis-7006
        hostname: redis-7006
        privileged: true
        ports:
          - "7006:6379"
          - "17006:16379"
        networks:
          - network
        volumes:
          - redis-7006:/data/
    networks:
      network:
        driver: bridge
        name: redis-cluster
    volumes:
      redis-7001: {}
      redis-7002: {}
      redis-7003: {}
      redis-7004: {}
      redis-7005: {}
      redis-7006: {}
    ````
8. 启动集群
    ````
    $ docker-compose up -d
    ````
    此时单节点都已启动,但未配置集群
9. 配置集群,进入任意一个节点
    ```bash
    $ docker exec -it redis-7001 /bin/bash
    ```
    集群配置
    ````bash
    $ redis-cli -a redis123 --cluster create 192.168.0.134:7001 192.168.0.134:7002 192.168.0.134:7003 192.168.0.134:7004 192.168.0.134:7005 192.168.0.134:7006 --cluster-replicas 1
    ````
    `--cluster-replicas 1` 表示一个主至少一个从
10. 集群部署完成 项目源码 