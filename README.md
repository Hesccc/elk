## 1.描述
>使用docker-compose 创建elasticsearch集群、kibana、cerebro、console相关容器

## 2.环境准备

### 2.1 elasticsearch

1. 宿主机创建elastic用户和elastic用户组
```shell
# 创建elastic用户组
groupadd -g 667 elastic

# 创建elastic用户
useradd -u 666 -g elastic -d /home/elastic -c "elastic user" -s /bin/bash elastic
```
2. 创建elasticsearch节点相关目录

|   目录   |      | 使用说明      |
| ---- | ---- | ---- |
| data   | --> | 数据保存目录     |
| config | --> | 配置文件保存目录 |
| logs   | --> | 日志保存目录     |
| plugins   | --> | elasticsearch插件保存目录 |

```shell
# es01
mkdir -p /home/elastic/elk/elasticsearch/es-node01
mkdir -p /home/elastic/elk/elasticsearch/es-node01/config
mkdir -p /home/elastic/elk/elasticsearch/es-node01/data
mkdir -p /home/elastic/elk/elasticsearch/es-node01/logs
mkdir -p /home/elastic/elk/elasticsearch/es-node01/plugins

# es02
mkdir -p /home/elastic/elk/elasticsearch/es-node02
mkdir -p /home/elastic/elk/elasticsearch/es-node02/config
mkdir -p /home/elastic/elk/elasticsearch/es-node02/data
mkdir -p /home/elastic/elk/elasticsearch/es-node02/logs
mkdir -p /home/elastic/elk/elasticsearch/es-node02/plugins

# es03
mkdir -p /home/elastic/elk/elasticsearch/es-node03
mkdir -p /home/elastic/elk/elasticsearch/es-node03/config
mkdir -p /home/elastic/elk/elasticsearch/es-node03/data
mkdir -p /home/elastic/elk/elasticsearch/es-node03/logs
mkdir -p /home/elastic/elk/elasticsearch/es-node03/plugins
```
3. 修改logs/data目录权限
```shell
# logs目录
chmod 777 /home/elastic/elk/elasticsearch/es-node01/logs
chmod 777 /home/elastic/elk/elasticsearch/es-node02/logs
chmod 777 /home/elastic/elk/elasticsearch/es-node03/logs

# data目录
chmod 777 /home/elastic/elk/elasticsearch/es-node01/data
chmod 777 /home/elastic/elk/elasticsearch/es-node02/data
chmod 777 /home/elastic/elk/elasticsearch/es-node03/data
```
4. 修改/home/elastic属组和属主
```shell
chown -R elastic:elastic /home/elastic
```
5. 创建elasticsearch.yml配置文件
> es01节点：/home/elastic/elk/elasticsearch/es-node01/config/elasticsearch.yml

```yml
cluster.name: es-cluster # 集群名称，集群名称相同的节点自动组成一个集群
node.name: es01 # 节点名称
network.host: 0.0.0.0 # 同时设置bind_host和publish_host
http.port: 9200  # rest客户端连接端口
transport.tcp.port: 9300  # 集群中节点互相通信端口
node.master: true # 设置master角色
node.data: true # 设置data角色
node.ingest: true # 设置ingest角色 在索引之前，对文档进行预处理，支持pipeline管道，相当于过滤器
bootstrap.memory_lock: false
node.max_local_storage_nodes: 1

# 跨域配置
http.cors.enabled: true
# 跨域配置
http.cors.allow-origin: "*"
```

> es02节点：/home/elastic/elk/elasticsearch/es-node02/config/elasticsearch.yml

```yml
cluster.name: es-cluster # 集群名称，集群名称相同的节点自动组成一个集群
node.name: es02 # 节点名称
network.host: 0.0.0.0 # 同时设置bind_host和publish_host
http.port: 9200  # rest客户端连接端口
transport.tcp.port: 9300  # 集群中节点互相通信端口
node.master: true # 设置master角色
node.data: true # 设置data角色
node.ingest: true # 设置ingest角色 在索引之前，对文档进行预处理，支持pipeline管道，相当于过滤器
bootstrap.memory_lock: false
node.max_local_storage_nodes: 1

# 跨域配置
http.cors.enabled: true
# 跨域配置
http.cors.allow-origin: "*"
```

> es03节点：/home/elastic/elk/elasticsearch/es-node03/config/elasticsearch.yml

