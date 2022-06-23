# importmap
## 概要
- Javascriptモジュールを直接ブラウザにインポートする仕組み。
- トランスパイルやバンドルを必要とせずに、ESモジュール向けJavascriptライブラリを用いてモダンなJavascriptアプリケーションを構築できる。
- 使用する際はWebpack/yarn/npmなどが不要となり、アセットパイプラインのみで構築できる。

### メリット
- JavaScriptのバンドルが不要となる。
- HTTP/2が使えるため、一つの接続で複数のファイルを処理できるため、読み込み時間が少なくて済む。
- バンドルが不要となるため、変更があったファイルのみ取得する運用となる。そのため、新たにダウンロードするファイルは最小限で済む。

## インストール
- Rails7以降の新規アプリケーションでは自動的に同梱される。
- 既存のアプリケーションに入れる場合は以下を操作する。
```
./bin/bundle add importmap-rails
./bin/rails importmap:install
```

## 使い方
- importは、config/importmap.rbの設定を介して、Rails.application.importでセットアップされる。
- config.importmap.rbは、更新されると自動的に設定を再読み込みさせる。
  - ピン留めの解除など、サーバーの再起動が必要な場合もある。
- `<%= javascript_importmap_tags %>`タグがheadタグ内でインライン化される。

### config/importmap.rbの書き方
#### pin(name, to: nil, preload: false)
- 第一引数はパッケージ名。規約上、app/javascript内のファイル名と同一にすること。
- `to:`オプションは、パッケージ名とファイル名が一致しない時に指定する。リモートサーバーのモジュールを読み込む際は、ここでURLを指定する。
- `preload:`オプションは、プリロードしたい時に指定する（後述）

#### pin_all_from(dir, under: nil, to: nil, preload: false)
- 指定したディレクトリ以下のファイル一式をマッピングする。
- 第一引数はディレクトリを指定する
- `under:`オプションはサブディレクトリを指定する。これの指定がない場合、パッケージ名は第一引数のディレクトリからみた相対パスとなる。
- `to:`オプションは、カスタマイズされたアセットパスを指定する。これの指定がない場合、アセットパスは第一引数で指定したディレクトリからみた相対パスとなる。
- `preload`オプションはプリロードしたい時に指定する。

### npmパッケージを使う
- bin/importmapコマンドを使うことで、npmパッケージの依存関係を解決しつつ、config/importmap.rbにパッケージを追加することができる。
- ターミナルで以下を実行
```
$ bin/importmap pin react react-dom
Pinning "react" to https://ga.jspm.io/npm:react@17.0.2/index.js
Pinning "react-dom" to https://ga.jspm.io/npm:react-dom@17.0.2/index.js
Pinning "object-assign" to https://ga.jspm.io/npm:object-assign@4.1.1/index.js
Pinning "scheduler" to https://ga.jspm.io/npm:scheduler@0.20.2/index.js
```
- config/importmap.rbに追加される
```
# config/importmap.rb
pin "react", to: "https://ga.jspm.io/npm:react@17.0.2/index.js"
pin "react-dom", to: "https://ga.jspm.io/npm:react-dom@17.0.2/index.js"
pin "object-assign", to: "https://ga.jspm.io/npm:object-assign@4.1.1/index.js"
pin "scheduler", to: "https://ga.jspm.io/npm:scheduler@0.20.2/index.js"
```
- パッケージを覗くときは、bin/importmap unpinを使う。
```
$ bin/importmap unpin react react-dom
Unpinning "react"
Unpinning "react-dom"
Unpinning "object-assign"
Unpinning "scheduler"
```

- ウォーターフォール効果（最も深くネストされたインポートに達するまで、ブラウザが次々とファイルを読み込む羽目になる現象）を回避するため、Railsではリンクの`modulepreload`をサポートする。
- 指定するには、config/importmap.rbのピン留めの際にオプションをつける。
- 生成されるビューには、importの下にタグが生成されていることがわかる
[![Image from Gyazo](https://i.gyazo.com/b87e3bc0124652778a23f0a620051b11.png)](https://gyazo.com/b87e3bc0124652778a23f0a620051b11)

### その他
#### パッケージをダウンロードする
- 本番環境でCDNを使いたくない場合、以下のように設定することができる。(`--download`オプションをつける)
```
$ ./bin/importmap pin react --download
Pinning "react" to vendor/react.js via download from https://ga.jspm.io/npm:react@17.0.2/index.js
Pinning "object-assign" to vendor/object-assign.js via download from https://ga.jspm.io/npm:object-assign@4.1.1/index.js
```
- config/importmap.rbは以下のようになり、パッケージはvendor/javascriptにダウンロードされる。
```rb
pin "react" # https://ga.jspm.io/npm:react@17.0.2/index.js
pin "object-assign" # https://ga.jspm.io/npm:object-assign@4.1.1/index.js
```
- ダウンロードしたパッケージを削除する場合は、`--download`オプションをつける。
```
$ ./bin/importmap unpin react --download
Unpinning and removing "react"
Unpinning and removing "object-assign"
```
#### 特定のモジュールをインポートする
- app/javascript以下のJavascriptモジュールを、特定のページでインポートできる。
```rb
pin "checkout"
# 上記のように記述すると、app/javascript/checkout.jsを読み込む
```
- 特定のページにタグ`javascript_import_module_tag`を貼って読み込むようにする。(おそらく、content_forヘルパーのブロック内に置く？)
  - この際に、`javascript_import_tags`の後に書くようにする。
```html
- content_for :head do
  = javascript_import_module_tag "checkout"
```
```html
= javascript_importmap_tags
= yield(:head)
```


## 参考
- [Rails 7: importmap-rails gem README（翻訳）- TechRacho](https://techracho.bpsinc.jp/hachi8833/2021_10_07/112183)
- [Rails 7.0 で標準になった importmap-rails とは何なのか？ - Zenn](https://zenn.dev/takeyuweb/articles/996adfac0d58fb)
