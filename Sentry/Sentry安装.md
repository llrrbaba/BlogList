### Sentry安装

#### 一.CentOS安装Docker&Docker-Compose

##### 1.1 首先，为了方便添加软件源，以及支持devicemapper存储类型，安装如下软件包：

> yum update
>
> yum install -y yum-utils \
> device-mapper-persistent-data \
> lvm2

##### 1.2 添加Docker稳定版本的yum软件源：

> yum-config-manager \
> --add-repo \
> https://download.docker.com/linux/centos/docker-ce.repo

##### 1.3 之后更新yum软件源缓存，并安装Docker：

> yum update
>
> yum install -y docker-ce

##### 1.4 最后，确认Docker服务启动正常：

> systemctl start docker
>
> systemctl status docker

##### 1.5 安装Docker-Compose

> curl -L https://get.daocloud.io/docker/compose/releases/download/1.24.1/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
>
> chmod +x /usr/local/bin/docker-compose

##### 1.6 确认Docker-Compose是否安装成功

> docker-compose version

#### 二.安装Sentry

##### 2.1 拉取onpremise

> ```shell
> git clone https://github.com/getsentry/onpremise.git
> ```

##### 2.2 创建项目目录

>  docker volume create --name=sentry-data && docker volume create --name=sentry-postgres 

##### 2.3 准备环境配置文件

> ```shell
> cp .env.example .env
> ```

##### 2.4生成secret key

> ```shell
> docker-compose run --rm web config generate-secret-key
> ```

##### 2.5将上一步生成的key设置到docker-compose.yml的 SENTRY_SECRET_KEY 项

> environment:
>     SENTRY_SECRET_KEY: '3r7df#jgkf(+2v(bxmdt7m^3+#59*@xn_%0whsi72xi8ac_&o3'
>     SENTRY_MEMCACHED_HOST: memcached
>     SENTRY_REDIS_HOST: redis
>     SENTRY_POSTGRES_HOST: postgres
>     #SENTRY_EMAIL_HOST: smtp

##### 2.6  更新配置及创建超级管理员用户 

> ```shell
> docker-compose run --rm web upgrade
> ```

##### 2.7  启动服务运行 

> ```shell
> docker-compose up -d
> ```

##### 2.8  添加邮件配置到.env文件中 

> ```properties
> SENTRY_EMAIL_HOST=smtp.qq.com
> SENTRY_EMAIL_PORT=587
> SENTRY_SERVER_EMAIL=你的邮箱地址
> SENTRY_EMAIL_USER=你的邮箱地址
> SENTRY_EMAIL_PASSWORD=邮箱密码(这个不是你的邮箱密码，是smtp的客户端密码)
> SENTRY_EMAIL_USE_TLS=true   //启动TLS传输
> ```

##### 2.9重启docker

> ```shell
> docker-compose down && docker-compose up -d
> ```

##### 2.10 访问Sentry

> ip:9000

