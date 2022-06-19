# 01 rubocopとrspecの導入とCI（GitHub Actions）の設定
## 課題
- 以下を導入する
  - rubocop
  - rubocop-rails
  - rspec-rails
  - factory_bot_rails
  - faker
- rspecとrubocopが正しく動くように設定してください。
- またCI(GitHub Actions)でrubocopとrspecが動くような設定もしてください。
- `/`にアクセスするとテストですという文言が表示されることがテストとして書かれていればOKです。rubocopも全て通しましょう。
- .rubocop.ymlは [こちら](https://github.com/DaichiSaito/insta_clone_ver7/pull/2/files#diff-4f894049af3375c2bd4e608f546f8d4a0eed95464efcdea850993200db9fef5c)をそのまま使ってください。

## 開発メモ
### アプリの作成
- ディレクトリを作り、Railsアプリケーションを作成する。
- rails newのときのオプションは以下の通り
  - --database=mysql
  - --skip-test
  - --skip-bundle
  - --skip-turbolink
```
mkdir i_am_here
rails _7.0.3_ new . i_am_here --database=mysql --skip-test --skip-turbolink --skip-bundle  
```
- importmapを有効にすると、各種ファイルが生成される。
```
$ rails importmap:install
$ rails turbo:install
$ rails stimulus:instal
```


### rubocopの導入
- Gemfileに記載してbundle installする
```
group :development do
  gem 'rubocop'
  gem 'rubocop-rails', require: false
  gem "web-console"
end
```




### Dockerの導入
- 必要なファイルを作成する
  - Dockerfile
  - docker-compose.yml
  - Gemfile
  - 

#### Dockerfileを作成する。
```
FROM ruby:3.1.1
ARG ROOT='/insta_clone'
ENV LANG=C.UTF-8
ENV TZ=Asia/Tokyo

WORKDIR ${ROOT}

RUN apt-get update; \
  apt-get install -y --no-install-recommends \
                mariadb-client tzdata

COPY Gemfile ${ROOT}
COPY Gemfile.lock ${ROOT}
RUN gem install bundler
RUN bundle install --jobs 4

COPY entrypoint.sh /usr/bin/
RUN chmod +x /usr/bin/entrypoint.sh
ENTRYPOINT [ "entrypoint.sh" ]
EXPOSE 3000

CMD ["rails", "server", "-b", "0.0.0.0"]
```

#### docker-compose.ymlを作る
```yml
version: "3.9"
services:
  web:
    build: .
    command: bash -c "rm -f tmp/pids/server.pid && bundle exec rails s -p 3000 -b '0.0.0.0'"
    volumes:
      - .:/insta_clone:cached
      - bundle:/usr/local/bundle
    ports:
      3000:3000
    depends_on:
      - db
    stdin_open: true
  db:
    image: mysql:5.7.33
    environment:
      MYSQL_ROOT_PASSWORD: "mysql"
    ports:
      - 3306: 3306
    volumes:
      - mysql_data:/var/lib/mysql

volumes:
  bundle:
  mysql_data:
```

#### endpoint.shを作成する
```sh
#!/bin/bash
# エラーが発生するとスクリプトを停止する
set -e

# Remove a potentially pre-existing server.pid for Rails.
rm -rf /insta_clone/tmp/pids/server.pid

# Then exec the container's main process (what's set as CMD in the Dockerfile).
# DockerfileのCMDを実行する。
exec "$@"
```
### Dockerの操作
- コンテナのイメージを作成する。
```
docker-compose build
```
- コンテナを作成・起動する。
```
docker-compose up -d
```
- データベースを作成する。
```
docker-compose run web rails db:create
```
## 参考
- [Docker ドキュメント日本語化プロジェクト](https://docs.docker.jp/index.html)
- [Docker超入門講座 合併版 - Youtube](https://www.youtube.com/watch?v=lZD1MIHwMBY)
- [ソフトウェアエンジニアの「Docker よくわからない」を終わりにする本]()