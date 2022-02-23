# ポリモーフィックについて
### ポリモーフィックとは
- ある一つのモデルが他の複数もモデルに属していることを、一つの関連づけだけで表現できる。

### 実装
#### ポリモーフィック関連付をするモデルを作成する
- 今回はポリモーフィックをマイグレーションファイルに記述したが、`rails g`のコマンドに入れると以下のようになる。
  - `#{関連名}:references{polymorphic}` と書くとマイグレーションファイルに記述される。
```
rails g model Activity subject:references{polymorphic} user:references action_type:integer read:boolean
```
#### 従属する（利用される）モデルに記述する。
```rb
class Activity < ApplicationRecord
  belongs_to :subject, polymorphic: true
end
```

#### 従属される（利用する）モデルに記述する。
- 教科書通りでは、以下の通り記述する。
```rb
class Comment < ApplicationRecord
  has_many :activities, as: :sucject
end

class Like < ApplicationRecord
  has_many :activities, as: :sucject
end
```
上記の通り設定すると、activityのコレクションを、 commentやlikeのインスタンスから取得できる。
```
@comment.activities
@like.activities
```
- Activityのインスタンスがあれば、`Actibity.subject`と記述することで、親のインスタンスを取得できる。
- そのためには、ポリモーフィックなインターフェースを使うモデル（ここでは従属する側のモデル）にて、外部キーのカラムと型のカラムを両方とも設定する必要がある。
```
class CreateActivities < ActiveRecord::Migration[5.2]
  def change
    create_table :activities do |t|
      t.bigint :subject_id
      t.string :subject_type
      t.timestamps
    end
    
    add_index :pictures, [:subject_id, :subject_type]
  end
end
```
t.referencesという書式を使うともっとシンプルに記述することができる。（インスタクローンの課題はこの書き方）
```
class CreateActivities < ActiveRecord::Migration[5.2]
  def change
    create_table :activities do |t|
      t.references :subject, polymorphic: true

      t.timestamps
    end
  end
end
```
referencesで記述した場合でも、shema.rbを確認すると確かにtypeとidが設定されていることがわかる。
```
  create_table "activities", options: "ENGINE=InnoDB DEFAULT CHARSET=utf8mb3", force: :cascade do |t|
    # ↓この部分！！
    t.string "subject_type"
    t.bigint "subject_id"
    
    t.bigint "user_id"
    t.integer "action_type", null: false
    t.boolean "read", default: false, null: false
    t.datetime "created_at", null: false
    t.datetime "updated_at", null: false
    t.index ["subject_type", "subject_id"], name: "index_activities_on_subject_type_and_subject_id"
    t.index ["user_id"], name: "index_activities_on_user_id"
  end
```

### ポリモーフィックの必要性
#### こういうったときに使うと良い
- 一つのモデルから複数モデルを参照するとき。
  - 中間テーブルを作って、通常通り関連付けすることもできるが、都度カラムを追加する必要があるし、関連するモデルを取得する際に条件分岐が必要となる。
  - 複数クラスのインターフェースを統一し、同じように扱えるようにする（これをダックタイピングという）と、ポリモーフィック関連付けを使うと良い。
  - ポリモーフィック関連付を使うと、関連付されたモデルのどれを使うのか？を都度意識せずに使える（語弊があるかも）。

### 参考
[2.9 ポリモーフィック関連付け - Railsガイド](https://railsguides.jp/association_basics.html#%E3%83%9D%E3%83%AA%E3%83%A2%E3%83%BC%E3%83%95%E3%82%A3%E3%83%83%E3%82%AF%E9%96%A2%E9%80%A3%E4%BB%98%E3%81%91)

[Railsのポリモーフィック関連とはなんなのか](https://qiita.com/itkrt2y/items/32ad1512fce1bf90c20b)

[複数のテーブルに対して多対一で紐づくテーブルの設計アプローチ](https://spice-factory.co.jp/development/has-and-belongs-to-many-table/)

[ポリモーフィック関連覚え書き](https://qiita.com/kasei-san/items/a9795de8a6558fbcff14)
