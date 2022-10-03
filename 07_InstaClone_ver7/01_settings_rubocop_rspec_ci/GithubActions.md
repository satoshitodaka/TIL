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
      db:
        image: mysql:5.7.33
        ports:
          - 3306:3306
        env:
          TZ: "Asia/Tokyo"
          MYSQL_ROOT_PASSWORD: mysql
        # コンテナを作成するときのオプションを指定できる。ここではMySQLの疎通確認をし、ヘルスチェックでDBへの接続が正常かどうかを確かめる。
        options: --health-cmd="mysqladmin ping -pmysql" --health-interval=5s --health-timeout=2s --health-retries=3
    container:
      image: ruby:3.1.1
    env:
      RAILS_ENV: test
    
    # ジョブで実行される全てのステップをグループ化する
    steps:
      - uses: actions/checkout@v2
      - name: Install dependent libraries
        run: |
          apt-get update
          apt-get install sudo
          sudo apt-get install -y libpq-dev
          curl -sS -L https://dl.google.com/linux/linux_signing_key.pub | apt-key add -
          echo "deb [arch=amd64] http://dl.google.com/linux/chrome/deb/ stable main" > /etc/apt/sources.list.d/google.list
          apt-get install -y google-chrome-stable
      - name: bundler config
        run: bundle config set path 'vendor/bundle'
      - name: cache gems
        id: cache-gems
        uses: actions/cache@v2
        with:
          path: vendor/bundle
          key: ${{ runner.os }}-gems-${{ hashFiles('**/Gemfile.lock') }}
          restore-keys: |
            ${{ runner.os }}-gems-
      - name: setup bundle
        if: steps.cache-gems.outputs.cache-hit != 'true'
        run: |
          bundle install --jobs 4 --retry 3
      - name: setup db schema
        run: |
          bundle exec rails db:create ridgepole:apply --trace
      - name: run spec
        run: bundle exec rspec
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