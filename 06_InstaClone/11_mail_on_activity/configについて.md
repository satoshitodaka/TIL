### configとは

### そもそも定数管理とは

### 導入
Gemfileに記述
```rb
gem 'config'
```
ターミナルで実行
```
bundle exec rails g config:install
```
- 各種ファイルが生成される。
```
config/settings.yml
config/settings.local.yml
config/settings/development.yml
config/settings/production.yml
config/settings/test.yml
```
- gitignoreは下記の追記
```
config/settings.local.yml
config/settings/*.local.yml
config/environments/*.local.yml

```


### 参考
[Config - Github](https://github.com/rubyconfig/config)

[gem configを理解する ~ configを使った定数管理の方法 - Qiita](https://qiita.com/tanutanu/items/8d3b06d0d42af114a383)
