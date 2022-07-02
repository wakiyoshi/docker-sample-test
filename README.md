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


