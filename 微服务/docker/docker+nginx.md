Docker中启动nginx
================================================

### 查找镜像

* docker search  nginx

      [root@VM_0_10_centos ~]# docker search nginx
      NAME                              DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
      nginx                             Official build of Nginx.                        11727               [OK]                
      jwilder/nginx-proxy               Automated Nginx reverse proxy for docker con…   1628                                    [OK]
      richarvey/nginx-php-fpm           Container running Nginx + PHP-FPM capable of…   728                                     [OK]
      linuxserver/nginx                 An Nginx container, brought to you by LinuxS…   70                                      



### 拉取命令

* docker pull nginx

### 查看镜像
* docker images

      [root@VM_0_10_centos ~]# docker images
      REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
      nginx               latest              98ebf73aba75        4 days ago          109MB


### 使用 NGINX 默认的配置来启动一个 Nginx 容器实例

* docker run
      [root@VM_0_10_centos ~]# docker run -id --name=nginx -p8080:8090 -d nginx
      77ed873810f0d775fa245c14fa36ed84e950c6b9ed81a1b091fbd57dbd5a303c

### 查看启动
* docker -ps

      [root@VM_0_10_centos ~]# docker ps
      CONTAINER ID        IMAGE               COMMAND                  CREATED              STATUS              PORTS                              NAMES
      77ed873810f0        nginx               "nginx -g 'daemon of…"   About a minute ago   Up About a minute   80/tcp, 0.0.0.0:8080->8090/tcp     nginx
### nginx部署

* 创建Nginx目录，用于存放东西
      创建Nginx映射文件目录： mkdir /web/data/doc
      创建Nginx日志目录： mkdir /usr/local/configs/nginx/logs
      创建Nginx配置文件目录： mkdir /usr/local/configs/nginx/config

* 部署： $ docker run -d -p 8080:8090 --name=nginx -v /web/data/doc:/usr/share/nginx/html -v  /usr/local/configs/nginx/config/nginx.conf:/etc/nginx/nginx.conf -v /usr/local/configs/nginx/logs:/var/log/nginx nginx

      -p 8080:8090： 将容器8090端口映射到宿主机8080端口
      --name=nginx ：  指定容器名为nginx
      -v /web/data/doc:/usr/share/nginx/html ：将本地文件/web/data/doc目录映射到容器/usr/share/nginx/html
      -v  /usr/local/configs/nginx/config/nginx.conf:/etc/nginx/nginx.conf ： 将本地 /usr/local/configs/nginx/config/nginx.conf 映射到容器/etc/nginx/nginx.conf
      -v /usr/local/configs/nginx/logs:/var/log/nginx ：将本地目录/usr/local/configs/nginx/logs映射到/var/log/nginx

### 如果要重新载入 NGINX
* docker kill -s HUP container-name
* docker restart container-name
