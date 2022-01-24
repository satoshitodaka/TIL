# 概要
- Railsにページング機能を簡単に導入できるようにするｇｅｍ

# 導入
- Gemfile に記載し、bundle installを実行
```
gem 'kaminari'
```
# 使い方
- コントローラに記述する
- kaminari の導入により、`page`と`per`というメソッドが使えるようになる。
```
@users = User.order(:name).page params[:page]
```
- ビューに記述する。
```
= paginate @users
```

# 設定
- 1ページあたりの表示件数など、デフォルト値の設定をすることができる。
- ターミナルで実行
```
rails g kaminari:config
```
- そうすると、設定ファイルconfig/initializers/kaminari_config.rbが生成される。<br>必要に応じて設定を変更する。
```
# frozen_string_literal: true

Kaminari.configure do |config|
  config.default_per_page = 15
  # config.max_per_page = nil
  # config.window = 4
  # config.outer_window = 0
  # config.left = 0
  # config.right = 0
  # config.page_method_name = :page
  # config.param_name = :page
  # config.max_pages = nil
  # config.params_on_first_page = false
end

```

# 見た目を変える
- ターミナルで実行
```
rails g kaminari:views　導入するテーマ
```
# 参考
- 現場rails
- https://pikawaka.com/rails/kaminari
- https://github.com/kaminari/kaminari
- https://github.com/amatsuda/kaminari_themes
