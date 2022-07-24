# チュートリアル2 Turboで管理画面をSPA風にする
## TurboDriveを有効化して画面遷移を高速化する。
- turbo deriveを使うと、bodyタグだけを置換するようになるので、画面遷移が高速になる。

## ページネーションのTurboFrames化
- turbo framesはdriveの部分置換版である。
- driveがbody全体を置換するのに対し、framesは`<turbo-frame>`タグで囲まれた範囲を置換する。
- ビューファイルに以下のようにかくと、HTMLが生成される。
```html
<%= turbo_frame_tag "cats-list" do %>
  <div>置換したい箇所</div>
<% end %>

<!-- 以下のHTMLが生成される -->
<turbo-frame id="cats-list">
  <div>置換したい箇所</div>
</turbo-frame>
```
- `turbo-frame`はカスタム要素と呼ばれている。これを使うと独自のHTML要素を定義できる。`<turbo-frame>`を使用すると、内部的にはJSが実行され、置換が行われる。
- リクエストからレスポンスまでは流れは以下の通り。
  - ブラウザのページネーションリンクをクリックした場合、Turboがリクエストをインターセプト（横取り？）してfetchを行う。`turbo-frame`内のリンクなので、リクエストヘッダーに`Turbo-Frame: cats-list`を付与する。これによりリクエストがTurboFrameリクエスストになる。
  - サーバー側では、リクエストヘッダーにturbo-frameが存在すると、TurboFrameリクエストだと判断する。TurboFrameリクエストの場合、メインテンプレートのレンダリングだけすることで高速にレスポンスする。（レイアウトテンプレートのレンダリングはしない）
  - クライアント側ではレスポンスを受け取り、`turbo-frame`のidが一致する`turbo-frame id="cats-list"`だけを置換する。それ以外の要素のついて、レスポンスがあっても捨てられる。

- TurboDriveからTurboFramesに変えると、以下の変化がある。
  - ページネーションが早くなる。
  - ページネーション時のURLが変わらなくなる。
  - ページネーション時に検索フォームの入力内容が失われなくなる。

## 検索のTurboFrames化
- 検索時に画面全体を更新するのではなく、検索結果となる一部分だけを更新する。
- 検索フォームは`turbo-frame`の外側にあるため、このままではTurboFrameリクエストにはならない。そのため、検索フォームに`data-turbo-frame`属性を指定する。
```html
<!-- data-turbo-frame属性を指定する -->
<%= search_form_for @search, html: { data: { turbo_frame: "cats-list" } } do |f| %>

<!-- 生成されたHTMLでは、data-turbo-frame="cats-list"によって、Turbo Frameリクエストになる -->
<form class="cat_search" id="cat_search" data-turbo-frame="cats-list" action="/cats" accept-charset="UTF-8" method="get">
  ...
</form>
```

## 編集のTurboFrames化
### 編集リンクのクリック時の処理
- 編集リンクをクリックした際に、_cat.html.erbの部分を、edit.html.erbの中でrenderしている_form.htm.erbを表示するようにする。
- まずは、_cat.html.erbの中身をturbo-frameで囲む。
  - この時、turbo_frame_tagの引数にcatオブジェクトを渡す。このように書くと、turbo_frame_tagが内部でdom_idを利用してcatを`cat_1`のようなidに変換してくれる。_cat.html.erbはcatの数だけレンダリングされるので、turbo_frameのidが被らないように注意する必要がある。
```html
<!-- 全体を`turbo_frame_tag`で囲う -->
<%= turbo_frame_tag cat do %>
  <div class="row py-2 border-top">
    <div class="col-4 my-auto">
      <%= cat.name %>
    </div>
    <div class="col-4 my-auto">
      <%= cat.age %>
    </div>
    <div class="col-4">
      <div class="d-flex justify-content-end">
        <%= link_to "編集", edit_cat_path(cat), class: "btn btn-sm btn-outline-primary me-2" %>
        <%= button_to "削除", cat, method: :delete, class: "btn btn-sm btn-outline-danger" %>
      </div>
    </div>
  </div>
<% end %>
```
- 次に、_form.html.erbを修正する。
  - 全体を`turbo_frame_tag`で囲む
  - 見た目を一覧に合わせる
