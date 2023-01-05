# admin_panel
## 簡易的にgem 'rails-admin'を導入する
> [【Rails】Gem "rails-admin"を使用して、管理画面を作成する流れ](https://atsushi101011.hatenablog.com/entry/2022/02/09/204015)
> [rails_admin](https://github.com/railsadminteam/rails_admin)
> [rails管理画面系gem比較してみた - Qiita](https://qiita.com/baban/items/f751fb05c4d2367878aa)

## 自分で管理画面をつくる
### Activity
```
$ docker-compose run web rails g controller Admin::Activities index show edit update destroy
```
- 管理者の編集画面では、承認状況と公開状況を編集できるようにする。checkboxは通常、入力の有無を0と1で送信するため、これをboolean値で送信できるようにする。
> [Railsでenum_helpを使ってBoolean型のチェックボックスを作る](https://techracho.bpsinc.jp/kotetsu75/2019_10_02/80570)
- 承認と公開のフラグ以外の編集を可能にするかどうか迷ったが、checkboxのreadonlyオプションが設定できなかったこともあり、ひとまず全項目を編集できるようにする。


#### 気になったこと
- メールのビューテンプレートをhtml/erbでそれぞれ用意すると、エラーが発生する。
```
Failures:

  1) ActivityMailer update activity mail メールが正しく作成されること
     Failure/Error: expect(mail.body).to include user.name
     
       expected #<Mail::Body:0x00007fdb64859de0 @boundary="--==_mimepart_639134eb75fb4_118d855086", @preamble=nil, @e...rt: false, Headers: <Content-Type: text/html>>], @raw_source="", @ascii_only=true, @encoding="7bit"> to include "石井 美羽"
       Diff:
       @@ -1,12 +1,23 @@
       -["石井 美羽"]
       +#<Mail::Body:0x00007fdb64859de0
       + @ascii_only=true,
       + @boundary="--==_mimepart_639134eb75fb4_118d855086",
       + @charset="US-ASCII",
       + @encoding="7bit",
       + @epilogue=nil,
       + @part_sort_order=["text/plain", "text/enriched", "text/html"],
       + @parts=
       +  [#<Mail::Part:31940, Multipart: false, Headers: <Content-Type: text/plain>>,
       +   #<Mail::Part:31960, Multipart: false, Headers: <Content-Type: text/html>>],
       + @preamble=nil,
       + @raw_source="">
       
     # ./spec/mailers/activity_spec.rb:14:in `block (3 levels) in <top (required)>'

Finished in 18 minutes 47 seconds (files took 5.42 seconds to load)
1 example, 1 failure
```
- debuggerで確認すると、mailのbodyが空らしい。正しくMailが作成されると、`@raw_source`にメールのHTMLが入る。
```
(ruby) mail.body
#<Mail::Body:0x00007fdb645fad88
 @ascii_only=true,
 @boundary="--==_mimepart_6391353e4ad1c_118d855249",
 @charset="US-ASCII",
 @encoding="7bit",
 @epilogue=nil,
 @part_sort_order=["text/plain", "text/enriched", "text/html"],
 @parts=
  [#<Mail::Part:36460, Multipart: false, Headers: <Content-Type: text/plain>>,
   #<Mail::Part:36480, Multipart: false, Headers: <Content-Type: text/html>>],
 @preamble=nil,
 @raw_source="">
```
- text形式のメールのビューファイルを削除することで、一旦解消はした。
