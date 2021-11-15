### 1.项目目录

```bash
├── Dockerfile  # dockerFile文件
├── config-dev.yaml # dev环境配置文件
├── config-local.yaml # 本地测试文件 
├── docker-compose.yaml # docker-compose文件
├── .env # 环境变量文件
....
├── go.mod
├── go.sum
├── logs
├── main.go
```



### 2. 新增`dockerfile`

```dockerfile
FROM golang:1.16.9-stretch AS builder

# 环境变量
ENV GO111MODULE=on \
    CGO_ENABLED=0 \
    GOOS=linux \
    GOARCH=amd64 \
    GOPROXY="https://goproxy.cn,direct"

# 切换到工作目录
WORKDIR /build

# 将代码复制到容器中
COPY . .

# 下载依赖
RUN go mod download

# 编译成二进制文件,二进制文件名：app
RUN go build -o app .


### --------- 二阶段，构建一个小镜像 ---------
FROM registry.cn-beijing.aliyuncs.com/liuqinghui/go-env:stretch-slim

# 项目目录
WORKDIR /www

## 复制配置文件
ARG APP_ENV
COPY ./config-${APP_ENV}.yaml /www/config.yaml

# 从builder镜像中把二进制文件/build/app 拷贝到当前目录
COPY --from=builder /build/app /www
```

### 3. 新增`docker-compose.yaml`

```yaml
version: '3'
networks:
  backend:
    driver: ${NETWORKS_DRIVER}
services:
  app:
    build:
      context: .
      args:
        - APP_ENV=${APP_ENV}
    command: sh -c "./app -c ./config.yaml"
    volumes:
      - ${PROJECT_PATH}/config-${APP_ENV}.yaml:/www/config.yaml
      - ${LOG_PATH}/logs:/www/logs
    ports:
      - "${APP_PORT}:8088"
```

### 4. 新增环境变量文件`.env`

```properties
# 项目地址(绝对路径)
PROJECT_PATH=..
# 日志目录
LOG_PATH=
# 网络
NETWORKS_DRIVER=bridge
# 环境
APP_ENV=local
# 端口
APP_PORT=8081

```

### 5.编译`docker-compose build`

```bash
➜ docker-compose build
WARNING: Some networks were defined but are not used by any service: backend
Building app
Step 1/11 : FROM golang:1.16.9-stretch AS builder
 ---> 8a22bd5879d4
Step 2/11 : ENV GO111MODULE=on     CGO_ENABLED=0     GOOS=linux     GOARCH=amd64     GOPROXY="https://goproxy.cn,direct"
 ---> Using cache
 ---> f1201d2611b7
Step 3/11 : WORKDIR /build
 ---> Using cache
 ---> 7e157372b7c9
Step 4/11 : COPY . .
 ---> 515c90ffcd8d
Step 5/11 : RUN go mod download
 ---> Running in 0a52cfdd5444
Removing intermediate container 0a52cfdd5444
 ---> 6f5f983a8fd5
Step 6/11 : RUN go build -o app .
 ---> Running in a76302af8f8e
Removing intermediate container a76302af8f8e
 ---> 7195b4de462c
Step 7/11 : FROM registry.cn-beijing.aliyuncs.com/liuqinghui/go-env:stretch-slim
 ---> f9204dd33156
Step 8/11 : WORKDIR /www
 ---> Using cache
 ---> b87192a29265
Step 9/11 : ARG APP_ENV
 ---> Using cache
 ---> fad4cc8a171b
Step 10/11 : COPY ./config-${APP_ENV}.yaml /www/config.yaml
 ---> Using cache
 ---> bc7e2bb9dfcb
Step 11/11 : COPY --from=builder /build/app /www
 ---> 951dce0d6536
Successfully built 951dce0d6536
Successfully tagged fund-server_app:latest
```

### 6.运行`docker-compose up -d`

