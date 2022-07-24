# チュートリアル3 Stimulusで管理画面をもっとSPA風にする


## Stimulusの基本的な使い方
- Stimulusのコードは、コントローラー、ターゲット、アクションの3つの要素から成る。
  - コントローラーは、`data-controller`属性の値とJSのファイル名が紐づく。
  - ターゲットは、`data-<コントローラー>-target`属性の値と`static targets = []`で定義された値が紐づく
  - アクションは、`data-action`属性の値とメソッドが紐づく

## インスタントサーチの実装
- 検索フォームに入力すると、自動で検索リクエストを送信してくれるようになる。
- インスタントサーチは、Stimulusコントローラに動作を記述する。
  - このStimulusのコントローラは、`stimulus-rails`というgemが提供するジェネレータ機能を使って作成する。

### インスタントサーチの実装
```
$ rails g stimulus form
```
- 作成されるコントローラの雛形は以下の通り
```js
import { Controller } from "@hotwired/stimulus"

// Connects to data-controller="form"
export default class extends Controller {
  connect() {
  }
}
```
- ジェネレータ起動時、一緒に`index.js`にコードが追加される。
  - 今回は`jsbundling-rails`を使用しているため。`importmap-rails`を使う場合、下記は挿入されない。
```js
  import { application } from "./application"

+ import FormController from "./form_controller.js"
+ application.register("form", FormController)

  import HelloController from "./hello_controller.js"
  application.register("hello", HelloController)
```

- `form_controller.js`を変更する。
  - Turboがリクエストをインターセプトするため、`requestSubmit()`を使う。
  - `submit()`だと、直でフォームの内容をリクエストしてしまい、Turboがリクエストをインターセプトできないため。
```js
import { Controller } from "@hotwired/stimulus"

// Connects to data-controller="form"
export default class extends Controller {
  // コントローラーに紐づく要素（=フォーム）をsubmitするアクション
  submit() {
    this.element.requestSubmit()
  }
}
```
- `index.html.erb`で、`data-controller`と`data-action`を指定する。
  - これにより、`form`コントローラは`<form>`要素のアタッチされる。
  - input時に`form`コントローラの`submit`アクションが実行される。
```html
<div class="card-body">
    <%= search_form_for(
      @search,
      html: {
        data: {
          turbo_frame: "cats-list",
          controller: "form",
          action: "input->form#submit"
        }
      } ) do |f| %>
      <div class="row g3 mb-3">
```

### 検索ボタンの削除
- インスタントサーチを利用する場合、検索ボタンは不要となるため削除する。

### Debonceの実装
- 現状ではキー入力の度にリクエストが発生するため、ユーザー体験やサーバー負荷の観点から望まし状態ではない。
- 複数のキー入力をまとめて一つの入力を見なすため、Debonceを実装する。
- form_controllerに以下を記述する。
```js
import { Controller } from "@hotwired/stimulus"

// Connects to data-controller="form"
export default class extends Controller {
  submit() {
    // セットされているTimeoutをクリアする
    clearTimeout(this.timeout)

    // Timeoutをセットする
    // 200ms後にリクエストを実行する
    // 連続で実行されるとTimeoutはクリアされるため、最後の処理だけしか実行されない
    this.timeout = setTimeout(() => {
      this.element.requestSubmit()
    }, 200)
  }
}
```
## 編集と登録のモーダル化
- 登録や編集のリンクがクリックされると、受け取ったモーダルのHTML片をTurbo Frameで置換する。
- formと同様に、Stimulusコントローラを作成する。
```
$ rails g stimulus modal
```
- Stimulusコントローラを以下のように編集する
```js
import { Controller } from "@hotwired/stimulus"
// BootstrapのModalをimport
import { Modal } from "bootstrap"

export default class extends Controller {
  // `connect()`はStimulusのライフサイクルコールバックの1つ
  // コントローラーがHTML要素にアタッチされた時（=HTML要素が画面に表示された時）に実行される
  connect() {
    // モーダル生成
    this.modal = new Modal(this.element)
    // モーダルを表示する
    this.modal.show()
  }

  // 保存成功時にモーダルを閉じる
  close(event) {
    // event.detail.successは、レスポンスが成功ならtrueを返す
    // バリデーションエラー時はモーダルを閉じたくないので、成功時のみ閉じる
    if (event.detail.success) {
      // モーダルを閉じる
      this.modal.hide()
    }
  }
}
```
- モーダルのパーシャルを用意する
```html
<!-- `"modal"`という<turbo-frame>で囲う -->
<%= turbo_frame_tag "modal" do %>
  <%# modalコントローラーをアタッチする %>
  <div class="modal fade" data-controller="modal">
    <div class="modal-dialog">
      <div class="modal-content">
        <div class="modal-header">
          <h5 class="modal-title"><%= title%></h5>
          <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
        </div>
        <div class="modal-body">
          <%= yield %>
        </div>
      </div>
    </div>
  </div>
<% end %>
```
- 登録・編集フォームをモーダル用に変える

