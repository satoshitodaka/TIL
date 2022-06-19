# Rails7の主要な変更点
## bin/dev コマンド
- `rails s`でサーバーを起動しても、JSやCSSがコンパイルされない。
- `bin/dev`コマンドで起動すると、CSSやJSがビルドされ、変更があった場合は自動的に再ビルドしてくれる。
- Rspecのテストを実行するときもJSとCSSを適用させる必要があるので、開発時から`bin/dev`コマンドでサーバーを立ち上げること。

## Turbo
- 以下の症状が出たら要注意
> 保存ボタンをクリックしても画面遷移せず、同じ画面が再描画される
> 保存ボタンをクリックすると画面遷移はしたが、処理の完了を知らせるフラッシュメッセージが出ない
> 削除ボタンをクリックしたら「本当に削除しますか？」の確認ダイアログが出ず、いきなり処理が実行される
> ボタンやリンクをクリックするとなぜか404エラーが発生した

### バリデーション失敗時には`status: unprosessable_entity`をつける必要がある。
- バリデーション失敗時には`status: unprosessable_entity`をつける必要がある。
- これがないと、バリデーション失敗時のエラーメッセージが表示されない。

### link_toメソッドの`method:`オプションの書き方が変わった。
```html
<!-- 従来の書き方（rails-ujsを使っている場合） -->
<%= link_to 'Delete', [task.project, task], method: :delete %>

<!-- Turboを使っている場合 -->
<%= link_to 'Delete', [task.project, task], data: { turbo_method: :delete } %>
```

### destroyのレスポンスに`see_other`をつける必要がある。
- turboを使っている場合、destroyのレスポンスに`status: :see_other`をつける必要がある。
- これを洩らした場合、deleteリクエストでページ遷移したり、予期せぬリソースの削除が発生したりする。
- この問題は、link_toメソッドを使う際に発生し、button_toを使うときは発生しない。
```html
<!-- status: :see_other が必須 -->
<%= link_to 'Delete', [task.project, task], data: { turbo_method: :delete } %>

<!-- status: :see_other がなくても大丈夫 -->
<%= button_to 'Delete', [task.project, task], method: :delete %>
```

### 確認ダイアログの出し方が変わった
- リンクやボタンをクリックしたときの確認ダイアログは以下のように書く。
```html
<!-- インスタクローン（Rails 5.2.6） -->
<%= link_to post_path(@post), class: 'mr-3', method: :delete, data: { confirm: '本当に削除しますか？' } do %> 

<!-- link_toの場合 -->
<%= link_to 'Delete', [task.project, task], data: { turbo_method: :delete, turbo_confirm: 'Are you sure?' } %>

<!-- button_toの場合 -->
<%= button_to "Delete", [task.project, task], method: :delete, form: { data: { turbo_confirm: "Are you sure?" } } %>
```

### 参考
- [Rails 7.0 + Ruby 3.1でゼロからアプリを作ってみたときにハマったところあれこれ - Qiita](https://qiita.com/jnchito/items/5c41a7031404c313da1f)