Docker中启动es+head+ik
================================================

### 拉取命令

* docker pull elasticsearch:7.7.0
### 创建用户自定义网络

* docker network create es_network

### 查看镜像
* docker images

### 创建mysql容器


* 集群启动，并带参数：

      docker run -e ES_JAVA_OPTS="-Xms1024m -Xms1024m" -d -p 9200:9200 -p 9300:9300 --restart=always -v /docker/es/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml --name es elasticsearch:7.7.0

* 集群启动不带参数：
      docker run -e ES_JAVA_OPTS="-Xms1024m -Xms1024m" -net es_network -d -p 9200:9200 -p 9300:9300 --restart=always  --name es elasticsearch:7.7.0
* 单机启动：
      docker run -d --name es -p 9200:9200 -p 9300:9300  --restart=always -v /docker/es/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml -e ES_JAVA_OPTS="-Xms512m -Xmx512m" -e "discovery.type=single-node"  elasticsearch:7.7.0
* 启动参数说明：
      -p: 代表映射端口，格式为  宿主机端口：容器中实际运行端口
      -e: 代表添加环境变量， MYSQL_ROOT_PASSWORD为设置mysqlroot账号密码
      -v：将docker镜像中的文件映射到宿主机器指定的文件，可以是文件夹，-v 宿主机文件:容器文件映射后可直接修改宿主机上的文件就可以改变docker中的配置，也可写多个。docker镜像中软件的配置文档默认在/usr/share”/{软件名}下

### 进入es容器

      docker exec -it es /bin/bash

### 安装ES Head

* 拉取

      docker pull mobz/elasticsearch-head:5

* 启动head

      docker run -d --name es_head -p 9100:9100 mobz/elasticsearch-head:5

* 浏览器访问 ip:9100

* 访问但是连不上ES的情况，坑你是因为跨域

    http.cors.enabled: true
    http.cors.allow-origin: "* "


### 安装ik中文分词器

* 1、进入es容器内部，/plugins下新建ik文件夹

      docker exec -it es /bin/bash
      mkdir ik
      cd ik

* 2、下载与es对应版本的ik压缩包，并解压  
       wget https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v7.1.1/elasticsearch-analysis-ik-7.1.1.zip

       unzip elasticsearch-analysis-ik-7.1.1.zip

* 3、退出容器，重启es容器
      exit
      docker restart es

* 4、发起请求,请求以下参数
      ip:9200/_analyze?pretty=true
      {
      "analyzer": "ik_max_word",
      "text": "这是我拷贝来的,我是不是很厉害"
      }
