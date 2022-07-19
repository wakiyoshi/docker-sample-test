1. リポジトリをクローンする
```
git clone <リポジトリ名>
```

2. クローンしたリポジトリに移動する
```
cd <リポジトリ>
```
3.phpディレクトリのdocker image作成
docker image build -t <phpコンテナ名> ./docker/php

4.phpコンテナの起動
docker run --name <phpコンテナ名> -v /Users/megumidai(ユーザー名)/<dockerのディレクトリ名>/src(プロジェクトファイル)>:/src -d <phpコンテナ名>

5.phpコンテナの中に入ってIPアドレスを調べる
docker exec -it <phpコンテナ名>　bash
hostname -i
exit

6.nginxディレクトリのdefault.confの下記の部分を調べたIPアドレスに書き換える
fastcgi_pass <調べたIPアドレス>:9000;

7.nginxディレクトリのdocker imageを作成
docker image build -t <nginxコンテナ名> ./docker/nginx

7.nginxコンテナの起動（ポート80:80が推奨）
docker run --name <nginxコンテナ名> -v /Users/yoshidataiga/menta_dockercommand_laravel/src:/src -p 80:80 -d <nginxコンテナ名>

8.docker volumeの作成（永続化）
docker volume create db_storage(storage名)

9.mysqlコンテナの起動
docker run -d --name <mysqlコンテナ名> -e MYSQL_DATABASE=<db名> -e MYSQL_ROOT_PASSWORD=root -v db_storage:/var/lib/mysql -p 3306:3306 mysql:5.7 

10.. mysqlコンテナの中に入りIPアドレスを調べる
docker exec -it  <mysqlコンテナ名>　bash
hostname -i
exit


11.PHPコンテナに入ってLaravelの.envを編集する
docker exec -it <PHPコンテナ> bash
vi .env
DB_CONNECTION=mysql  
DB_HOST=<mysqlコンテナのIPアドレス>  
DB_PORT=3306  
DB_DATABASE=db_name  
DB_USERNAME=root  
DB_PASSWORD=root


-------------------------------

課題２.docker-composeでlaravel環境を立ち上げる。

1.docker-compose.ymlファイルをdockerのディレクトリ下（ディレクトリ名/docker-compose.yml）に作成。

2.docker-compose.ymlファイルを以下のように編集（例）、インデントは基本2つで統一する。

version: '3'
services:
  php:
    build: ./php
    volumes:
      - ../src:/var/www/html

  nginx:
    image: nginx:1.15.6
    ports:
      - 80:80
    volumes:
      - ./nginx/default.conf:/etc/nginx/conf.d/default.conf
      - ../src:/var/www/html
    depends_on:
      - php

  mysql:
    image: mysql:8.0
    ports:
      - 4306:3306
    volumes:
      - ./mysql/my.cnf:/etc/mysql/conf.d/my.cnf
      - ./mysql/initdb.d:/docker-entrypoint-initdb.d
      - ./mysql/db:/var/lib/mysql
    environment:
      MYSQL_ROOT_PASSWORD: root
      TZ: "Asia/Tokyo"


３.phpのdocker-fileの編集（例）

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

RUN php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
RUN php composer-setup.php --version=2.2.6
RUN php -r "unlink('composer-setup.php');"
RUN mv composer.phar /usr/local/bin/composer

ENV COMPOSER_ALLOW_SUPERUSER 1
ENV COMPOSER_HOME /composer
ENV PATH $PATH:/composer/vendor/bin

RUN curl -fsSL https://deb.nodesource.com/setup_16.x | bash -
RUN apt-get install -y nodejs && \
    npm i -g n && \
    n 16
RUN npm i -g --unsafe-perm node-sass
RUN npm i -g yarn lebab

WORKDIR /var/www/html


4.nginxのdefault.confの編集。

server {
    listen 80;
    root  /var/www/html/public;
    index index.php index.html;
    
    location / {
        try_files $uri $uri/ /index.php$is_args$args;
    }

    location ~ \.php$ {
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass php:9000;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $fastcgi_path_info;
    }
}

5.docker-compose up -d --buildでコンテナを起動する。
