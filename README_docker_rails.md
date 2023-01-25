# DockerでのRails7環境の構築方法

## 環境

- Ruby: 3.1
- Rails: 7.x
- MySQL: 5.7

## アプリ名の変更

テンプレートとして用意している下記のファイル内の myapp の箇所を任意のアプリ名に変更する。

- docker-compose.yml
- Dockerfile

## ポート番号の変更

ポート番号がの他のDockerコンテナのそれと重複すると動作しないので、重複しない値に変更する。

- docker-compose.yml
- Dockerfile

## コマンドの実行

### Railsアプリの生成

    $ docker-compose run web rails new . --no-deps --force --database=mysql

上記コマンドでrails newが実行されアプリ構築に必要なファイルが生成される。なお、`rails new`コマンドのforceオプションはGemfileを上書きOKとするオプション。

もし、追加のGemが必要であればGemfileにいまのうちに追加しておく。以下、追加例。

```
group :production do
  gem 'unicorn', '5.4.1'
end

gem 'slim-rails'
gem 'html2slim'

gem 'pry-rails'
```

### buildコマンドの実行

```
$ docker-compose build
```

## DB設定

自動生成された config/database.yml のパスワードを設定し、DBのホストをdb（docker-compose.ymlで指定している値）に変更する。

```
$ vi config/database.yml
# MySQL. Versions 5.1.10 and up are supported.
#
# Install the MySQL driver
#   gem install mysql2
#
# Ensure the MySQL gem is defined in your Gemfile
#   gem 'mysql2'
#
# And be sure to use new-style password hashing:
#   https://dev.mysql.com/doc/refman/5.7/en/password-hashing.html
#
default: &default
  adapter: mysql2
  encoding: utf8
  pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>
  username: root
  password: passw0rd
  host: db
  ...
```

## コンテナの起動とDBの作成

```
$ docker-compose up -d
$ docker-compose exec web rails db:create
```


## Bootstrapの導入

### 1. Gemfileに以下を追加する。

```
gem 'cssbundling-rails'
```

### 2. Bundleの実行とBootstrapの導入

```
$ docker-compose exec app bash
# bundle install
# rails css:install:bootstrap
```

### 3. package.jsonにエントリーを追加

```
sass ./app/assets/stylesheets/application.bootstrap.scss:./app/assets/builds/application.css --no-source-map --load-path=node_modules
```

追加後のファイルは以下のようになる。

```
{
  "name": "app",
  "private": "true",
  "dependencies": {
    "@popperjs/core": "^2.11.6",
    "bootstrap": "^5.2.3",
    "bootstrap-icons": "^1.10.3",
    "sass": "^1.57.1"
  },
  "scripts": {
    "build:css": "sass ./app/assets/stylesheets/application.bootstrap.scss:./app/assets/builds/application.css --no-source-map --load-path=node_modules"
  }
}
```

### 4. scssのコンパイル

```
# yarn build:css
```