```html
<!-- 全体を`turbo_frame_tag`で囲う -->
<!-- 編集リンクをクリックすると、_cat.html.erbの<turbo-frame>部分がこの部分に置換される -->
<%= turbo_frame_tag cat do %>
  <%= bootstrap_form_with(model: cat) do |form| %>
    <div class="row py-2 border-top">
      <div class="col-4">

        <!-- _cat.html.erbの見た目に合うように、フォームの見た目を修正する -->
        <!-- オプションはbootstrap_formのもので、詳細は以下の通り -->
        <!-- skip_label: ラベルは不要 -->
        <!-- label_as_placeholder: ラベルをプレースホルダーとして使う -->
        <!-- wrapper: <div>ラッパーは不要 -->
        control_class: コントロールのclass属性を指定
        <%= form.text_field :name,
                            skip_label: true,
                            label_as_placeholder: true,
                            wrapper: false,
                            control_class: "form-control form-control-sm"
        %>
      </div>
      <div class="col-4">
        <%= form.number_field :age,
                              skip_label: true,
                              label_as_placeholder: true,
                              wrapper: false,
                              control_class: "form-control form-control-sm"
        %>
      </div>

      <div class="col-4">
        <div class="d-flex justify-content-end">
          <%= form.primary class: "btn btn-primary btn-sm me-2" %>
        </div>
      </div>
    </div>
  <% end %>
<% end %>
```
- ポイントとして、ここでもturbo_frame_tagの引数としてcatを渡していること。
  - show.html.erbのturbo-frame id="cat_1"とedit.html.erbのturbo-frame id="cat_1"がマッチする。
  - これにより、編集リンクをクリックすることで、_cat.html.erbのturbo-frame id="cat_1"の部分が、_form.hmtl.erbのturbo-frame id="cat_1"に痴漢される。

### 更新バリデーションエラー時の処理
- リクエストからレスポンスまでの流れ
  - クライアント側では、「更新する」ボタンをクリックすると、TurboFrameリクエストが送信される。
  - サーバー側では、バリデーションエラー時にedit.html.erbのレンダリング結果をレスポンスする。
  - クライアント側では、edit.html.erbでrendersしている_form.html.erbの`turbo-frame id="cat_1"`を置換してバリデーションエラーを表示する。

### 更新成功時の処理
- リクエストからレスポンスまでの流れ
  - クライアント側では、「更新する」ボタンをクリックすると、TurboFrameリクエストが送信される。
  - サーバー側では、`/cats/:id`にリダイレクトする。これにより、cats#showでshow.html.erbのレンダリング結果をレスポンスする。
  - クライアント側では、show.html.erbでrendeしている_cat.html.erbの<turbo-frame>部分を置換して表示する。

### キャンセル時の処理
- リクエストからレスポンスまでの流れ
  - クライアント側で、キャンセルボタンを押すとサーバーにTurboFrameリクエストを送信する。
  - サーバー側では、cats#showでshow.html.erbの結果をレスポンスする。
  - クライアント側では、show.html.erbでrendeしている_cat.html.erbの<turbo-frame>部分を置換して表示する。
```html
<div class="col-4">
  <div class="d-flex justify-content-end">
    <%= form.primary class: "btn btn-primary btn-sm me-2" %>
    <%= link_to "キャンセル", cat, class: "btn btn-sm btn-outline-secondary" %>
  </div>
</div>
</div>
```

## 編集のTurbo Streams化
- 編集に成功したときにFlashメッセージを表示するようにする。
  - ただこれには課題があり、TurboFramesで置換できるのは<turbo-frame>で囲んだ一箇所のみという制約がある。そのため、TurboFramesで編集フォームとFlashメッセージの両方を操作することはできない。
  - そのため、TurboStreamを使うことで、複数箇所を同時に更新できる。
