Docker中启动mysql
================================================

### 拉取命令

* docker pull mysql:5.5

### 查看镜像
* docker images

### 创建mysql容器

* docker run -id -name=mysql5.5 -p3306:3306 -e  MYSQL_ROOT_PASSWORD=123456 mysql:5.5
      -p: 代表映射端口，格式为  宿主机端口：容器中实际运行端口
      -e: 代表添加环境变量， MYSQL_ROOT_PASSWORD为设置mysqlroot账号密码

### 进入mysql容器

* 进入容器： docker exec -it mxg_mysql /bin/bash
* 登录mysql： mysql -u root -p
