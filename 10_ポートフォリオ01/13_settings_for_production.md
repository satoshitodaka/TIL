# Settings for Production
## 本番環境に必要な設定を確認する
- 本番用にダミーデータを整える
- herokuでアプリを設定
  - DBをアドオンで設定する。
- 本番環境用のストレージを用意する。
- sidekiqにBasic認証をかける
## ダミーデータを整える
```rb
# 環境によって作成するデータを変える
load(Rails.root.join("db", "seeds", "#{Rails.env.downcase}.rb"))
```
- 環境ごとのseed.rbを用意する。
```rb
require './db/seeds/production/location_types'
require './db/seeds/production/activities/anywhere'
require './db/seeds/production/activities/bakery'
require './db/seeds/production/activities/book_store'
require './db/seeds/production/activities/cafe'
require './db/seeds/production/activities/park'
require './db/seeds/production/activities/spa'
require './db/seeds/production/activities/tourist_attraction'
```
- 読み込むときは、環境変数を渡す（未検証）
```
$ docker-compose run web rails db:reset RAILS_ENV=production
```
> [開発環境別にseed ファイルを分けて管理する - Qiita](https://qiita.com/karlley/items/c7183167fc6cc24f5fa4)

## DNSリバインディング攻撃への保護への対応
- `config/environments/development.rb`に追記
```rb
config.hosts << "walking-lot.herokuapp.com"
```
> []()
## Herokuでアプリを作成
- 以前使っていたものからコマンドが変わっていたっぽい
```
$ heroku apps:create walking-lot
```
> [heroku apps:create [APP]](https://devcenter.heroku.com/ja/articles/heroku-cli-commands#heroku-apps-create-app)

## 本番環境用のストレージ(AWS S3)
- AWSの各種設定
> [Rails, Laravel(画像アップロード)向けAWS(IAM:ユーザ, S3:バケット)の設定 - Qiita](https://qiita.com/nobu0717/items/4425c02157bc5e88d7b6)
- `config/environments/production.rb`で本番環境で使用するストレージを指定する
```rb
Rails.application.configure do
  # Store uploaded files on the local file system (see config/storage.yml for options).
  config.active_storage.service = :amazon
end
```
- `config/environments/production.rb`にストレージを記述する
```rb
# Use bin/rails credentials:edit to set the AWS secrets (as aws:access_key_id|secret_access_key)
amazon:
  service: S3
  access_key_id: <%= Rails.application.credentials.dig(:aws, :access_key_id) %>
  secret_access_key: <%= Rails.application.credentials.dig(:aws, :secret_access_key) %>
  region: ap-northeast-1
  bucket: walking-lot-backet
```
> [【初学者必見】 HerokuとAWS S3を連携して投稿画像を保持する方法 - Qiita](https://qiita.com/jibiking/items/0e8c1d826271ac9e4a7d)
## Credentialsの管理にMulti Environment Credentialsを使う
- docker-composeで操作する場合、`EDITOR="vi"`はwebコマンドの前に環境変数として指定することで実行できる。
```
$ docker-compose run -e EDITOR="vi" web bin/rails credentials:edit --environment production
```
- 上記を実行すると、config直下のcredentialsのtreeは以下のようになる。IAMユーザーを作成した際にダウンロードしたcsvを確認し、登録する。
```
.
├── credentials
│   ├── production.key
│   └── production.yml.enc
└── credentials.yml.enc
```
> [Rails 6よりサポートされたMulti Environment Credentialsをプロジェクトに導入する - Zenn](https://zenn.dev/banrih/articles/f22f0a70bbead2a02110)
> [Rails on Dockerでcredentialsをeditしたい - Qiita](https://qiita.com/at-946/items/8630ddd411d1e6a651c6)

- Multi Environment Credentialsを使うと、`config/credentials.yml.enc`は読み込まれないので、従来使っていた秘匿情報を`config/credentials/production.yml.enc`に転記する必要がある。
> [Rails 6.0で環境ごとにcredentialsを準備するとconfig/credentials.yml.encは読み込まれない](https://ryotatake.hatenablog.com/entry/2020/11/28/rails_credentials)
> [【secret_key_base】secret_key_baseの理解をセッションとクッキーと共に深める](https://post-output.com/%E3%80%90secret_key_base%E3%80%91secret_key_base%E3%81%AE%E7%90%86%E8%A7%A3%E3%82%92%E3%82%BB%E3%83%83%E3%82%B7%E3%83%A7%E3%83%B3%E3%81%A8%E3%82%AF%E3%83%83%E3%82%AD%E3%83%BC%E3%81%A8%E5%85%B1/#:~:text=secret_key_base%E3%81%A8%E3%81%AF,-secret_key_base%E3%81%A8%E3%81%AF&text=Rails%E3%82%A2%E3%83%97%E3%83%AA%E3%82%B1%E3%83%BC%E3%82%B7%E3%83%A7%E3%83%B3%E3%82%92%E6%9C%AC%E7%95%AA%E7%92%B0%E5%A2%83,%E3%81%A0%E3%81%A8%E5%88%86%E3%81%8B%E3%82%8A%E3%81%BE%E3%81%97%E3%81%9F%E3%80%82)

- herokuアプリに、環境変数`RAILS_MASTER_KEY`として`config/credentials/production.key`の値を登録する。

## SidekiqにBasic認証をかける
### Sorceryを利用する方法もある。
- `config/routes.rb`に追記する
```rb
# config/routes.rb
Rails.application.routes.draw do
  require 'sidekiq/web'
  require 'admin_constraint'
  mount Sidekiq::Web => '/sidekiq', :constraints => AdminConstraint.new
end
```
- `lib/admin_constraint.rb`を作成して追記する
```rb
class AdminConstraint
  def matches?(request)
    return false unless request.session[:user_id]
    user = User.find request.session[:user_id]
    user && user.admin?
  end
end
```
> [Restful Authentication or Sorcery](https://github.com/mperham/sidekiq/wiki/Monitoring#restful-authentication-or-sorcery)
### Basic認証をつける
- sidekiqのルーティングを以下の通り記述
```rb
if Rails.env.development?
  mount LetterOpenerWeb::Engine, at: '/letter_opener'
end

require 'sidekiq/web'
mount Sidekiq::Web, at: '/sidekiq'
```
- `config/initializers/sidekiq.rb`に追記
```rb
require 'sidekiq/web'
Sidekiq::Web.use(Rack::Auth::Basic) do |user, password|
  [user, password] == [ENV['SIDEKIQ_ADMIN_BASIC_AUTH_USER'], ENV['SIDEKIQ_ADMIN_BASIC_AUTH_PASSWORD']]
end
```
- 開発環境での環境変数をdocker-composeに記述
- 本番環境では、ひとまずアプリの環境変数として登録した。
```yml
environment:
  SELENIUM_DRIVER_URL: http://chrome:4444/wd/hub
  REDIS_URL: redis://redis:6379
  SIDEKIQ_ADMIN_BASIC_AUTH_USER: sidekiq
  SIDEKIQ_ADMIN_BASIC_AUTH_PASSWORD: password
```
> [Sidekiqの管理画面にBasic認証を設定する](https://sunday-morning.app/posts/2020-05-28-sidekiq-admin-basic-auth)

## 本番環境のRedisについて
```
$ heroku addons:create heroku-redis:hobby-dev
```

## 独自ドメイン
> [お名前.comで購入したドメインをHerokuに設定する - Qiita](https://qiita.com/ozin/items/62bc7ef1dd3c827177fb)
> [Custom Domain Names for Apps](https://devcenter.heroku.com/articles/custom-domains)
> [初めてHerokuで独自ドメインを公開するあなたへ - Qiita](https://qiita.com/kenjikatooo/items/07c3d911210a4ca96781)

## 独自ドメインのSSL証明
- 独自ドメインでSSL証明を取得するには、Herokuの有料プラン(Hobby以上)である必要がある。
```
$ heroku certs:auto:enable
```
- アプリ側でも常時SSL化を強制する
```rb
# config/environments/production.rb
Rails.application.configure do
  # Force all access to the app over SSL, use Strict-Transport-Security, and use secure cookies.
  config.force_ssl = true
end
```
> [Automated Certificate Management](https://devcenter.heroku.com/articles/automated-certificate-management#setup)
> [herokuで常時SSL化（rails) - Qiita](https://qiita.com/akkyta/items/9723398de51febfed65a)

## 本番環境でのメール送信
- 本番環境でのメール送信のアドオンとして、SendgridとMailgunがある。今回はドキュメントの日本語対応という点より、Sendgridを使ってみる
> [SendGridとmailgunを比較 - Qiita](https://qiita.com/kusano00/items/9a82dea6e0a77abbb9c4)
> [Ruby on Rails](https://sendgrid.kke.co.jp/docs/Integrate/Frameworks/rubyonrails.html)

## デプロイ時の詰まりメモ
### アセットコンパイル時のエラー
- アセットコンパイル時にエラーが発生した。
  - Rails7でesbuildを使ってherokuにデプロイすると発生するらしい。
```
~/develop/walking_lot on  main [⇡] via  v16.17.1 via 💎 v3.1.2 via  11GiB/16GiB | 11GiB/12GiB on ☁️   
✗  docker-compose run web rails assets:precompile
[+] Running 4/0
 ⠿ Container walking_lot-db-1       Running                                                                                             0.0s
 ⠿ Container walking_lot-chrome-1   Running                                                                                             0.0s
 ⠿ Container walking_lot-redis-1    Running                                                                                             0.0s
 ⠿ Container walking_lot-sidekiq-1  Running                                                                                             0.0s
yarn install v1.22.19
[1/4] Resolving packages...
success Already up-to-date.
Done in 0.41s.
yarn run v1.22.19
$ esbuild app/javascript/*.* --bundle --sourcemap --outdir=app/assets/builds --public-path=assets

  app/assets/builds/application.js      215.8kb
  app/assets/builds/application.js.map  377.3kb

Done in 0.55s.
yarn install v1.22.19
[1/4] Resolving packages...
success Already up-to-date.
Done in 0.23s.
yarn run v1.22.19
$ tailwindcss -i ./app/assets/stylesheets/application.tailwind.css -o ./app/assets/builds/application.css --minify

🌼 daisyUI components 2.31.0  https://github.com/saadeghi/daisyui
  ✔︎ Including:  base, components, themes[29], utilities
  

Done in 3814ms.
Done in 9.70s.
W, [2022-11-20T20:48:29.884091 #1]  WARN -- : Removed sourceMappingURL comment for missing asset 'assets/application.js.map' from /walking_lot/app/assets/builds/application.js
W, [2022-11-20T20:48:30.669818 #1]  WARN -- : Removed sourceMappingURL comment for missing asset 'rails_admin/popper.js.map' from /usr/local/bundle/gems/rails_admin-3.1.0/vendor/assets/javascripts/rails_admin/popper.js
W, [2022-11-20T20:48:30.727136 #1]  WARN -- : Removed sourceMappingURL comment for missing asset 'rails_admin/bootstrap.js.map' from /usr/local/bundle/gems/rails_admin-3.1.0/vendor/assets/javascripts/rails_admin/bootstrap.js
rails aborted!
LoadError: cannot load such file -- sassc

Tasks: TOP => assets:precompile
(See full trace by running task with --trace)
```
- 最終的には、gem 'sassc-rails'は使用したまま、以下の通り`config.assets.css_compressor`をnilに設定することで解消。
  - 余談だが、最初は毎回デプロイして試していたが、ローカルで`rails assets:precompile`すれば済む話だった。
```rb
# config/environments/production.rb
require 'active_support/core_ext/integer/time'

Rails.application.configure do
  # Compress CSS using a preprocessor.
  config.assets.css_compressor = nil
end
```
> [Rails 7 asset pipeline SassC::SyntaxError with Tailwind](https://stackoverflow.com/questions/70401077/rails-7-asset-pipeline-sasscsyntaxerror-with-tailwind/70665740#70665740)

### ActionView::Template::Error (The asset "activities_images/anywhere" is not present in the asset pipeline.
```
Nov 20 16:04:23 walking-lot app/web.1 I, [2022-11-21T00:04:23.254324 #4]  INFO -- : [f28b4d6a-b09f-4c19-b5a8-b13ae14c3249] Started GET "/mypage/activities/1" for 133.106.40.130 at 2022-11-21 00:04:23 +0000
Nov 20 16:04:23 walking-lot app/web.1 I, [2022-11-21T00:04:23.255220 #4]  INFO -- : [f28b4d6a-b09f-4c19-b5a8-b13ae14c3249] Processing by Mypage::ActivitiesController#show as HTML
Nov 20 16:04:23 walking-lot app/web.1 I, [2022-11-21T00:04:23.255285 #4]  INFO -- : [f28b4d6a-b09f-4c19-b5a8-b13ae14c3249]   Parameters: {"id"=>"1"}
Nov 20 16:04:23 walking-lot app/web.1 I, [2022-11-21T00:04:23.262532 #4]  INFO -- : [f28b4d6a-b09f-4c19-b5a8-b13ae14c3249]   Rendered mypage/activities/show.html.erb within mypage/layouts/application (Duration: 2.5ms | Allocations: 574)
Nov 20 16:04:23 walking-lot app/web.1 I, [2022-11-21T00:04:23.262606 #4]  INFO -- : [f28b4d6a-b09f-4c19-b5a8-b13ae14c3249]   Rendered layout mypage/layouts/application.html.erb (Duration: 2.6ms | Allocations: 604)
Nov 20 16:04:23 walking-lot app/web.1 I, [2022-11-21T00:04:23.262787 #4]  INFO -- : [f28b4d6a-b09f-4c19-b5a8-b13ae14c3249] Completed 500 Internal Server Error in 7ms (ActiveRecord: 2.4ms | Allocations: 1323)
Nov 20 16:04:23 walking-lot app/web.1 F, [2022-11-21T00:04:23.263837 #4] FATAL -- : [f28b4d6a-b09f-4c19-b5a8-b13ae14c3249]   
Nov 20 16:04:23 walking-lot app/web.1 [f28b4d6a-b09f-4c19-b5a8-b13ae14c3249] ActionView::Template::Error (The asset "activities_images/anywhere" is not present in the asset pipeline.
Nov 20 16:04:23 walking-lot app/web.1 ):
Nov 20 16:04:23 walking-lot app/web.1 [f28b4d6a-b09f-4c19-b5a8-b13ae14c3249]     1: <% set_meta_tags title: 'アクティビティ詳細' %>
Nov 20 16:04:23 walking-lot app/web.1 [f28b4d6a-b09f-4c19-b5a8-b13ae14c3249]     2: <div class="container mx-auto px-4 py-10 max-w-lg">
Nov 20 16:04:23 walking-lot app/web.1 [f28b4d6a-b09f-4c19-b5a8-b13ae14c3249]     3:   <p class="text-2xl font-bold mb-5">アクティビティ詳細</p>
Nov 20 16:04:23 walking-lot app/web.1 [f28b4d6a-b09f-4c19-b5a8-b13ae14c3249]     4:   <%= image_tag "activities_images/#{@activity.location_types.first.location_type}" %>
Nov 20 16:04:23 walking-lot app/web.1 [f28b4d6a-b09f-4c19-b5a8-b13ae14c3249]     5:   <div class="my-4">
Nov 20 16:04:23 walking-lot app/web.1 [f28b4d6a-b09f-4c19-b5a8-b13ae14c3249]     6:     <p class="text-xl font-bold mb-5"><%= @activity.content %></p>
Nov 20 16:04:23 walking-lot app/web.1 [f28b4d6a-b09f-4c19-b5a8-b13ae14c3249]     7:   </div>
Nov 20 16:04:23 walking-lot app/web.1 [f28b4d6a-b09f-4c19-b5a8-b13ae14c3249] 
```
- `config/environments/production.rb`で`config.assets.compile`をtrueに設定
> [rails6 「ActionView::Template::Error (The asset “application.css” is not present in the asset pipeline」の対処法](https://mebee.info/2021/01/19/post-27981/)

