# Sidekiq

## 実装
### Sidekiqの導入・設定
- Gemfile に記述してインストールする
```
gem 'sidekiq'
```

- また、前提としてSidekiqの使用にはRedisが必要となる。
  - インスタクローンでは、課題１にて導入ずみなのでスキップ。もし導入する場合はbrewで導入する
```
brew install redis
```

- Initializerに起動時の設定を書く。

config/initializers/sidekiq.rb
```rb
Sidekiq.configure_server do |config|
  config.redis = {
    url: 'redis://localhost:6379'
  }
end

Sidekiq.configure_client do |config|
  config.redis = {
    url: 'redis://localhost:6379'
  }
end
```
- ActiveJobのアダプタにSidekiqを指定する

config/application.rb
```rb
module InstaClone
  class Application < Rails::Application
    # 以下を追記
    config.active_job.queue_adapter = :sidekiq
  end
end
```

### Sidekiqの高度な設定
- Sidekiqの高度な設定を行うことができる。
  - Sidekiqを実行するときにオプションで実行するか、config/sidekiq.ymlに記述する。

インスタクローンの内容通り、キューを指定する場合は　`-q`オプションを付けてターミナルで実行する
```
bundle exec sidekiq -q default -q mailers
```

- config/sidekiq.ymlに設定する場合
  - 設定ファイルに記述する場合、ファイルの場所を明示的に伝える必要があるので、下記を実行する。
```yml
# 並行して実行するSidekiqプロセスの数を指定する。デフォルトでは10スレッド
:concurrency: 25
# キューを設定する。
:queues:
  - default
  - mailers
```
```
sidekiq -C config/initializer/sidekiq.yml
```

### ダッシュボードを作る
Sidekiqの稼働状況をブラウザから確認できるようにする。

Gemfileに記述し、インストールする。
```
gem 'sinatra'
```

config/routes.rbに記述する。
```rb
# 以下を追記
require 'sidekiq/web'

Rails.application.routes.draw do
  if Rails.env.development?
    mount LetterOpenerWeb::Engine, at: "/letter_opener"
    # 以下を追記
    mount Sidekiq::Web, at: '/sidekiq'
  end
end
```

Sisdekiqを起動する
```
bundle exec sidekiq
```

ブラウザから確認できるようになる
![Image from Gyazo](https://i.gyazo.com/8506fe49a0867ba31f0e4d0b74ffc10d.png)

### 参考
#### Sidekiq/wiki
[Getting Started](https://github.com/mperham/sidekiq/wiki/Getting-Started)

[Advanced Options](https://github.com/mperham/sidekiq/wiki/Advanced-Options)

[Active Job](https://github.com/mperham/sidekiq/wiki/Active-Job)

[Monitoring](https://github.com/mperham/sidekiq/wiki/Monitoring)

#### その他記事
[[Ruby on Rails] Sidekiq で非同期処理を実装する](https://dev.classmethod.jp/articles/ruby-on-rails-sidekiq/)

[Railsで非同期処理を行える「Sidekiq」](https://qiita.com/yumiyon/items/6835d90e621e73268021)

[Sidekiqの高度なオプション [翻訳]](https://qiita.com/ts-3156/items/a5f8794a79148c9cfcbe)
