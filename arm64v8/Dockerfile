FROM arm64v8/ubuntu:bionic AS base

# Set Locales 

RUN apt update
RUN apt install -y locales && rm -rf /var/lib/apt/lists/* \
    && localedef -i en_US -c -f UTF-8 -A /usr/share/locale/locale.alias en_US.UTF-8
ENV LANG en_US.utf8

RUN apt-get update 
RUN apt-get install -y \
    net-tools \
    build-essential \
    wget \
    pkg-config \
    git \
    autoconf \
    bison \
    libtool \
    re2c \
    sqlite3 \
    libsqlite3-0 \
    vim \
    curl \
    libssl-dev \
    libcurl4-openssl-dev \
    libxml2-dev \
    libreadline-dev \
    libzip-dev \
    nginx \
    openssl \
    curl \
    zlib1g-dev \
    unzip \
    libsqlite3-dev \
    libonig-dev

# Build PHP from source
FROM base AS build-php
WORKDIR /opt
RUN wget https://www.php.net/distributions/php-8.0.11.tar.xz 
RUN tar xf php-8.0.11.tar.xz 
WORKDIR /opt/php-8.0.11
RUN ./configure \
    --enable-fpm \
    --with-openssl \
    --with-zlib \
    --enable-mbstring \
    --enable-opcache

RUN make && make install
RUN cp php.ini-production /usr/local/lib/php.ini
RUN cp /usr/local/etc/php-fpm.conf.default /usr/local/etc/php-fpm.conf
RUN cp /usr/local/etc/php-fpm.d/www.conf.default /usr/local/etc/php-fpm.d/www.conf 


# Config php-fpm
FROM build-php AS config-php
WORKDIR /usr/local/etc/php-fpm.d
RUN sed -i 's/user = nobody/user = www-data/g' www.conf
RUN sed -i 's/group = nobody/group = www-data/g' www.conf
RUN sed -i 's/;listen.owner = nobody/listen.owner = www-data/g' www.conf
RUN sed -i 's/;listen.group = www-data/listen.group = www-data/g' www.conf
RUN sed -i 's/pm = dynamic/pm = static/g' www.conf
RUN sed -i 's/pm.max_children = 5/pm.max_children = 2/g' www.conf
RUN sed -i 's/pm.start_servers = 2/pm.start_servers = 2/g' www.conf
RUN sed -i 's/;pm.status_path = \/status/pm.status_path = \/status/g' www.conf
RUN sed -i 's/;ping.path = \/ping/ping.path = \/ping/g' www.conf
RUN sed -i 's/listen = 127.0.0.1:9000/listen = \/var\/run\/php-fpm.sock/g' www.conf

WORKDIR /usr/local/etc
# Remove none from include path include=NONE/etc/php-fpm.d/*.conf
RUN sed -i 's/NONE\///g' php-fpm.conf

# Testify php-fpm config
RUN /usr/local/sbin/php-fpm -t

# Ignore the config php and Opcache for now
WORKDIR /usr/local/lib/
RUN sed -i 's/;opcache.enable=1/opcache.enable=1/g' php.ini
RUN sed -i 's/;opcache.enable_cli=0/opcache.enable_cli=0/g' php.ini
RUN sed -i 's/;opcache.memory_consumption=128/opcache.memory_consumption=128/g' php.ini
RUN sed -i 's/;opcache.interned_strings_buffer=8/opcache.interned_strings_buffer=8/g' php.ini
RUN sed -i 's/;opcache.max_accelerated_files=10000/opcache.max_accelerated_files=10000/g' php.ini


# Config nginx
FROM config-php AS config-nginx
WORKDIR /etc/nginx
# Remove try_files section to avoid php-fpm status/ping 404 error
# RUN sed -i 's/try_files/# try_files/g' /etc/nginx/snippets/fastcgi-php.conf
RUN cp sites-available/default sites-available/default.backup
COPY ./nginx.conf /etc/nginx/sites-available/default 
# Testify nginx conf
RUN /usr/sbin/nginx -t

# Set index php
FROM config-nginx AS alpha
WORKDIR /var/www
RUN mkdir php
# touch status and php file to avoid php-fpm status/ping 404 error
RUN touch php/status
RUN touch php/ping

COPY ./phpinfo.php /var/www/php/index.php
RUN chown -R www-data:www-data /var/www/php/*

COPY ./entrypoint.sh /entrypoint.sh

RUN chmod +x /entrypoint.sh

ENTRYPOINT ["/entrypoint.sh"]
