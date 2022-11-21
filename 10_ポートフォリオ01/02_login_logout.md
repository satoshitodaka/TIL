# login_logout
## 不要なファイルが生成されないように設定
- スペックファイルの生成は個別に設定できるらしいので、今回は追記してみる。
```rb
module WalkingLot
  class Application < Rails::Application
    config.generators do |g|
      g.helper false
      g.test_framework :rspec,
        controller_specs: false,
        view_specs: false,
        helper_specs: false,
        routing_specs: false
    end

  end
end
```
## annotate
- Rails6.1辺りより、`annotate --routes`が正しく動作しないようである。今回は下記のissueを参考にし、`lib/tasks/routes.rake`にタスクを追記することで解消した。
```rake
task routes: :environment do
  puts `bundle exec rails routes`
end
```
> [not working in Rails 6.1 #845](https://github.com/ctran/annotate_models/issues/845)

## GithubActrionsの設定
- yarnを使用するので、ワークフローの中でダウンロードするよう設定。だいそんさんの昔の記事をみかけたので参考にさせてもらった。基本的にはDockerfileと同じようにcurlで取得する。
```yml
steps:
  - uses: actions/checkout@v2 #@v3で通るか確認、またruby-setupも試す
  - name: Setup node and yarn
    run: |
      curl -Ss https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add -; \
      echo "deb https://dl.yarnpkg.com/debian/ stable main" | tee /etc/apt/sources.list.d/yarn.list; \
      curl -sL https://deb.nodesource.com/setup_16.x | bash -; \
      apt-get install -y --no-install-recommends nodejs yarn
  - name: yarn install
    run: yarn install
```
> [Rails + GitHub Actionsではまったこと](https://www.tech-blog.startup-technology.com/2020/85c36da0a4b51cf9ccfd/)

- actionsで用意されたnodejsを利用する方法もあるらしいが、Nodejsのバージョンが15系までと制限がありそうなので、今回は開発環境似合わせる上記の方が良さそう。
```yml
prepare:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: actions/setup-node@v2
        with:
          node-version: $NODE_VERSION
      - name: Install Yarn
        run: npm install -g yarn@$YARN_VERSION
      - name: Cache node_modules
        id: node_modules_cache_id
        uses: actions/cache@v2
        with:
          path: |
            node_modules
            src/*/node_modules
            /home/runner/.cache/Cypress
          key: node-v$NODE_VERSION-v1-${{ hashFiles(format('{0}{1}', github.workspace, '/yarn.lock')) }}
      - name: Install npm modules
        run: yarn install
        if: steps.node_modules_cache_id.outputs.cache-hit != 'true'
```
> [GitHub Actionsで日々の開発運用を楽にする取り組みの紹介](https://developers.freee.co.jp/entry/make-it-easy-with-github-actions-in-nagoya)
> [Node.js のビルドとテスト](https://docs.github.com/ja/actions/automating-builds-and-tests/building-and-testing-nodejs)

- GithubActionsを実行したところ、アセットパイプラインのエラーが発生した。

## ログイン系のシステムスペックで発生する例外処理のエラー
- 404や500エラーの際に専用のページを表示するようにしたところ、ログイン系のスペックで例外処理が発生するらしく、スペックでエラーが出るようになった。
```
1) ログイン ログイン機能 入力内容が正しい場合 ログインができること
     Failure/Error: render template: 'errors/error_404', status: 404, layout: 'application', content_type: 'text/html'
     
     ActionView::MissingTemplate:
       Missing template errors/error_404 with {:locale=>[:ja], :formats=>[:png], :variants=>[], :handlers=>[:raw, :erb, :html, :builder, :ruby, :jbuilder]}.
     
       Searched in:
         * "/walking_lot/app/views"
         * "/usr/local/bundle/gems/rails_admin-3.1.0/app/views"
         * "/usr/local/bundle/gems/kaminari-core-1.2.2/app/views"
         * "/usr/local/bundle/gems/actiontext-7.0.4/app/views"
         * "/usr/local/bundle/gems/actionmailbox-7.0.4/app/views"
     # /usr/local/bundle/gems/meta-tags-2.18.0/lib/meta_tags/controller_helper.rb:22:in `render'
     # ./app/controllers/application_controller.rb:17:in `render_404'
     # /usr/local/bundle/gems/actiontext-7.0.4/lib/action_text/rendering.rb:20:in `with_renderer'
     # /usr/local/bundle/gems/actiontext-7.0.4/lib/action_text/engine.rb:69:in `block (4 levels) in <class:Engine>'
     # /usr/local/bundle/gems/rack-2.2.4/lib/rack/tempfile_reaper.rb:15:in `call'
     # /usr/local/bundle/gems/rack-2.2.4/lib/rack/etag.rb:27:in `call'
     # /usr/local/bundle/gems/rack-2.2.4/lib/rack/conditional_get.rb:27:in `call'
     # /usr/local/bundle/gems/rack-2.2.4/lib/rack/head.rb:12:in `call'
     # /usr/local/bundle/gems/rack-2.2.4/lib/rack/session/abstract/id.rb:266:in `context'
     # /usr/local/bundle/gems/rack-2.2.4/lib/rack/session/abstract/id.rb:260:in `call'
     # /usr/local/bundle/gems/railties-7.0.4/lib/rails/rack/logger.rb:40:in `call_app'
     # /usr/local/bundle/gems/railties-7.0.4/lib/rails/rack/logger.rb:25:in `block in call'
     # /usr/local/bundle/gems/railties-7.0.4/lib/rails/rack/logger.rb:25:in `call'
     # /usr/local/bundle/gems/rack-2.2.4/lib/rack/method_override.rb:24:in `call'
     # /usr/local/bundle/gems/rack-2.2.4/lib/rack/runtime.rb:22:in `call'
     # /usr/local/bundle/gems/rack-2.2.4/lib/rack/sendfile.rb:110:in `call'
     # /usr/local/bundle/gems/railties-7.0.4/lib/rails/engine.rb:530:in `call'
     # /usr/local/bundle/gems/rack-2.2.4/lib/rack/urlmap.rb:74:in `block in call'
     # /usr/local/bundle/gems/rack-2.2.4/lib/rack/urlmap.rb:58:in `each'
     # /usr/local/bundle/gems/rack-2.2.4/lib/rack/urlmap.rb:58:in `call'
     # /usr/local/bundle/gems/rack-2.2.4/lib/rack/builder.rb:244:in `call'
     # /usr/local/bundle/gems/capybara-3.38.0/lib/capybara/server/middleware.rb:60:in `call'
     # /usr/local/bundle/gems/puma-5.6.5/lib/puma/configuration.rb:252:in `call'
     # /usr/local/bundle/gems/puma-5.6.5/lib/puma/request.rb:77:in `block in handle_request'
     # /usr/local/bundle/gems/puma-5.6.5/lib/puma/thread_pool.rb:340:in `with_force_shutdown'
     # /usr/local/bundle/gems/puma-5.6.5/lib/puma/request.rb:76:in `handle_request'
     # /usr/local/bundle/gems/puma-5.6.5/lib/puma/server.rb:443:in `process_client'
     # /usr/local/bundle/gems/puma-5.6.5/lib/puma/thread_pool.rb:147:in `block in spawn_thread'
     # ------------------
     # --- Caused by: ---
     # Capybara::CapybaraError:
     #   Your application server raised an error - It has been raised in your test code because Capybara.raise_server_errors == true
     #   /usr/local/bundle/gems/capybara-3.38.0/lib/capybara/session.rb:163:in `raise_server_error!'
```
- `Capybara.raise_server_errors`がtrueとなっていることが原因らしいので、以下の記事を参考にして特定のスペックで404エラーが発生してもスペックを失敗しないように変更
```rb
context '入力内容が正しい場合' do
  it 'ログインができること' do
    Capybara.raise_server_errors = false
    visit '/login'
    within '#login-form' do
      fill_in 'メールアドレス', with: user.email
      fill_in 'パスワード', with: 'password'
      click_on 'ログイン'
    end
    expect(page).to have_content 'ログインしました'
  end
end
```
> [Railsのシステムテスト（Minitest）で「期待どおりに404エラーが発生したこと」を検証する方法 - Qiita](https://qiita.com/jnchito/items/37fcaf4486c4bdf78802)
