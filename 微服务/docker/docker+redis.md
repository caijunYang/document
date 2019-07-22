Docker中启动redis
================================================


### 查找镜像

[root@VM_0_10_centos ~]# docker search  redis

      NAME                             DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
      redis                            Redis is an open source key-value store that…   7129                [OK]                
      bitnami/redis                    Bitnami Redis Docker Image                      120                                     [OK]
      sameersbn/redis                                                                  76                                      [OK]
      grokzen/redis-cluster            Redis cluster 3.0, 3.2, 4.0 & 5.0               52                                      
      kubeguide/redis-master           redis-master with "Hello World!"                29                                      


### 拉取命令

* docker pull redis

### 查看镜像
* docker images

      [root@VM_0_10_centos ~]# docker images
      REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
      redis               latest              598a6f110d01        10 days ago         118MB



### 创建mysql容器
* docker run
      [root@VM_0_10_centos ~]# docker run -p 6379:6379 -d redis:latest redis-server
      [root@VM_0_10_centos ~]# docker run -p 6379:6379 -v $PWD/data:/data  -d redis:3.2 redis-server --appendonly yes

      命令说明：
      -p 6379:6379 : 将容器的6379端口映射到主机的6379端口
      -v $PWD/data:/data : 将主机中当前目录下的data挂载到容器的/data
      redis-server --appendonly yes : 在容器执行redis-server启动命令，并打开redis持久化配置
### 查看启动
* docker -ps

      [root@VM_0_10_centos ~]# docker ps
      CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                              NAMES
      6448eecf093d        redis:latest        "docker-entrypoint.s…"   3 days ago          Up 3 days           6379/tcp, 0.0.0.0:6380->6380/tcp   redis
