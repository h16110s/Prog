# Ruby on Railsアプリケーションの作成

## Docker for Macのインストール
[Docker for Mac](https://docs.docker.com/docker-for-mac/)からインストーラをダウンロード
ダウンロードした「Docker.dmg」を起動して，インストール


## Dockerの起動
Dockerアプリケーションを起動させる. 
*Dokerの確認*
```
$ docker --version
```

## Dockerイメージのダウンロード
今回使用するRubyとMySQLの２つのイメージをダウンロード
```
$ docker pull ruby:2.5.1
$ docker pull mysql:5.7
```
ダウンロードの際は、最新のバージョンでも「latest」とせずにバージョン指定した方がいいらしい．

## Railsアプリケーション用のディレクトリを作成
```
$ mkdir ~/works/app_name
$ cd ~/works/app_name
```

## Dockerfileの作成
イメージを作る際の命令をまとめて記載できるらしい.
```
$ vim Dockerfile
```
Dockerfileに以下を記載
```
FROM ruby:2.5.1
ENV LANG C.UTF-8
RUN apt-get update -qq && apt-get install -y build-essential libpq-dev nodejs
RUN gem install bundler
WORKDIR /tmp
ADD Gemfile Gemfile
ADD Gemfile.lock Gemfile.lock
RUN bundle install
ENV APP_HOME /app_name
RUN mkdir -p $APP_HOME
WORKDIR $APP_HOME
ADD . $APP_HOME
```
コマンドの意味は[こちら](http://docs.docker.jp/engine/reference/builder.html)から

## Gemfileの作成
```
$ vim Gemfile
```
railsをインストールするだけのコマンドを記述する
```
source 'https://rubygems.org'
gem 'rails'
```


## 空のGemfile.lockの作成
Dockerfile の構築には、空の Gemfile.lock が必要だそう。なので作成．
```
$ touch Gemfile.lock
```


## docker-compose.ymlの作成
複数のコンテナをうまいことするために定義するファイル。
今回「Rubyコンテナ」「MySQLコンテナ」２つのコンテナを作成するので必要。
```
$ vim docker-compose.yml
```
以下の内容を記述
```
version: '2'
services:
  db:
    image: mysql:5.7
    environment:
      - MYSQL_ROOT_PASSWORD=password
  web:
    build: .
    command: bundle exec rails s -p 3000 -b '0.0.0.0'
    volumes:
      - .:/app_name
    ports:
      - "3000:3000"
    depends_on:
      - db
```
今回「db」と「web」という２つのサービスを定義
詳しいことは[こちら](http://docs.docker.jp/compose/compose-file.html)で確認できる

## コンテナにRailsアプリを作成
「docker-compose run」コマンドで「web」コンテナに「rails new」を行う.
```
$ docker-compose run web rails new . --force --database=mysql --skip-bund
```
実行後、app_nameディレクトリの中にRailsのファイル群が作成されているはず．

## Gemfileの変更 
Gemfileのtherubyracer の行のコメントを外す
なければ最終行に追加
```
gem 'therubyracer', platforms: :ruby
```
Gemfileの変更を適用するには次のコマンドを実行
```
$ docker-compose build
```

## config/database.ymlの設定
configディレクトリ内のdatabase.ymlの設定を行う．
```
default: &default
  adapter: mysql2
  encoding: utf8
  pool: 5
  username: root
  password: password
  host: db
...
```
passwordをdocker-compose.ymlの「MYSQL_ROOT_PASSWORD」と同じにする
hostをdocker-compose.ymlの「depends_on」と同じにする

## コンテナの実行 & ブラウザ確認
```
$ docker-compose up
```

# 参考にしたサイト
https://qiita.com/orangeboy/items/668dea05722706a11874
