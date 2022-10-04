# GithubActions
## GithubActionsについて
### 概要
- Githubが提供する機能で、CI/CDを実現するもの
- ワークフローファイルはYAML構文を使い、リポジトリの.github/workflow/に保存する。

### ワークフローについて
- 


### config.ymlの内容
```yml
# GithubリポジトリのActionsタブに表示される名前
name: Test
# ワークフローのトリガーを指示する。
on: [push]

# 実行される全てのジョブを表す（直下にグループを明示する）
jobs:
  # rspecというジョブを定義している
  rspec:
    # 使用する仮想マシンを指定する。
    runs-on: ubuntu-latest
    # ジョブ実行時に起動するDockerコンテナを定義する。
    services:
      # servise_idとしてdbを指定
      db:
        # 使用するイメージを指定。docker-compose.ymlに合わせる
        image: mysql:5.7.33
        ports:
          - 3306:3306
        env:
          TZ: "Asia/Tokyo"
          MYSQL_ROOT_PASSWORD: mysql
        # コンテナを作成するときのオプションを指定できる。ここではMySQLの疎通確認をし、ヘルスチェックでDBへの接続が正常かどうかを確かめる。
        options: --health-cmd="mysqladmin ping -pmysql" --health-interval=5s --health-timeout=2s --health-retries=3
    # コンテナのベースとなるイメージを指定
    container:
      image: ruby:3.1.1
    # すべてのステップで使用できる環境変数を指定
    env:
      RAILS_ENV: test
    
    # stepsと呼ばれる一連のタスクを定義する
    steps:
      # usesでは実行するアクションを指定する。
      - uses: actions/checkout@v2
      # nameを使ってGithubに表示するステップの名前を指定できる
      - name: Install chrome
        # OSのシェルを使ってコマンドを実行する。コマンドが複数行の場合は、｜を置いて次の行からコマンドを記述する。
        run: |
          # UbuntuにChromeをインストールする。まず最初に秘密鍵を取得し、登録する。
          wget -q -O - https://dl.google.com/linux/linux_signing_key.pub | apt-key add -
          # Chromeのリポジトリを追加する。
          echo 'deb http://dl.google.com/linux/chrome/deb/ stable main' | tee /etc/apt/sources.list.d/google-chrome.list
          # パッケージリストをアップデートする。
          apt update -y
          # google-chrome-stableとlibvipsをインストールする
          apt install -y google-chrome-stable libvips
      
      - name: bundler config
        # bundlerのインストール先のpathを指定
        run: bundle config set path 'vendor/bundle'
      - name: cache gems
        id: cache-gems
        uses: actions/cache@v2
        # withを指定することでパラメータを渡せる。ここでは、キャッシュする内容を指定する。
        with:
          path: vendor/bundle
          # キャッシュを保存・復元するためのキー
          key: ${{ runner.os }}-gems-${{ hashFiles('**/Gemfile.lock') }}

          restore-keys: |
            ${{ runner.os }}-gems-
      - name: setup bundle
        # キャッシュが存在しない場合、bundle installを実行してgemをインストールする。
        if: steps.cache-gems.outputs.cache-hit != 'true'
        run: |
          bundle install --jobs 4 --retry 3
      - name: setup db schema
        # DBを作成し、schemaファイルを読み込む
        run: |
          bundle exec rails db:create db:schema:load --trace
      - name: run spec
        # rspecを実行する。
        run: bundle exec rspec
      - name: Archive rspec result screenshots
        # rspecが失敗した場合、actions/upload-artifactを使ってスクリーンショットを保存する
        if: failure()
        uses: actions/upload-artifact@v3
        with:
          name: rspec result screenshots
          path: |
            tmp/screenshots/
            tmp/capybara/
  rubocop:
    runs-on: ubuntu-latest
    container:
      image: ruby:3.1.1
    steps:
      - uses: actions/checkout@v2
      - name: bundler config
        run: bundle config set path 'vendor/bundle'
      - name: Cache gems
        id: cache-gems
        uses: actions/cache@v2
        with:
          path: vendor/bundle
          key: ${{ runner.os }}-gems-${{ hashFiles('**/Gemfile.lock') }}
          restore-keys: |
            ${{ runner.os }}-gems-
      - name: setup bundle
        if: steps.cache-gems.outputs.cache-hit != 'true'
        run: bundle install --jobs 4 --retry 3
      - name: run rubocop
        run: bundle exec rubocop
```

### 参考
#### コンテナ関連
- [サービスコンテナについて](https://docs.github.com/ja/actions/using-containerized-services/about-service-containers)
- [jobs.<job_id>.services.<service_id>.options](https://docs.github.com/ja/actions/using-workflows/workflow-syntax-for-github-actions#jobsjob_idservicesservice_idoptions)
- [mysqlの疎通確認(ログイン確認)- Qiita](https://qiita.com/sakito/items/7ddcbfb49edc7a50c6d7)

#### ステップ関連
- [GitHub Actions の基本](https://blog1.mammb.com/entry/2021/12/11/090000)
- [Workflow syntax for GitHub Actions](https://docs.github.com/ja/actions/using-workflows/workflow-syntax-for-github-actions)
- [Checkout V3](https://github.com/actions/checkout)
- [cache](https://github.com/actions/cache)
- [Upload-Artifact v3](https://github.com/actions/upload-artifact)
- [Google Chromeインストール[On Ubuntu 16.04 LTS] - Qiita](https://qiita.com/spiderx_jp/items/e6189a736ddec14ffa23)