# 01 rubocopとrspecの導入とCI（GitHub Actions）の設定
## 課題
- 以下を導入する
  - rubocop
  - rubocop-rails
  - rspec-rails
  - factory_bot_rails
  - faker
- rspecとrubocopが正しく動くように設定してください。
- またCI(GitHub Actions)でrubocopとrspecが動くような設定もしてください。
- `/`にアクセスするとテストですという文言が表示されることがテストとして書かれていればOKです。rubocopも全て通しましょう。
- .rubocop.ymlは [こちら](https://github.com/DaichiSaito/insta_clone_ver7/pull/2/files#diff-4f894049af3375c2bd4e608f546f8d4a0eed95464efcdea850993200db9fef5c)をそのまま使ってください。

## 開発メモ
### アプリの作成
- ディレクトリを作り、Railsアプリケーションを作成する。
- rails newのときのオプションは以下の通り
  - --database=mysql
  - --skip-test
  - --skip-bundle
  - --skip-turbolink
```
mkdir i_am_here
rails _7.0.3_ new . i_am_here --database=mysql --skip-test --skip-turbolink --skip-bundle  
```
- Rails7よりフロント周りの実装が大きく変わったので、以下コマンドを実行する。
```
$ rails importmap:install
$ rails turbo:install
$ rails stimulus:install
```

> [importmap.md]()
> [turbo.md]()
> [stimulus.md]()

### Dockerの導入
> []()