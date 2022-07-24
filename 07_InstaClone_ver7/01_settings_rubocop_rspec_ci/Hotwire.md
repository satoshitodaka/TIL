# Hotwire
## Hotwireの概要
- Rails7からRailsのデフォルトとなった、モダンなWebアプリケーションを作るためのアプローチ
- TurboとStimulusとStradaの3つから構成される。
- Turboは4つの技術から構成されている。
  - Turbo Drive
  - Turbo Frames
  - Turbo Streams
  - Turbo Native
- TurboはHotwireの中核をなすJavascriptのライブラリ
- StimulusはTurboと相性が良いJavascriptライブラリ。カオスになりがちなJavascriptにレールを敷く役割を持っている。
- StradaはHotwireを使ってモバイルアプリケーションを開発する際に使用するライブラリ。未発表の技術。

## Hotiwireの特徴
- Hotwireの特徴は、サーバーがHTMLレスポンスを返すこと。
- リンクやフォームからのリクエストが全てFetch APIを利用した非同期リクエストになる
  - ReactやVueを利用する際は、サーバーはJSONをレスポンスする。
- Hotwireはfecthに対してHTMLをレスポンスするため、レンダリングはサーバーのみで行えばOK。状態管理はサーバー側で行うことができるし、モデルやバリデーションはサーバー側にのみ書けば良い。従来のgemもそのまま利用できる。
- 既存のRailsアプリケーション開発の延長上で、モダンなWebアプリケーションを作ることができる。

## Turboの特徴
- Javascriptを書かずにSPA風のアプリケーションを実現できるようになる仕組み
- Turboは3つの技術から構成される。
  - Turbo Drive
  - Turbo Frames
  - Turbo Streams

## TurboDriveの特徴
- 画面遷移を高速にしてくれるTurboの機能で、Turbolinksの名前を変えたもの。
- リンクやフォームのリクエストをTurboDriveがインターセプト（横取り）し、fetchによる非同期リクエストに差し替える。
- レスポンスされたHTMLのbodyだけを抜き出して差し替える。
- 画面遷移の際にbodyだけ差し替えるため、JavascriptやCSSがそのまま適用することができるので、画面遷移が高速になる。

## Turbo Framesの特徴
- Turbo FramesはTurbo Driveの部分置換版
- Turbo Driveがbody全体を置換するのに対して、Turbo Framesはturbo-frameタグに囲まれた部分のみ差し替える。
```html
<turbo-frame>hogehoge</turbo-frame>
```

## Turbo Stream
- Turbo Framesが`<turbo-frame>`タグで囲った部分一箇所しか更新できないのに対し、Turbo Streamは複数のHTML要素を同時に処理できる。
- Turbo Framesは差し替え処理をするのに対し、Turbo Streamは複数のHTML要素の追加・更新・削除を処理できる。

## turbo-rails
- turboをrailsで使うためのgem
- turboを直接railsに導入することもできるが、gemを使う方が簡単に記述することができる。

```html
<!-- Turbo Streamsを使い、`#cats`に`_cat.html.erb`のレンダリング結果を追加する -->
<turbo-stream action="append" target="cats">
  <template>
    <%= render 'cats/cat', cat: @cat %>
  </template>
</turbo-stream>

<!-- 上記をturbo-railsで実装すると -->
<%= turbo_stream.append "cats", @cat %>
```

## Stimulusについて
- Stimulsは、Turboと相性が良いJavascriptライブラリで、カオスになりやすいJSにレールを敷いてくれる（わかりやすくしてくれるということ？）
- Turboを使うと、JSを書かずに非同期の処理を実装することができる。そのため、ReactやVueを導入するよりもJSを書く量は減る。
- どうしてもJSを書く必要がある場合、Stimulusのレールに沿ってJSを書く。

## 参考
- [猫でもわかるHotwire入門 Turbo編](https://zenn.dev/shita1112/books/cat-hotwire-turbo/viewer/intro)