- まず、コントローラのリダイレクト部分を削除し、updateテンプレートをレンダリングするようにする。
```rb
  # PATCH/PUT /cats/1
  def update
    if @cat.update(cat_params)
      # リダイレクトを削除（リダイレクトがないと暗黙的に`render`が実行される）
      # redirect_to @cat, notice: "ねこを更新しました。"
    else
      render :edit, status: :unprocessable_entity
    end
  end
```
- renderするupdateのテンプレートを作成する。ポイントは、フォーマットはhtmlではなくturbo-streamとする。
```html
<!-- ファイル名はapp/views/cats/update.turbo_stream.erb -->
<%= turbo_stream.replace @cat %>
```

### turbo_stream.replaceというメソッドを使う
- turbo-stream.replace部分で使うreplaceは、turbo-railsが用意しているビューヘルパーで、内部的にはパーシャルをrenderする。
```
<!-- renderを省略する場合 -->
<%= turbo_stream.replace @cat %>

<!-- renderを省略しない場合 -->
<!-- partialとlocalsは@catから推測できるため、通常は省略される -->
<%= turbo_stream.replace @cat do %>
  <%= render partial: "cats/cat",
             locals: { cat: @cat } %>
<% end %>
```
- 生成されるHTMLは下記のとおり
  - actionは、操作方法で、replaceはコンテンツを置換する。
  - targetは操作対象となる要素のid属性。
```html
<turbo-stream action="replace" target="cat_1">

  <!-- templateの中にはrenderの結果が置かれる -->
  <template>
    <turbo-frame id="cat_1">
      <div class="row py-2 border-top">
        <div class="col-4 my-auto">
          ジャック
        </div>
        <div class="col-4 my-auto">
          1
        </div>
        <div class="col-4">
          <div class="d-flex justify-content-end">
            <a class="btn btn-sm btn-outline-primary me-2" href="/cats/205/edit">編集</a>
          </div>
        </div>
      </div>
    </turbo-frame>
  </template>

</turbo-stream>
```
- 指定できるactionにはいくつか種類がある。
  - 追加系
    - append targetの末尾に追加する
    - prepend targetの先頭に追加する
    - before targetの前に追加
    - after  targetの後に追加
  - 更新
    - replace targetを更新する。（target要素も含めて更新） 
    - update targetを更新する。（target要素のコンテンツだけ更新）
  - 削除
    - remove targetを削除する

### テンプレートをupdate.turbo_stream.erbとする
- 更新処理の流れは以下の通り
  - クライアント側で「更新する」ボタンをクリックすると、Turboがフォームからのリクエストをインターセプトし、リクエストヘッダーにturbo_streamを追加してfetchする。
  - サーバー側では、ヘッダーの情報により`update.turbo_stream.erb`をrenderする。
  - クライアント側では、レスポンスされた`<turbo-stream action="replace" target="cat_1">...</turbo-stream>`をTurboが処理し、idが一致する要素をreplaceする。
- TurboStream が使えるのはGET***以外**のHTTPリクエストのみ。具体的には、index/show/edit/newアクションではTurboStreamを使用できない。

#### チュートリアルでの使い分け（参考）
##### Turbo Frames
- index
- show
- edit
- new
- update（バリデーションエラー時）
- create（バリデーションエラー時）

##### Turbo Streams
- update（成功時）
- create（成功時）
- destroy（成功時）

## FlashのTurbostream化
- flashをTurboStream のターゲットにするため、idを設定する。
```html
  <div class="col my-4">

    <div id="flash">
      <%= render "flash" %>
    </div>
    <%= yield %>
  </div>
</div>
```
- 

- コントローラでflashを設定する。
  - 通常だとflashはリダイレクト時に使用するが、リクエストに対してflashを使用したいので、`now`を使用する。