```yml
cluster.name: es-cluster # 集群名称，集群名称相同的节点自动组成一个集群
node.name: es03 # 节点名称
network.host: 0.0.0.0 # 同时设置bind_host和publish_host
http.port: 9200  # rest客户端连接端口
transport.tcp.port: 9300  # 集群中节点互相通信端口
node.master: true # 设置master角色
node.data: true # 设置data角色
node.ingest: true # 设置ingest角色 在索引之前，对文档进行预处理，支持pipeline管道，相当于过滤器
bootstrap.memory_lock: false
node.max_local_storage_nodes: 1

# 跨域配置
http.cors.enabled: true
# 跨域配置
http.cors.allow-origin: "*"
```

### 2.2 kibana

> 创建kibana配置文件保存目录

```shell
mkdir -p /home/elastic/elk/kibana
chown -R elastic:elastic /home/elastic/elk/kibana
```

> 创建kibana配置文件：/home/elastic/elk/kibana/kibana.yml

```shell
server.name: kibana     # kibana服务名称
server.host: "0.0.0.0"  # kibana的主机地址 0.0.0.0可表示监听所有IP
elasticsearch.hosts: [ "http://es01:9200" ] # 链接es集群地址
# elasticsearch.username: 'kibana'
# elasticsearch.password: '123456'
xpack.monitoring.ui.container.elasticsearch.enabled: true  # 显示登陆页面
i18n.locale: "zh-CN" # 开启中文模式
```

### 2.3 console

> 创建console配置文件保存目录

```shell
mkdir -p /home/elastic/elk/console/logs
mkdir -p /home/elastic/elk/console/data
```

## 3.使用方法

###　3.1 docker-compose.yml

```yml
version: '2.2'
services:
  es01:
    image: elasticsearch:7.5.2
    container_name: es01
    restart: always
    environment:
      - discovery.seed_hosts=es02,es03
      - cluster.initial_master_nodes=es01,es02,es03
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - /home/elastic/elk/elasticsearch/es-node01/data:/usr/share/elasticsearch/data
      - /home/elastic/elk/elasticsearch/es-node01/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml
      - /home/elastic/elk/elasticsearch/es-node01/logs:/usr/share/elasticsearch/logs
      - /home/elastic/elk/elasticsearch/es-node01/plugins:/usr/share/elasticsearch/plugins
    ports:
      - 9200:9200
    networks:
      - elastic
      
  es02:
    image: elasticsearch:7.5.2
    container_name: es02
    restart: always
    environment:
      - discovery.seed_hosts=es01,es03
      - cluster.initial_master_nodes=es01,es02,es03
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - /home/elastic/elk/elasticsearch/es-node02/data:/usr/share/elasticsearch/data
      - /home/elastic/elk/elasticsearch/es-node02/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml
      - /home/elastic/elk/elasticsearch/es-node02/logs:/usr/share/elasticsearch/logs
      - /home/elastic/elk/elasticsearch/es-node02/plugins:/usr/share/elasticsearch/plugins
    networks:
      - elastic

  es03:
    image: elasticsearch:7.5.2
    container_name: es03
    restart: always
    environment:
      - discovery.seed_hosts=es01,es02
      - cluster.initial_master_nodes=es01,es02,es03
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - /home/elastic/elk/elasticsearch/es-node03/data:/usr/share/elasticsearch/data
      - /home/elastic/elk/elasticsearch/es-node03/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml
      - /home/elastic/elk/elasticsearch/es-node03/logs:/usr/share/elasticsearch/logs
      - /home/elastic/elk/elasticsearch/es-node03/plugins:/usr/share/elasticsearch/plugins
    networks:
      - elastic

  kibana:
    image: kibana:7.5.2
    container_name: kibana
    restart: always
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - /home/elastic/elk/kibana/kibana.yml:/usr/share/kibana/config/kibana.yml
    privileged: true
    ports:
      - 5601:5601
    networks:
      - elastic

  cerebro:
    image: lmenezes/cerebro:latest
    container_name: cerebro
    restart: always
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - /etc/localtime:/etc/localtime
    ports:
      - 9100:9000
    networks:
      - elastic

  console:
    image: dockerproxy.com/infinilabs/console:latest
    container_name: console
    restart: always
    volumes:
      - /home/elastic/elk/console/data:/data
      - /home/elastic/elk/console/log:/log
    ports:
      - 9000:9000
    networks:
      - elastic

networks:
  elastic:
    driver: bridge
```

### 3.2 启动命令

```shell
# 测试启动
docker-compose -f /home/elastic/elk/docker-compose.yml up

# 正式启动环境，后台启动
docker-compose -f /home/elastic/elk/docker-compose.yml up -d
```