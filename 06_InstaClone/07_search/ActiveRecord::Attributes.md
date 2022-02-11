# （前提として）ActiveRecordのAttribute　API
- テーブルに存在しない属性をモデルに設定することができる。
- 必要であれば、既に存在する属性の型をオーバーライドすることもできる。
```rb
create_table "books", force: :cascade do |t|
  t.string   "name"
  t.integer  "price"
  t.date     "published_on"
  t.datetime "created_at",                     null: false
  t.datetime "updated_at",                     null: false
  t.string   "out_of_print", default: "false"
end
```
Bookクラスでは、当然　priceはinteger型となるが、何らかの事情により、priceをfloat型で扱う必要が出てきた場合に、attributes API　を使用して型を設定できる。
```rb
class Book < ApplicationRecord
 attribute :price, :float
end
```
実行結果はこんな感じ
```
Book.first.price = 1200
p Book.first.price.class
#=> float
```

とても便利な機能だが、***ActiveRecordでのみ使うことができた。***

# ActiveRecord::Attributes
- Rails5.2以降で使用できるようになった。
- ActiveModelにて、任意の属性の追加や型のオーバーライドを可能にする機能。
  - 従来、ActiveModelで属性の型を定義する際に、Attributes APIが使用できず不便であった。
  - ActiveModelで属性の型を定義する際はコードが冗長になっていた（らしい）

## 使い方・実装について
- インスタクローンにおいては、検索フォームをFormObjectとして切り出す。
- 検索フォームはDBにアクセスしないので、、、
  -　`include ActiveModel::Model`と記述することにより、ActiveModelの機能を利用する。
  -　　'include ActiveModel::Attributes'と記述することで、ActiveModel：：Attributesを利用する。
```rb
class SearchPostsForm
  include ActiveModel::Model
  include ActiveModel::Attributes

  attribute :body, :string
  attribute :comment_body, :string
  attribute :username, :string
end
```

# 参考
[Rails 5のActive Record attributes APIについて](https://y-yagi.tumblr.com/post/140725723370/rails-5%E3%81%AEactive-record-attributes-api%E3%81%AB%E3%81%A4%E3%81%84%E3%81%A6)

[ActiveModel::Attributes が最高すぎるんだよな](https://qiita.com/alpaca_taichou/items/bebace92f06af3f32898)
