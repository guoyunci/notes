### php
___
```
FROM php:7.3-fpm-alpine
ENV TimeZone=Asia/Shanghai \
    PHPREDIS_VERSION=redis-4.3.0 \
    SWOOLE_VERSION=swoole-4.4.0 \
    INOTIFY_VERSION=inotify-2.0.0

# Switch repositories mirror
RUN echo "http://mirrors.aliyun.com/alpine/latest-stable/main/" > /etc/apk/repositories && \
    echo "http://mirrors.aliyun.com/alpine/latest-stable/community/" >> /etc/apk/repositories

# Install dependency
RUN apk update && apk upgrade && \
    apk add --no-cache openssh-server tzdata curl vim openrc m4 autoconf make gcc g++ linux-headers nginx

# Install composer
RUN curl -sS https://getcomposer.org/installer | php \
    && mv composer.phar /usr/local/bin/composer \
    && composer self-update --clean-backups

# Switch composer mirror
RUN composer config -g repo.packagist composer https://packagist.phpcomposer.com

# Timezone
RUN ln -snf /usr/share/zoneinfo/${TimeZone} /etc/localtime && echo ${TimeZone} > /etc/timezone


RUN pecl install ${SWOOLE_VERSION} && rm -rf /tmp/pear/download/${SWOOLE_VERSION}.tgz
RUN pecl install ${PHPREDIS_VERSION} && rm -rf /tmp/pear/download/${PHPREDIS_VERSION}.tgz
RUN pecl install ${INOTIFY_VERSION} && rm -rf /tmp/pear/download/${INOTIFY_VERSION}.tgz

RUN mv "$PHP_INI_DIR/php.ini-production"  "$PHP_INI_DIR/php.ini" && docker-php-ext-install pcntl posix sysvmsg pdo_mysql
RUN docker-php-ext-enable swoole redis inotify

RUN mkdir -p /run/nginx && rm -rf /tmp/pear/cache/*

WORKDIR /var/www/html

# CMD ["nginx","-c","/etc/nginx/nginx.conf"]
```
*基于此镜像创建的容器，默认情况下php-fpm和nginx都未启动*

> 若要php-fpm后台运行，将/usr/local/etc/php-fpm.d/zz-docker.conf中的daemonize参数改为yes

*nginx支持php*
```nginx
location ~ \.php$ {
  fastcgi_pass 127.0.0.1:9000;
  fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
  include fastcgi_params;
}
```

### consul
___
```dockerfile
FROM  alpine

#配置环境变量consul版本
ENV CONSUL_VERSION=1.4.3
ENV HASHICORP_RELEASES=https://releases.hashicorp.com

#安装consul
RUN apk upgrade  && \
    apk add  net-tools && \
    apk add wget && \
    apk add unzip && \
    wget ${HASHICORP_RELEASES}/consul/${CONSUL_VERSION}/consul_${CONSUL_VERSION}_linux_amd64.zip && \
    unzip consul_${CONSUL_VERSION}_linux_amd64.zip && \
    rm -rf consul_${CONSUL_VERSION}_linux_amd64.zip && \
    mv consul /usr/local/bin


VOLUME /consul/data

#预开放端口
EXPOSE 8300
EXPOSE 8301 8301/udp 8302 8302/udp
EXPOSE 8500 8600 8600/udp
```

*创建和启动*
> docker run -itd --name cs1 --net swoftNetwork -p 8501:8500 --ip 192.168.1.12 consul
> docker exec -it cs1 sh

*进入容器后设置consul-server集群*
> consul agent -server -ui -node=cs1 -bootstrap-expect=3 -bind=192.168.1.12 -data-dir /consul/data -join=192.168.1.12 -client 0.0.0.0

*客户端容器内设置consul-client*
> consul agent -client -node=c1 -bind=192.168.1.2 -data-dir=/consul/data -join=192.168.1.12 -client 0.0.0.0 &
