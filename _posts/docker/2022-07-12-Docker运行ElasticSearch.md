---
   layout: default
   title: 两个容器网络互通
   tags: 
   - Docker
   - ElasticSearch
---
# {{ page.title }}
{{ page.date | date: "%Y-%m-%d" }}

# 启动

```sh
# 创建网络
docker network create es

# 启动elasticsearch
docker pull elasticsearch:8.3.2
docker run -d --name elasticsearch -p9200:9200 -p9300:9300 -e ES_JAVA_OPTS="Xms512m -Xmx512m" -e "discovery.type=single-node" --net es elasticsearch:8.3.2
docker exec --it elasticsearch bash
elasticsearch-reset-password -u elastic -i # 设置elastic的密码

# 启动kibana
docker pull kibana:8.3.2
docker run -d --name kibana --net es -p5601:5601 kibana:8.3.2

```
# 登录Kibana

使用浏览器访问http://localhost:5601，此时需要输入Enrollment token

通过一下命令获取token

```sh
docker exec -it elasticsearch bash
./bin/elasticsearch-create-enrollment-token -s kibana
```

然后在获取kibana验证码即可。

# 配置Kibana HTTPS

## 生成Kibana https证书

```sh
docker exec --it elasticsearch bash

# 生成https://elasticsearch https://localhost两个域名的证书
elasticsearch-certutil csr -name kibana-server -dns elasticsearch, localhost

# 输出 csr-bundle.zip 使用如下的命令来进行解压缩：
unzip csr-bundle.zip 
# Archive:  csr-bundle.zip
#    creating: kibana-server/
#   inflating: kibana-server/kibana-server.csr  
#   inflating: kibana-server/kibana-server.key  

pwd
# /usr/share/elasticsearch

# 在host中将elasticsearch容器中的证书复制到kibana容器中
exit
# 从elasticsearch容器复制到本机
cp elasticsearch:/usr/share/elasticsearch/kibana-server/kibana-server.key ./
cp elasticsearch:/usr/share/elasticsearch/kibana-server/kibana-server.csr ./

# 从本机复制到kibana容器
cp ./kibana-server.key kibana:/usr/share/kibana/config
cp ./kibana-server.csr kibana:/usr/share/kibana/config
```

我们接下来使用如下的命令来生成 kibana-server.crt 文件

```
docker exec -it kibana bash
cd /usr/share/kibana/config
openssl x509 -req -in kibana-server.csr -signkey kibana-server.key -out kibana-server.crt
```

## 修改Kibana配置

```sh
# 从容器中复制到本机修改
docker cp kibana:/usr/share/kibana/config/kibana.yml ./ 

# 使用本机VS Code编辑
code kibana.yml 

# 修改完后复制到容器中
docker cp kibanan.yml kibana:/usr/share/kibana/config/  

docker restart kibanan
```



## 配置文件修改内容

```yaml

# 添加以下内容
server.ssl.certificate: config/kibana-server.crt
server.ssl.key: config/kibana-server.key
server.ssl.enabled: true

```

# 参考

https://elasticstack.blog.csdn.net/article/details/122936411
https://blog.csdn.net/UbuntuTouch/article/details/122946268