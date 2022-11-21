# Install Sidekiq
## 実装の方針
- Sidekiqを使ってメールの送信を非同期で実行できるようにする。
- SidekiqはRedisサーバーを通しておこなう。
## RedisとSidekiqのコンテナを作成する。
- docker-compose.ymlにredisとsidekiqのコンテナを記述する。
  - アプリはRedisとSidekiqに依存する。
  - SidekiqはRedisに依存する。
```yml
version: "3.9"
services:
  web:
    depends_on:
      - db
      - chrome
      - redis # 追記
      - sidekiq # 追記
  sidekiq:
    build: .
    environment:
      REDIS_URL: redis://redis:6379
    volumes:
      - .:/walking_lot:cached
      - bundle:/usr/local/bundle
    depends_on:
      - db
      - redis
    # 今回は`config/sidekiq.yml`にオプションを記述するので、-Cオプションを付ける（後述）
    command: bundle exec sidekiq -C config/sidekiq.yml
  redis:
    image: redis:latest
    # redisのデフォルトのポート番号は6379なので、これを使う。
    ports:
      - 6379:6379
    volumes:
      - redis:/data
volumes:
  bundle:
  mysql_data:
  redis: # 追記
```
> [docker-composeでRails+PostgreSQL+Redis+Sidekiq環境を作る](https://snyt45.com/I54Z7umy_)
## Sidekiqを導入
```
gem 'sidekiq'
```
### config/initializer/sidekiq.rbの設定
- ローカルで使用する場合は、`config/initializers/sidekiq.rb.`に`Sidekiq.configure_server`と`Sidekiq.configure_client`を設定する必要がある。urlには、それぞれdocker-compose.ymlで設定した環境変数を設定できるようにする。
```rb
Sidekiq.configure_server do |config|
  config.redis = { url: ENV['REDIS_URL'] }
end

Sidekiq.configure_client do |config|
  config.redis = { url: ENV['REDIS_URL'] }
end

```
> [Using an initializer](https://github.com/mperham/sidekiq/wiki/Using-Redis#using-an-initializer)

## ActiveJobのセットアップ
```rb
module WalkingLot
  class Application < Rails::Application

    # 以下のように、キューイングバックエンドを指定する
    config.active_job.queue_adapter = :sidekiq
  end
end
```
> [4.2 バックエンドを設定する](https://railsguides.jp/active_job_basics.html#%E3%83%90%E3%83%83%E3%82%AF%E3%82%A8%E3%83%B3%E3%83%89%E3%82%92%E8%A8%AD%E5%AE%9A%E3%81%99%E3%82%8B)

- ルーティングを設定する。`require 'sidekiq/web'`が無いとダメらしい。
```rb
if Rails.env.development?
  require 'sidekiq/web'
  mount Sidekiq::Web, at: '/sidekiq'
end
```
#### 高度なオプション
- Sidekiqの高度なオプションを`config/sidekiq.yml`で指定することができる。指定する場合は、コマンド実行時に`-C`オプションをつける。今回はdocker-compose.yml内で実行するコマンドでオプションを付けて実行する。
- 設定するオプションについて
  - concurrencyで同時に実行するプロセスの数を指定できる。デフォルトは10
  - デフォルトでは、sidekiqはdefaultと呼ばれる単一のキューをRedisで実行する。任意で追加することもできる。
    - `active_strage_analysis`は、ActiveStorageでファイルがアタッチされたときの解析ジョブで発生するキュー
    - `active_strage_purge`は、ActiveStorageのファイル差し替えの際に、元のファイルを削除するキュー
```yml
:concurrency: 5
:queues:
  - default
  - active_strage_analysis
  - active_strage_purge
```
> [Advanced Options](https://github.com/mperham/sidekiq/wiki/Advanced-Options)
> [Rails 6の新しいデフォルト設定と安全な移行方法を詳しく解説（翻訳）](https://techracho.bpsinc.jp/hachi8833/2019_11_14/83047)
- [3.16 Active Storageを設定する](https://railsguides.jp/configuring.html#active-storage%E3%82%92%E8%A8%AD%E5%AE%9A%E3%81%99%E3%82%8B)
