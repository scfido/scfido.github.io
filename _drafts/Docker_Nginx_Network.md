# Docker环境使用Nginx代理

## 创建网络

```sh
docker network create webnet
```
## 启动 nginx

```sh
docker run \
-d \
-p 80:80 \
-v /root/apps/nginx/conf.d:/etc/nginx/conf.d \
-v /root/apps/nginx/nginx.conf:/etc/nginx/nginx.conf \
--restart always \
--network webnet \
--name webnet_nginx \
nginx
```

## Mantis

### 启动 mysql

```sh
docker run \
-d \
-e MYSQL_ROOT_PASSWORD=root \
-e MYSQL_DATABASE=bugtracker \
-e MYSQL_USER=mantisbt \
-e MYSQL_PASSWORD=mantisbt \
-v /root/apps/mantis/data:/var/lib/mysql \
--network webnet \
--restart always \
--name mantis_mysql \
mysql:8.0.19 \
--character-set-server=utf8mb4 \
--collation-server=utf8mb4_unicode_ci
```

### 启动 mantis

```sh
docker run \
-d \
-v /root/apps/mantis/config/config_inc.php:/var/www/html/config/config_inc.php \
-v /root/apps/mantis/config/config_defaults_inc.php:/var/www/html/config/config_defaults_inc.php \
--restart always \
--network webnet \
--name mantis_app \
registry.cn-hangzhou.aliyuncs.com/xunmei/mantisbt:2.23.0
```

### 还原数据库

```sh
docker cp mantisbt.sql mantis_mysql:/root/
docker exec -it mantis_mysql bash
cd root
mysql -umantisbt -p --default-character-set=utf8mb4 bugtracker < ./mantisbt.sql
```

## Joomla

### 启动 Mysql

```
docker run \
-d \
-e MYSQL_ROOT_PASSWORD=root \
-e MYSQL_DATABASE=joomla \
-e MYSQL_USER=joomla \
-e MYSQL_PASSWORD=joomla \
-v /root/apps/joomla/data:/var/lib/mysql \
--network webnet \
--restart always \
--name joomla_mysql \
mysql:5.7 \
--character-set-server=utf8mb4 \
--collation-server=utf8mb4_unicode_ci
```

### 启动 Joomla

```
docker run  \
-d \
-e JOOMLA_DB_HOST=joomla_mysql \
-e JOOMLA_DB_USER=joomla \
-e JOOMLA_DB_PASSWORD=joomla \
-e JOOMLA_DB_NAME=joomla \
--restart always \
--name joomla_app \
--network webnet  \
joomla
```

### 修改上传最大文件限制

- PHP

```sh
docker exec -it joomla_app bash
cd /usr/local/etc/php/conf.d

# 配置uploads.ini
cat > uploads.ini << EOF
file_uploads = On
memory_limit = 500M
upload_max_filesize = 100M
post_max_size = 100M
max_execution_time = 600
EOF
```

- Nginx

修改服务器Nginx配置文件`/root/apps/nginx/conf.d/joomla.conf`的`client_max_body_size`参数。

```
server {
    listen       80;
    server_name  joomla.xunmei.com;

    #charset koi8-r;
    #access_log  /var/log/nginx/host.access.log  main;

    location / {
        proxy_set_header Host $host:$server_port;
        proxy_pass http://joomla_app;
        proxy_set_header X-Forwarded-For $remote_addr;
        client_max_body_size 50m;
    }
```

## Orchard

```sh
docker run  \
-d \
--restart always \
--name orchard_app \
--network webnet  \
orchardproject/orchardcore-cms-linux:latest
```

