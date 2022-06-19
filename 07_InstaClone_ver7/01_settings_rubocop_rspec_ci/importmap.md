# importmap
## 概要
- Javascriptモジュールを直接ブラウザにインポートする仕組み。
- トランスパイルやバンドルを必要とせずに、ESモジュール向けJavascriptライブラリを用いてモダンなJavascriptアプリケーションを構築できる。
- 使用する際はWebpack/yarn/npmなどが不要となり、アセットパイプラインのみで構築できる。

### メリット
- JavaScriptのバンドルが不要となる。

## インストール
- Rails7以降の新規アプリケーションでは自動的に同梱される。
- 既存のアプリケーションに入れる場合は以下を操作する。
```
./bin/bundle add importmap-rails
./bin/rails importmap:install
```

- `rails importmap:install`を実行すると、以下のファイルが生成される。
```rb

```

## 使い方
### config/importmap.rbの書き方
### npmパッケージを使う


## 参考
- [Rails 7: importmap-rails gem README（翻訳）- TechRacho](https://techracho.bpsinc.jp/hachi8833/2021_10_07/112183)
- [Rails 7.0 で標準になった importmap-rails とは何なのか？ - Zenn](https://zenn.dev/takeyuweb/articles/996adfac0d58fb)