```html
<%= turbo_frame_tag cat do %>
  レイアウトをモーダル用に整える
  <%# turbo:submit-end（Turboのsubmit後）イベント発火時に、modal#close（モーダルを閉じる）を実行する %>
  <%= bootstrap_form_with(model: cat, data: { action: "turbo:submit-end->modal#close" }) do |form| %>
    <%= form.text_field :name %>
    <%= form.text_field :age %>
    <%= form.primary %>
  <% end%>
<% end %>
```

## Turboのキャッシュの無効化
- Turboはページをキャッシュする機能を備えており、主に二つの目的で使用される。
  - ブラウザの戻る・進む機能で利用するもの。
  - プレビューと呼ばれる機能で、すでにキャッシュがあるページを表示する際、ページの表示が完了するまでキャッシュを表示するもの。ユーザーの体感速度が向上する。
- ただし、JS関連で意図しないバグが発生することがあるので、無効にすることもできる。
```html
<meta name="viewport" content="width=device-width,initial-scale=1">

<!-- これを追記する -->
<meta name="turbo-cache-control" content="no-cache">

<%= csrf_meta_tags %>
```

## ページネーションやソート、検索時にURLを更新させる
- TurboFrameのリクエストではURLは変わらない。
- Turbo Driveで遷移した時と同様にURLを変えたい場合、`data-turbo-action`属性を使用する。
```html
<div class="card-body mx-3">
   <%= turbo_frame_tag "cats-list", data: { turbo_action: :advance } do %>
     <div class="row py-2">
       <div class="col-4 mt-auto">
         <%= sort_link(@search, :name) %>
```

## 無限スクロールの実装
### Turbo Framesの遅延読み込み
- 画面読み込みをページロード後に読み込ませる技術
- 遅延読み込みを行うためには、`turbo-frame-tag`で`src`オプションを使う。

```html
<%= turbo_frame_tag "new_cat", src: new_cat_path %>
```
- 生成されるHTMLは下記のとおり
  - ページロード後に`new_cat_path`に対してTurbo Frameリクエストする。
  - レスポンスのち、`turbo-frame`が置換される。
  - `new_cat_path`が重い時などに、初回レンダリング速度を上げることができる。

```html
<turbo-frame id="new_cat" src="/cats/new"></turbo-frame>
```

- `loading`オプションに`lazy`を指定すると、ページロード後ではなく、スクロールで該当する`turbo-frame`が表示された時にTurbo Frameリクエストがされるようになる。
```html
<%= turbo_frame_tag "new_cat", src: new_cat_path, loading: :lazy %>
<!-- 下記のようになる。 -->
<turbo-frame id="new_cat" src="/cats/new" loading="lazy"></turbo-frame>
```
- 以下のように実装する。
```
<div id="cats">
  <%= turbo_frame_tag "cats-page-#{@cats.current_page}" do %>
    <%= render @cats>
    <%= turbo_frame_tag "cats-page-#{@cats.next_page}", loading: :lazy, src: path_to_next_page(@cats) %>
  <% end %>
</div>
```
## FlashにToastを利用する
- Toastとは、軽量な通知のことで、Bootstrapのテーマとして用意されている。
- ToastのStimulusコントローラを作成する。
```
$ rails g stimulus toast
```
- コントローラの内容は以下の通り
```js
import { Controller } from "@hotwired/stimulus"
// BootstrapのToastをimport
import { Toast } from "bootstrap"

// Connects to data-controller="toast"
export default class extends Controller {
  connect() {
    // Toastを生成
    const toast = new Toast(this.element)
    // Toastを表示
    toast.show()
  }
}
```
- Flashのパーシャル
```html
<%# 成功時のToast %>
<% if notice %>
  <div class="mb-2 toast hide border-primary" data-controller="toast">
    <div class="toast-header text-primary">
      <strong class="me-auto"><%= icon_with_text("check-circle", "成功") %></strong>
      <button class="btn-close" data-bs-dismiss="toast">
    </div>
    <div class="toast-body"><%= notice %></div>
  </div>
<% end %>

<%# エラー時のToast %>
<% if alert %>
  <div class="mb-2 toast hide border-danger" data-controller="toast">
    <div class="toast-header text-danger">
      <strong class="me-auto"><%= icon_with_text("exclamation-circle", "エラー") %></strong>
      <button class="btn-close" data-bs-dismiss="toast">
    </div>
    <div class="toast-body"><%= alert %></div>
  </div>
<% end %>
```
- Toastのコンテナは、bodyの一番下、modalと同じ場所に置く。
```html
  <%= turbo_frame_tag "modal" %>
  <div id="flashes" class="position-fixed top-0 end-0" style="margin: 0.75rem"></div>
 </body>
```
## 参考
- [チュートリアル3 Stimulusで管理画面をもっとSPA風にする](https://zenn.dev/shita1112/books/cat-hotwire-turbo/viewer/tutorial-3#%E7%84%A1%E9%99%90%E3%82%B9%E3%82%AF%E3%83%AD%E3%83%BC%E3%83%AB%E3%81%AE%E5%AE%9F%E8%A3%85)
- [throttleとdebounce](https://aloerina01.github.io/blog/2017-08-03-1)
- [Toasts (トースト)](https://getbootstrap.jp/docs/5.0/components/toasts/)