```rb
  # PATCH/PUT /cats/1
  def update
    if @cat.update(cat_params)
      flash.now.notice = "ねこを更新しました。"
    else
      render :edit, status: :unprocessable_entity
    end
```
- update.turbo_stream.erbにflashをupdateするコードを追加する。
  - 内部的には、flashのパーシャルをrenderしている。
```
<%= turbo_stream.replace @cat %>
<%= turbo_stream.update "flash", partial: "flash" %>

<!-- 省略せずに書くと以下の通り -->
<%= turbo_stream.update "flash" %>
  <%= render partial: "flash" %>
<% end %>
```

- Flashは他でも使うので、ビューヘルパーとして切り出すと良い。
```rb
# app/helpers/application_helper.rb
def turbo_stream_flash
  turbo_stream.update "flash", partial: "flash"
end
```
- この場合、app/views/cats/update.turbo_stream.erbえrbを以下のように書き換える。
```
  <%= turbo_stream.replace @cat %>
  <%= turbo_stream_flash %>
```

## 登録のTurboStream化
- 登録ボタンをクリックした際に、TurboFrameリクエストで、new.html.erbでrenderされる_form.html.erb を取得し、catsの一番上に表示させる。
- そのために、登録リンクに`data-turbo-frame`を設定し、一覧の一番上に置換するための`turbo-frame`を置く。
```html
        <div class="col-4 d-flex justify-content-end">
          <%= link_to icon_with_text("plus-circle", "登録"),
                      new_cat_path,
                      class: "btn btn-outline-primary",
                      <!-- 以下を追記する -->
                      data: { turbo_frame: "new_cat" }
          %>
        </div>
      </div>


      <!-- 登録リンククリック時に、cats#newの新規登録フォーム部分をここに置換する -->
      <%= turbo_frame_tag "new_cat" %>
      <%= render @cats %>

      <div class="d-flex justify-content-end mt-3">
        <%= paginate @cats %>
```

- 登録が成功したら、登録したCatをCat一覧の先頭に追加する。turbo_streamで操作するため、一覧にidを付与する。
```html
<div id="cats">
  <%= render @cats %>
</div>
```
- コントローラのリダイレクト処理を削除し、flashメッセージは`now`を使って表示する。
```rb
def create
  @cat = Cat.new(cat_params)

  if @cat.save
    flash.now.notice = "ねこを登録しました。"
  else
    render :new, status: :unprocessable_entity
  end
end
```

- create.turbo_stream.erb に処理を記述する。
```
<!-- `#cats`の先頭に登録されたcatを追加する -->
<%= turbo_stream.prepend "cats", @cat %>

<!-- 登録の入力フォームの中身を空にする -->
<%= turbo_stream.update Cat.new, "" %>

<!-- Flashを表示 -->
<%= turbo_stream_flash %>
```

## 削除のTurboStream化
- コントローラでリダイレクト処理を削除し、flashメッセージで`now`を使うように修正する。
  - コントローラーのコードでは明示しないが、暗黙的にdestroy.turbo_stream.erb が実行される。
```rb
# DELETE /cats/1
def destroy
  @cat.destroy
  flash.now.notice = "ねこを削除しました。"
end
```

- destroy.turbo_stream.erbに処理を記述する。
```
<!-- Cat詳細（`#cat_1`となる要素）を削除する -->
<%= turbo_stream.remove @cat %>

<!-- Flashを表示 -->
<%= turbo_stream_flash %>
```

- 削除する際に確認ダイアログを表示するようにする。Turboが標準となったrails7では、新しく用意された`data-turbo-confirm`属性を設定する。
  - button_toが作るformタグに対して`data-turbo-confirm`属性を付与するため、formオプションを使用する。
```html
<div class="col-4">
  <div class="d-flex justify-content-end">
    <%= link_to "編集", edit_cat_path(cat), class: "btn btn-sm btn-outline-primary me-2" %>
    <%= button_to "削除", cat, method: :delete, class: "btn btn-sm btn-outline-danger", form: { data: { turbo_confirm: "本当に削除しますか？" } } %>
  </div>
</div>
```