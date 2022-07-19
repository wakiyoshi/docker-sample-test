FROM php:8.0-fpm
COPY php.ini /usr/local/etc/php/

ENV TZ=Asia/Tokyo

RUN apt-get update \
  && apt-get install -y zlib1g-dev mariadb-client vim libzip-dev \
     libfreetype6-dev \
     libjpeg62-turbo-dev \
     libpng-dev \
  && docker-php-ext-configure gd --with-freetype --with-jpeg \
  && docker-php-ext-install -j$(nproc) gd zip pdo_mysql

#Composer install
RUN php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
RUN php composer-setup.php --version=2.2.6
RUN php -r "unlink('composer-setup.php');"
RUN mv composer.phar /usr/local/bin/composer

ENV COMPOSER_ALLOW_SUPERUSER 1
ENV COMPOSER_HOME /composer
ENV PATH $PATH:/composer/vendor/bin

# Node.js install
RUN curl -fsSL https://deb.nodesource.com/setup_16.x | bash -
RUN apt-get install -y nodejs && \
    npm i -g n && \
    n 16
RUN npm i -g --unsafe-perm node-sass
RUN npm i -g yarn lebab

WORKDIR /var/www/html

