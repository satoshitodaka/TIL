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
