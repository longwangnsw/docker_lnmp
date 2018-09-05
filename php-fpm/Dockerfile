######
# See: https://hub.docker.com/_/php/
######

FROM php:7.1.21-fpm
MAINTAINER david.wang@zerologix

######
# You can install php extensions using docker-php-ext-install
######

RUN apt-get update && \
    apt-get install -y \
    libfreetype6-dev \
    libjpeg62-turbo-dev \
    libmcrypt-dev \
#    libpng12-dev \
    gnupg \
    git \
    vim \
    mysql-client \
    redis-tools \
    build-essential 

RUN docker-php-ext-install -j$(nproc) mcrypt \
    && docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
    && docker-php-ext-install -j$(nproc) gd \
    && docker-php-ext-install mysqli pdo pdo_mysql

RUN curl -sS https://getcomposer.org/installer | php \
    && mv composer.phar /usr/local/bin/composer

Run curl -sL https://deb.nodesource.com/setup_8.x | bash \
    && apt-get install -y nodejs