```bash
# 运行
➜ docker-compose up -d
WARNING: Some networks were defined but are not used by any service: backend
fund-server_app_1 is up-to-date

# 查看日志
➜ docker-compose logs
WARNING: Some networks were defined but are not used by any service: backend
Attaching to fund-server_app_1
app_1  | [GIN-debug] [WARNING] Running in "debug" mode. Switch to "release" mode in production.
app_1  |  - using env:	export GIN_MODE=release
app_1  |  - using code:	gin.SetMode(gin.ReleaseMode)
app_1  |
app_1  | [GIN-debug] GET    /system/config            --> 52lu/fund-analye-system/api/demo.GetConfig (3 handlers)
app_1  | [GIN-debug] POST   /demo/user/login          --> 52lu/fund-analye-system/api/demo.Login (3 handlers)
app_1  | 【 当前环境: dev 当前版本: v1.0.0 接口地址: http://0.0.0.0:8081 启动时间:2021-10-18 22:10:23 】
```

### 7. 踩坑记录`connect: no route to host`

启动容器后，在容器内访问本地mysql报错: `connect: no route to host`

#### 7.1 报错信息

```bash
➜ docker-compose up
WARNING: Some networks were defined but are not used by any service: backend
Starting fund-server_app_1 ... done
Attaching to fund-server_app_1
app_1  | panic: 创建mysql客户端失败: dial tcp 172.20.0.1:3306: connect: no route to host, {172.20.0.1 3306 root root fund_system utf8mb4 false true Local 255 true false {false true false false fas_ true} 50ms info true true}
app_1  |
app_1  | goroutine 1 [running]:
app_1  | 52lu/fund-analye-system/initialize.initGorm()
app_1  | 	/build/initialize/gorm.go:50 +0x6ae
app_1  | 52lu/fund-analye-system/initialize.SetLoadInit()
app_1  | 	/build/initialize/load.go:9 +0x2f
app_1  | main.main()
app_1  | 	/build/main.go:18 +0x45
fund-server_app_1 exited with code 2
```

#### 7.2 解决方法

##### 1. 替换mysql的host

```bash
# 查看网络信息
➜ docker network ls
NETWORK ID          NAME                  DRIVER              SCOPE
1ff63fe03b51        bridge                bridge              local
4067e988f21e        fund-server_default   bridge              local
3e3c8036f789        host                  host                local
5066fef7de07        none                  null                local

#  查看网络，对应的IP信息
➜  ifconfig
br-4067e988f21e: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 172.30.0.1  netmask 255.255.0.0  broadcast 172.30.255.255
        ether 02:42:09:22:b7:96  txqueuelen 0  (Ethernet)
        RX packets 341232  bytes 28039546 (26.7 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 341232  bytes 28039546 (26.7 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
# docker ip 
docker0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 172.20.0.1  netmask 255.255.255.0  broadcast 172.20.0.255
        ether 02:42:ce:98:11:2d  txqueuelen 0  (Ethernet)
        RX packets 585535  bytes 33313003 (31.7 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 605413  bytes 4440526782 (4.1 GiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.17.109.13  netmask 255.255.240.0  broadcast 172.17.111.255
        ether 00:16:3e:12:a4:49  txqueuelen 1000  (Ethernet)
        RX packets 8259415  bytes 5899125416 (5.4 GiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 6539516  bytes 2592253401 (2.4 GiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```



##### 2. 修改防火墙配置文件

`vim /etc/firewalld/zones/public.xml`

```bash
# 文件位置
<?xml version="1.0" encoding="utf-8"?>
<zone>
  <short>Public</short>
  <description>For use in public areas. You do not trust the other computers on networks to not harm your computer. Only selected incoming connections are accepted.</description>
  <service name="ssh"/>
  <service name="dhcpv6-client"/>
  <port protocol="tcp" port="20"/>
  <port protocol="tcp" port="21"/>
  <port protocol="tcp" port="22"/>
  <port protocol="tcp" port="80"/>
  <port protocol="tcp" port="8888"/>
  <port protocol="tcp" port="39000-40000"/>
  <port protocol="tcp" port="443"/>
  <port protocol="tcp" port="12345"/>
  <port protocol="udp" port="12345"/>
  <port protocol="tcp" port="888"/>

<!-- docker ip-->
<rule family="ipv4">
    <source address="172.20.0.0/16"/>
    <accept/>
</rule>
<!-- 服务创建的网络段-->
<rule family="ipv4">
    <source address="172.30.0.0/16"/>
    <accept/>
</rule>
</zone>
```

##### 3. 重启防火墙

```bash
# 重启防火墙
➜ service firewalld  restart
Redirecting to /bin/systemctl restart firewalld.service
```

##### 4. 重启服务

```bash
# 重启服务
➜  docker-compose up -d
```


