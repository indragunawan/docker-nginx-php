FROM ubuntu:xenial

LABEL maintainer="Indra Gunawan <guind.online@gmail.com>"

ENV DEBIAN_FRONTEND=noninteractive

RUN \
    apt-get update && apt-get upgrade -y \
    && apt-get install -y --no-install-recommends \
        ca-certificates \
        curl \
        git-core \
        make \
        openssh-client \
        openssl \
        unzip \
        vim \
        wget

# Nginx 1.13.2
RUN \
    wget --quiet -O - https://nginx.org/keys/nginx_signing.key | apt-key add - \
    && echo "deb http://nginx.org/packages/mainline/ubuntu/ xenial nginx" >> /etc/apt/sources.list.d/nginx.list \
    && echo "deb-src http://nginx.org/packages/mainline/ubuntu/ xenial nginx" >> /etc/apt/sources.list.d/nginx.list \
    && apt-get update && apt-get install -y --no-install-recommends nginx=1.13.2-1~xenial \

    # forward request and error logs to docker log collector
    && ln -sf /dev/stdout /var/log/nginx/access.log \
	&& ln -sf /dev/stderr /var/log/nginx/error.log

EXPOSE 80

# PHP 7.2
RUN \
    apt-get install -y --no-install-recommends software-properties-common \
    && LANG=C.UTF-8 add-apt-repository -y ppa:ondrej/php \
    && apt-get update \
    && apt-get install -y --no-install-recommends \
        php7.2-bcmath \
        php7.2-cli \
        php7.2-common \
        php7.2-curl \
        php7.2-dev \
        php7.2-fpm \
        php7.2-gd \
        php7.2-intl \
        php7.2-json \
        php7.2-mbstring \
        php7.2-mysql \
        php7.2-opcache \
        php7.2-pgsql \
        php7.2-sqlite3 \
        php7.2-xml \
        php7.2-zip \
        php-apcu \
        php-imagick \
        php-mongodb \
        php-redis \
        php-xdebug \
    # forward logs to docker log collector
    && ln -sf /dev/stdout /var/log/php7.2-fpm.log

# Config PHP and NGINX
RUN \
    mkdir -p /run/php \
    && chown root:root /run/php \
    && sed -i "s/;date.timezone =.*/date.timezone = Asia\/Jakarta/g" /etc/php/7.2/fpm/php.ini \
    && sed -i "s/;date.timezone =.*/date.timezone = Asia\/Jakarta/g" /etc/php/7.2/cli/php.ini \
    && sed -i "s/upload_max_filesize =.*/upload_max_filesize = 250M/g" /etc/php/7.2/fpm/php.ini \
    && sed -i "s/memory_limit = 128M/memory_limit = 512M/g" /etc/php/7.2/fpm/php.ini \
    && sed -i "s/post_max_size =.*/post_max_size = 250M/g" /etc/php/7.2/fpm/php.ini \
    && sed -i "s/user = www-data/user = root/g" /etc/php/7.2/fpm/pool.d/www.conf \
    && sed -i "s/group = www-data/group = root/g" /etc/php/7.2/fpm/pool.d/www.conf \
    && sed -i "s/listen.owner = www-data/listen.owner = root/g" /etc/php/7.2/fpm/pool.d/www.conf \
    && sed -i "s/listen.group = www-data/listen.group = root/g" /etc/php/7.2/fpm/pool.d/www.conf \
    && sed -i "s/pm = dynamic/pm = ondemand/g" /etc/php/7.2/fpm/pool.d/www.conf \
    && sed -i "s/pm.max_children = 5/pm.max_children = 25/g" /etc/php/7.2/fpm/pool.d/www.conf \
    && sed -i "s/;pm.process_idle_timeout = 10s;/pm.process_idle_timeout = 10s;/g" /etc/php/7.2/fpm/pool.d/www.conf \
    && sed -i "s/worker_processes 2;/worker_processes auto;/g" /etc/nginx/nginx.conf \
    && sed -i "s/listen       80;/listen       80    default_server;/g" /etc/nginx/conf.d/default.conf \

    # clear cache
    && apt-get clean \
    && apt-get autoremove --purge \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# NodeJS and NPM
RUN \
    apt-get update \
    && curl -sL https://deb.nodesource.com/setup_8.x | bash - \
    && apt-get install -y nodejs jpegoptim \
    && npm install -g npm \

    # Install Yarn
    && curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add - \
    && echo "deb https://dl.yarnpkg.com/debian/ stable main" | tee /etc/apt/sources.list.d/yarn.list \
    && apt-get update \
    && apt-get install yarn libelf1

# Project
RUN mkdir -p /home/projects
VOLUME /home/projects
WORKDIR /home/projects

# Docker Container
COPY entrypoint.sh /
ENTRYPOINT ["/entrypoint.sh"]
