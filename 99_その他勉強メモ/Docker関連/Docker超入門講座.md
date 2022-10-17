# Docker超入門講座
 [Docker超入門講座 合併版](https://www.youtube.com/watch?v=lZD1MIHwMBY)

## Dockerfileを作る

## Docker ComposeでRailsを構築する
```yml
version: '3'

# 二つのサービスを定義
services:
  db:
    # MySQL8.0のイメージを指定
    image: mysql:8.0
    # MySQL8.0より認証が変わったことに関するコマンド
    command: --default-authentication-plugin=mysql_native_password
    # 左がローカルディレクトリ、右でDockerでのディレクトリを指定している。
    # ローカルのデータをDocker側に同期するという設定。コンテナを作り直す時に自動で同期してくれるので便利
    volumes:
      - ./src/db/mysql_data:/var/lib/mysql
    # MySQLのパスワードを環境変数に設定する
    environment:
      MYSQL_ROOT_PASSWORD: password
  web:
    # docker-compose.ymlが置いてあるディレクトリにあるDockerfileを使用するという意味
    build: .
    # Railsのサーバー起動コマンド。ポートを3000に指定し、IPアドレスは任意のものを受け付ける。
    command: bundle exec rails s -p 3000 -b '0.0.0.0'
    ローカルのファイル共有。./srcの内容をDockerの/appに共有する。
    volumes:
      - ./src:/app
    # ポートを指定する。ローカルの3000ポートがDockerの3000ポートになる。
    ports:
      - "3000:3000"
    # webサービスはDBに依存する旨。アプリがMySQLに接続する際。下記を指定しないと、IPアドレスなど接続情報を明記する必要がある。
    depends_on:
      - db
```
## 本番環境に公開
### 事前準備
- Herokuにログインしてクレジットカードの登録が必要
- Heroku CLIを導入する
```
brew tap heroku/brew && brew install heroku
```
- Heroku CLIでターミナルからログイン
```
heroku login
```
- Dockerコンテナをデプロイするためには、Heroku Container Registryにログインする必要がある。
``` 
heroku container:login
```
> [Container Registry および Runtime (Docker デプロイ)](https://devcenter.heroku.com/ja/articles/container-registry-and-runtime)
### herokuでアプリを作成する
```
heroku create rails_docker_practice_satoshi
```
- DBとして、アドオンのClearDBのigniteプランを使用する旨を指定
```
heroku addons:create cleardb:ignite -a rails-docker-practice-satoshi
```
- DBに接続するための環境変数を設定する。
  - 今回取り組んだ中では、変数をシングルクオートで囲むとうまくいかなかった。
```
<!-- 以下で設定する情報を取得 -->
heroku config -a rails-docker-practice-satoshi
<!-- 以下を取得できる -->
CLEARDB_DATABASE_URL: mysql://ユーザー名:パスワード@ホスト名/DB名?reconnect=true
```
- 設定した情報は下記で確認できる。
```
heroku config -a rails-docker-practice-satoshi
```

- Dockerfileでは環境ごとの動作を記述できないため、start.shを作成して記述する。（省略）

- これなんだけ
```
heroku config:add RAILS_SERVE_STATIC_FILES='true' -a rails-docker-practice-satoshi
```

- herokuにプッシュする前に、ローカルのコンテナを削除する
```
docker-compose down
<!-- この辺のファイルが残っているとエラーが発生するので、念のため削除 -->
rm src/tmp/pids/server.pid
```

### herokuにプッシュする
- コンテナをherokuにプッシュする（めっちゃ時間かかる）
```
heroku container:push web -a rails-docker-practice-satoshi
```

- herokuにプッシュ出来たら、リリースする。
```
heroku container:release web -a rails-docker-practice-satoshi
```

- テーブルを更新した場合は、以下のコマンドを実行してテーブルを更新する。
```
heroku run bundle exec rake db:migrate RAILS_ENV=production -a rails-docker-practice-satoshi
```

#### 本番環境でエラーが出たときの対応について
- heroku のログを確認するにあたり、デフォルトだとログの情報量が少ないので、以下を設定する。
```
heroku config:add RAILS_LOG_TO_STDOUT='true' -a rails-docker-practice-satoshi
```
- 以下コマンドでログを出力する。
```
heroku logs -t -a rails-docker-practice-satoshi
```

## CI/CD
### Githubに登録
省略
### CI
#### テストコードを記載
省略
#### CircleCIに登録
- Github連携でユーザー登録すること
#### プロジェクトを登録
- よくわからなかったので先に進む
#### configを設定
- .circleci/config.ymlに下記を記載する。
```yml
version: 2.1

orbs:
  # ジョブの記法があらかじめ用意されていて、それを使用するという宣言
  ruby: circleci/ruby@1.1.2

jobs:
  build:
    docker:
      # 使用するイメージを指定
      - image: circleci/ruby:2.7
    # 作業するディレクトリを指定。Githubのレポジトリ名と同じにすること
    working_directory: ~/rails-docker-practice-satoshi/src
    # ステップスでジョブの中身を記述する
    steps:
      # チェックアウトでは、ソースがあるディレクトリの内容を、下記で指定する場所にコピー（チェックアウト）するということを指示する。
      - checkout:
          path: ~/rails-docker-practice-satoshi
      # bundle installなどを実行する。orbsを指定しているため、下記の記法を使える。
      - ruby/install-deps

  test:
    docker:
      - image: circleci/ruby:2.7
      # CircleCIのMySQLのバージョンに合わせるため、5.5を指定
      - image: circleci/mysql:5.5
        # 下記のデータベース用の環境変数の中身は、config/database.ymlの「テスト」の内容と合わせる。
        environment:
          MYSQL_ROOT_PASSWORD: password
          MYSQL_DATABASE: app_test
          MYSQL_USER: root
    # テストジョブ全体の環境変数を指定
    environment:
      BUNDLE_JOBS: "3"
      BUNDLE_RETRY: "3"
      APP_DATABASE_HOST: "127.0.0.1" # 自分のコンピュータという意味
      RAILS_ENV: test
    # 作業するディレクトリ
    working_directory: ~/rails-docker-kyt/src
    
    # 実行する処理を指定
    steps:
      - checkout:
          path: ~/rails-docker-practice-satoshi
      - ruby/install-deps
      - run:
          name: Database setup
          command: bundle exec rails db:migrate
      - run:
          name: test
          command: bundle exec rake test

# ワークフローに記載された順番で実行していく。
workflows:
  version: 2
  build_and_test:
    jobs:
      # まずはビルドする
      - build
      # 次にテストを実行する
      - test:
          requires:
            - build
```

- config/database.ymlを修正する。
  - defaultのホスト名がdbになっている。この設定はDocker Compose用のため、CircleCiには使えない。
  - 環境変数があればそれを使い、なければ`db`を使うように指定する。
```
test:
  <<: *default
  database: app_test
  host: <%= ENV.fetch('APP_DATABASE_HOST') { 'db' } %>
```
#### 環境変数を設定
- CircleCIはGithub上のコードを使用してテストを実行するため、Github上に上げていないmaster keyなどは別途指定する必要がある。
```
heroku config:add RAILS_MASTER_KEY = 'マスターキーの値' -a rails-docker-practice-satoshi
```

#### Githubにプッシュ
- gitを使ってGithubにプッシュする。
```
git add .
git commit -m 'CircleCIの設定を追加'
```

### CD
#### configを設定
- .circleci/config.ymlに下記を記載する。
```yml
version: 2.1

orbs:
  ruby: circleci/ruby@1.1.2
  # 下記を追記
  heroku: circleci/heroku@1.2.3
  
# 省略

  # デプロイについての記述を追加
  deploy:
    docker:
      - image: circleci/ruby:2.7
    steps:
      - checkout
      # setup_remote_dockerを使うことで、Dockerコマンドを使用できるようになる。
      - setup_remote_docker:
          version: 19.03.13
      # herokuコマンドをインストールする（上の方でorbsを書いているので、このような書き方ができる）
      - heroku/install
      - run:
          name: heroku login
          command: heroku container:login
      - run:
          name: push docker image
          command: heroku container:push web -a $HEROKU_APP_NAME
      - run:
          name: release docker image
          command: heroku container:release web -a $HEROKU_APP_NAME
      - run:
          name: database setup
          command: heroku run bundle exec rake db:migrate RAILS_ENV=production -a $HEROKU_APP_NAME

# ワークフローに記載された順番で実行していく。
workflows:
  version: 2
  # ワークフローの名前を変更（これは任意のもので可）
  build_test_and_deploy:
    jobs:
      - build
      - test:
          requires:
            - build
      # デプロイのジョブを追加
      - deploy:
          requires:
            - test
          # 条件を指定できる
          filters:
            # （マージされた）ブランチをMainに限定する。
            branches:
              only: main
```
#### 環境変数の設定
- CircleCI上で環境変数を設定する。
  - CircleCIでプロジェクトを開き、Project SettingsのEnvironment Variablesを開く
  - 以下の環境変数を設定する。
    - HEROKU_APP_NAME
    - HEROKU_API_KEY
  