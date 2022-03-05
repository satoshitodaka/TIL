### configとは

### そもそも定数管理とは
- 定数の管理を楽にし、メンテナンス性をあげるもの。
  - 環境ごとに関数を変えたり、１箇所にまとめて設定することができたりする。
- Railsアプリケーションにおいては、定数管理の手法がある。
  - application_controllerに記述する。
  - config/initializers/constants.rbにて宣言する
  -  gemを使う(config/settingsなど)  

### 導入
Gemfileに記述
```rb
gem 'config'
```
ターミナルで実行
```
bundle exec rails g config:install
```
各種ファイルが生成される。
```
# configの設定ファイル
config/initializers/config.rb

# すべての環境で利用する定数を定義
config/settings.yml
# ローカル環境で利用する定数を定義
config/settings.local.yml
# 開発環境で利用する定数を定義
config/settings/development.yml
# 本番環境で利用する定数を定義
config/settings/production.yml
# テスト環境で利用する定数を定義
config/settings/test.yml
```
gitignoreは下記が追記される
```
config/settings.local.yml
config/settings/*.local.yml
config/environments/*.local.yml
```
config/settings/development.ymlに定数を設定する。
```yml
default_url_options:
  host: 'localhost:3000'
```
今回は、config/environments/development.rbにて定数を呼び出す。
```rb
Rails.application.configure do
  # 以下を追記
  config.action_mailer.default_url_options = Settings.default_url_options.to_h
  config.action_mailer.delivery_method = :letter_opener_web
end
```

### 参考
[Config - Github](https://github.com/rubyconfig/config)

[gem configを理解する ~ configを使った定数管理の方法 - Qiita](https://qiita.com/tanutanu/items/8d3b06d0d42af114a383)
