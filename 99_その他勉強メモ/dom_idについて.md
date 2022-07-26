# dom_idについて
## 概要
- ActionViewに用意されたヘルパーである。
- 以前からあるヘルパーではあるが、rails7よりHotwireがrailsの標準となったことで再注目されたらしい。
- 

## 使い方
- ビューファイルで以下のように書くと、idの番号を含むidを設定してくれる。
```html
<!-- 以下のように記述すると -->
<div id="<%= dom_id(post) %>"></div>
<!-- 以下のようなHTMLを生成する -->
<div id="post-10"></div>
<!-- 下記のようにベタ打ちしなくて良い -->
<div id="post-10" ></div>
```

## もう少し詳しく
- DOMとは、Document Object Model の頭文字をとったもの
- dom_idを使うことで、アプリケーションのデータとDOM要素を簡単に結びつけることができる。
- dom_id は「レコード」とオプションの「プレフィックス」の2つの引数を受け取る
|引数|説明|
|:-------|:-------:|
|record|`to_key`と`model_name`に対応するオブジェクトならなんでも渡せる。|
|prefix|`to_s`に応答するものであればなんでも渡せる。|
- dom_idを使うことで、railsの規約に自然に沿うことができる。

### Hotwireでの仕様について
- 式展開も可能だが、Railsが用意しているtagビルダーを用意するとスッキリ書けるらしい。いまいちピンとこないので、今度試してみよう。
```html
<%= tag.div id: dom_id(@post, :comments), class: "flex flex-col divide-y" do %>
  <%= render @post.comments %>
<% end %>
```
#### タグビルダーをスッキリ書く
- ERBテンプレートで式展開する書き方もあるが、railsが用意しているタグビルダーを使用するとスッキリ書ける
```
<%= tag.div id: dom_id(@post, :comments), class: "flex flex-col divide-y" do %>
  <%= render @post.comments %>
<% end %>

<!-- タグビルダーを使わない場合はこんな感じ？？動くか分からないので、機会があれば検証して見たい -->
<div id="<%= dom_id(@post, :comments)%>", class: "flex flex-col divide-y">
  <%= render @post.comments %>
</div>
```
#### アンカータグにディープリンクする
- アンカータグを使う場合に表示するidを指定するが、ここでdom_id を使用することができる。
```html
<%= link_to "View comment", posts_path(@post, anchor: dom_id(@comment)) %>
```
- リダイレクト時にも同様にdom_idを指定することができる。
```rb
class CommentsController < ApplicationController
  def create
    @post.comments.create!(comment_params)

    redirect_to posts_path(@post, anchor: dom_id(@comment))
  end
end
```

#### Turbo Framesで使う。
- TurboFramesを使う場合、idはビューの中で一意である必要がある。
- `turbo_frame_tag`内部でも`dom_id`を使用しているが、独自のidを付与することも可能
```
turbo_frame_tag @post # => <turbo-frame id="post_123"></turbo-frame>
turbo_frame_tag dom_id(@post, :comments) # => <turbo-frame id="comments_post_123"></turbo-frame>
```

- TurboFrameを使って、別のリンクをクリックしてフォームを操作する場合
```
<%= turbo_frame_tag @comment, src: comment_path(@comment) %>

<!-- 他の場所 -->
<%= link_to "Edit", edit_comment_path(@comment), data: { turbo_frame: @comment } %>
```
#### TurboStreamsのレスポンス
- TurboStreamのアクションはturbo_stream.erbファイルに記述する必要があるため、ビューファイルと内容を一致させるのが面倒となる場合がある。
- この場合、
```html
<!-- app/views/plans/quick_edit/update.turbo_stream.erb -->
<%= turbo_stream.replace dom_id(@plan, :title), partial: "plans/title" %>
<%= turbo_stream.replace dom_id(@plan, :notes), partial: "plans/notes" %>
<%= turbo_stream.replace dom_id(@plan, :assigned), partial: "plans/assigned" %>

<!-- パーシャルを省略する場合、以下の通り更にスッキリ書ける -->
<!-- app/views/comments/destroy.turbo_stream.erb -->
<!-- 背後で`dom_id(@comment)`が呼ばれる -->
<%= turbo_stream.remove @comment %>
```

## 参考
- [Rails: Action Viewのdom_idヘルパーは実は有能（翻訳）](https://techracho.bpsinc.jp/hachi8833/2022_07_06/119249)
- [Building HTML tags](https://api.rubyonrails.org/classes/ActionView/Helpers/TagHelper.html#method-i-tag)
- [ActionViewのdom_id便利そう](https://blog.piyo.tech/posts/2014-09-01-200000/)
- [DOM の紹介](https://developer.mozilla.org/ja/docs/Web/API/Document_Object_Model/Introduction)