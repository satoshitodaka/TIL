# Docker
## Dockerについて
- Docker自体については読んだ記事をまとめてみた
> [Docker関連](https://github.com/satoshitodaka/TIL/tree/main/99_%E3%81%9D%E3%81%AE%E4%BB%96%E5%8B%89%E5%BC%B7%E3%83%A1%E3%83%A2/Docker%E9%96%A2%E9%80%A3)

## 実装
### 必要なファイルを作成する
```
touch {dockerfile,docker-compose.yml,endpoint.sh}
```

### Dockerfileを作る
- Dockerfileは、コンテナの作成に使われる命令を記したもの。
- `命令 引数`といった書式で記述する。
  - 命令(instruction)は大文字と小文字を区別しないが、引数と区別をつけやすくするため、慣習的に大文字にする。
- Dockerfileは命令を記述順に実行する。
- Dockerfileは必ず`FROM`から始める必要がある。
```Dockerfile
# ベースとなるイメージを指定する。RubyRuby3.1.11を使う。
FROM ruby:3.1.1

# ビルド時に渡す（設定する）変数を指定する。
ARG ROOT="/i_am_here"

# 環境変数を指定する
ENV LANG=C.UTF-8
ENV TZ=Asia/Tokyo

# コンテナ起動時の作業ディレクトリを指定する
WORKDIR ${ROOT}

# 引数にコマンドを指定しているため、シェル内で実行される。
# Linux(Ubuntu)のパッケージリストの更新を行う。
RUN apt-get update; \
  # パッケージをインストールする
  # -yオプションは、Yes/Noを聞かれると面倒な時に設定すると良い（特に今回のようなスクリプトを自動化したい時）
  # デフォルトでは推奨のパッケージ含めてインストールしてしまい時間がかかるので、推奨のパッケージはインストールしないよう指定
  apt-get install -y --no-install-recommends \
                # MariaDBを使うことを指定
                # タイムゾーンデータの指定。先に環境変数の設定で、TZがAsia/Tokyoであることを宣言している。
                mariadb-client tzdata

# ローカルのファイルを、コンテナの指定の場所にコピーする
COPY Gemfile ${ROOT}
COPY Gemfile.lock ${ROOT}

# imageの中で実行するshellコマンド
RUN gem install bundler
# bundle installに--jobs 44をつけると、並列処理をするため処理速度が速くなる。
RUN bundle install --jobs 4

COPY entrypoint.sh /usr/bin/
# コピーした/usr/bin/entrypoint.shに実行権限を付与
RUN chmod +x /usr/bin/entrypoint.sh
ENTRYPOINT [ "entrypoint.sh" ]

# コンテナを実行するポートを指定する指定する
EXPOSE 3000

# コンテナ起動時に実行するコマンド。
CMD ["rails", "server", "-b", "0.0.0.0"]
```

#### docker-compose.ymlを作成する。
- yml形式で記述し、サービスやネットワーク、ボリュームを定義する。
- サービスの定義では、各コンテナをサービスとして定義できる。
```yml
version: "3.9"
services:
  # Servicesの直下には、利用するコンテナを記述する。webはアプリケーション本体
  web:
    # buildは構築時に適用するオプションを指定する。今回はカレントディレクトリに作成する旨を指定
    build: .
    command: bash -c "rm -f tmp/pids/server.pid && bundle exec rails s -p 3000 -b '0.0.0.0'"
    # ローカルファイルの変更をコンテナに同期するという設定。ローカル:コンテナと記述する。o
    # :cachedオプションをつけると、ホスト側の情報が信頼できることになる（コンテナへの同期は遅延を許容）
    volumes:
      - .:/i_am_here:cached
      - bundle:/usr/local/bundle
    # ポートを指定する。ホスト側:コンテナ側と書く。
    ports:
      - 3000:3000
    # サービスの依存関係を指定する。
    # 依存関係を指定した（以下では(コンテナdb)が先に起動する。
    depends_on:
      - db
    stdin_open: true
    tty: true
  db:
    # コンテナのもととなるイメージを指定する。
    image: mysql:5.7.33
    # 環境変数を指定する。ここでは、config/database.ymlのデフォルトの内容を記載する。
    environment:
      MYSQL_ROOT_PASSWORD: "mysql"
      ports:
        - 3306:3306
      volumes:
        - mysql_data:/var/lib/mysql

ホストマシンの相対パスをマウントする。
volumes:
  bundle:
  mysql_data:
```
#### entrypoint.sh
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
```
コンテナのイメージを作成する。
$ docker-compose build
コンテナを作成・起動する。(-dオプションはデタッチモード)
$ docker-compose up -d
データベースを作成する。
$ docker-compose run web rails db:create
```
## 参考
- [Dockerfile リファレンス](https://docs.docker.jp/engine/reference/builder.html#dockerfile)
- [Compose ファイル・リファレンス](http://docs.docker.jp/v1.12/compose/compose-file.html)
- [重要なのに忘れがちなapt関係のコマンドのメモ - Qiita](https://qiita.com/karaage0703/items/f01db1cf49b151022b7c)
- [debianパッケージ周りでよく使うコマンドとオプション](http://blog.livedoor.jp/sonots/archives/50115456.html)
- [chmod コマンド - Qiita](https://qiita.com/ntkgcj/items/6450e25c5564ccaa1b95)
- [consistent、cached、delegated 設定のチューニング](https://docs.docker.jp/docker-for-mac/osxfs-caching.html#:~:text=%E3%81%82%E3%82%8B%E3%81%A7%E3%81%97%E3%82%87%E3%81%86%E3%80%82-,consistent%E3%80%81cached%E3%80%81delegated%20%E8%A8%AD%E5%AE%9A%E3%81%AE%E3%83%81%E3%83%A5%E3%83%BC%E3%83%8B%E3%83%B3%E3%82%B0,-%E5%B9%B8%E3%81%84%E3%81%AB%E3%82%82)