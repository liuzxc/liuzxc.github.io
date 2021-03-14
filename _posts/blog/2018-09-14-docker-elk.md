---
title: 通过 docker-compose 安装 ELK 的问题总结
tags:
  - docker-compose
  - ELK
---

> [docker-elk Git repo](https://github.com/deviantony/docker-elk)

## X-Pack support

如果需要 X-Pack 功能支持，需要选择 [docker-elk](https://github.com/deviantony/docker-elk/tree/x-pack) 的 x-pack 分支


## 镜像拉取超时

如果使用了阿里云的容器镜像服务，可以用阿里云的镜像加速器

```sh
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://yourcode.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

## failed to authenticate user [logstash_system]

创建了 logstash 的 pipline，但是数据却没有被索引到 elasticsearch


**解决方案：** 需要自己新建一个 user 和 role，分配足够的权限


## 在 Kibana UI 上新建 logstash pipeline 不生效

**解决方案：** 可以更改仓库中的配置文件，重启镜像后 pipeline 可生效

> 疑问：如何让界面配置的 logstash pipeline 可以生效呢？

## logstash 镜像不能安装 mongo-shell

添加 `USER root` 到 logstash dockerfile，然后重新 build 镜像

然后运行：

```sh
docker-compose up -d --force-recreate --build
```

## Elasticsearch health is Yellow

ELK 默认 shards 为 5，replicas 也是 5，对于单机部署，这种配置是非常消耗资源的，因此最好设置分片数为1，副本为0

```sh
curl -H "Content-Type: application/json" -XPUT http://xigua_admin:xigua123@localhost:9200/_template/filebeat -d '{
  "template": "filebeat*",
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 0
  }
}'

```

## 删除已经生成的索引

```sh
curl -X DELETE "http://localhost:9200/nginx*"
```

## 如何持久化数据？

在 `docker-compose.yml` 加入以下配置：

```sh
elasticsearch:

  volumes:
    - /path/to/storage:/usr/share/elasticsearch/data
```

## Error: Address already in use

5000 端口已经被使用：

```log
[2018-09-14T15:26:05,003][INFO ][org.logstash.beats.Server] Starting server on port: 5000
[2018-09-14T15:26:11,256][ERROR][logstash.pipeline        ] A plugin had an unrecoverable error. Will restart this plugin.
  Pipeline_id:nginx_log
  Plugin: <LogStash::Inputs::Beats port=>5000, id=>"2f86f95bf2055694fb69b6c74e35dfe71c1ad2125d68ddbdd92c28429ad222cc", enable_metric=>true, codec=><LogStash::Codecs::Plain id=>"plain_7000b63c-197c-4165-b2c2-b716246aff4e", enable_metric=>true, charset=>"UTF-8">, host=>"0.0.0.0", ssl=>false, add_hostname=>true, ssl_verify_mode=>"none", ssl_peer_metadata=>false, include_codec_tag=>true, ssl_handshake_timeout=>10000, tls_min_version=>1, tls_max_version=>1.2, cipher_suites=>["TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384", "TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384", "TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256", "TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256", "TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA384", "TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA384", "TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256", "TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA256"], client_inactivity_timeout=>60, executor_threads=>2>
  Error: Address already in use
  Exception: Java::JavaNet::BindException
  Stack: sun.nio.ch.Net.bind0(Native Method)
sun.nio.ch.Net.bind(sun/nio/ch/Net.java:433)
sun.nio.ch.Net.bind(sun/nio/ch/Net.java:425)
```

**解决方案：** 去掉默认的 pipeline.yml 中的 main pipeline


## 如何在 Dockerfile 中安装 mongo-shell?

添加以下代码到 Dockerfile:

```sh
RUN touch /etc/yum.repos.d/mongodb-org-4.0.repo

RUN echo $'[mongodb-org-4.0]\n\
name=MongoDB Repository\n\
baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/4.0/x86_64/\n\
gpgcheck=1\n\
enabled=1\n\
gpgkey=https://www.mongodb.org/static/pgp/server-4.0.asc\n'\
>> /etc/yum.repos.d/mongodb-org-4.0.repo

RUN yum install -y mongodb-org-shell
```
