# README

**文件夹内的所有文件不要重命名**

### 构建镜像

```
docker build -t  镜像名:镜像版本 .
```

最后的一个点 .  不能省略

### 查看镜像

```
docker images
```

### 启动一个容器

```
docker run -id -p 8080:80 -p 3301:3306 -e MYSQL_USER_DB='public_busi' -e MYSQL_ROOT_PWD='1qaz!QAZ' --name 容器名 镜像名:镜像版本
```

public_busi：公共数据库名，代码中已经写死，不可更改

1qaz!QAZ：root用户密码，代码中已经写死，不可更改

8080.3301：nginx端口和数据库端口，数据库端口没有使用，可以自定义。nginx端口与前端一致即可

### 查看容器

```
docker ps
```

如果以上命令查询不到，说明容器启动失败

查看全部容器（包括已停止的）

```
docker ps -a
```

### 查看容器启动日志

```
docker logs 容器名
```

### 进入容器

```
docker exec -it 容器名 /bin/sh
```

因为是精简系统，很多在标准系统中的命令被删减，会提示不存在。常用命令都有

### 执行数据库脚本

脚本位置：/scripts/db.sql

执行方式与mysql一致。这个步骤可以优化，在容器启动时自动执行。

### 启动java服务

jar包位置：/mnt/jar/

启动：

```
java -jar /mnt/jar/public-service.jar  --spring.redis.port=6379 --spring.redis.password=root &
```

如果redis密码是root，端口是6379 ，启动参数可以不配置

### 查看日志

日志位置：/mnt/logs/

### 退出

exit

### **重点**

基础镜像已经提交至docker hub。直接拉取即可

```
docker pull steven0921/service:3.0
```

仓库名和镜像名会改变，此处只是展示命令使用方法。

接下来可以从启动容器开始。

### 优化

- 数据库脚本自动执行

- java服务自动启动

- redis配置可以启动时指定

- redis打到容器中

  

**冲突：将所有服务打包在一起是docker不推荐的一种方式，正确做法是一个容器只管理一种服务**