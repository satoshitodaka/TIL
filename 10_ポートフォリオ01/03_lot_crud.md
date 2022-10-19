# Lot_crud
## Lotモデルの作成
- LotはUserに紐付けるが、user_idがnullを許容するようにしたいので、optionalオプションをつける。
```
docker-compose run web rails g model Lot start_point_name:string start_point_address:string start_point_latitude:float start_point_longitude:float destination_name:string destination_address:string destination_latitude:float destination_longitude:float neaby_locations:json user:references
```
### プライマリーキーをUUIDをに変えてみる
- くじを作成するにあたり、Lotのidが整数値とするのを避けたいと考えた。
  - くじ引きである以上、むやみやたらとくじの結果を開示したくない。
  - 自宅周辺をスタート地点とする場合を想定し、可能な限りプライバシーに配慮したい。

- まず最初にマイグレーションファイルを編集する。
  - idの生成をfalseにし、代わりにstring型の`id`を設定する。UUIDは36文字なので文字数の制限を設け、また`primary_key: true`を設定する。
  - 今回は未ログインユーザーにもくじを作って欲しいので、`user:references`を指定するが、`t.references :user`の`null: false`制約は外した。
```rb
class CreateLots < ActiveRecord::Migration[7.0]
  def change
    create_table :lots, id: false do |t|
      t.string :id,                   limit: 36, null: false, primary_key: true
      t.string :start_point_name
      t.string :start_point_address
      t.float :start_point_latitude,  null: false
      t.float :start_point_longitude, null: false
      t.string :destination_name
      t.string :destination_address
      t.float :destination_latitude
      t.float :destination_longitude
      t.json :neaby_locations
      t.references :user, foreign_key: true

      t.timestamps
    end
  end
end
```
- Lot生成時にidを生成するため、`app/models/concerns/id_generator.rb`に記述する。
  - MySQLではidに関数を設定できないため、Rubyで作成して代入するようにする。
  - 今回はLot以外のテーブルでUUIDをを設定する予定はないが、共通化できる機能ということでconcerns下に切り出す。
```rb
module IdGenerator
  def self.included(klass)
    klass.before_create :fill_id
  end

  def fill_id
    self.id = loop do
      uuid = SecureRandom.uuid
      break uuid unless self.class.exists?(id: uuid)
    end
  end
end
```
- `app/models/lot.rb`にインクルードする。
```rb
class Lot < ApplicationRecord
  include IdGenerator
  belongs_to :user
end
```
- Lotのuser_idのnilを許容するため、belongs_toに`optional: true`をつける。また、userの存在性をチェックしたいため、バリデーションをつける。
  - これで存在しないuser_idを設定するとエラーを返すようになるが、エラーはアソシエーション先のUserに対して発生するため、メッセージがちょっとずれてしまう。
```rb
class Lot < ApplicationRecord
  include IdGenerator
  belongs_to :user, optional: true
  validates :user, presence: true, if: :user_id?
end
```
[![Image from Gyazo](https://i.gyazo.com/741fe010b19f93e23ef64c27e8e1506d.png)](https://gyazo.com/741fe010b19f93e23ef64c27e8e1506d)

> [[Rails]外部キーにnilを許可したいけど値を入れる場合は存在チェックしたい場合の実装方法 - Qiita](https://qiita.com/ham0215/items/5872d150b3c468dbc4a9)
> [Ruby on Railsのbelongs_toのバリデーションを外部キーで行うか、アソシエーション名で行うか？ - Qiita](https://qiita.com/ledsun/items/25823f5addc41459b6b8)

## LotのCRUD機能
- 最初から全部実装するのはハードルが高いので、実装の順番は以下の通りとしたい。
  - とりあえず、地図をクリックして出発地点の緯度経度を取得し、くじを作成するようにする。
  - 出発地点の緯度経度をもとにnearby_searchを取得できるようにする。
  - 行先のタイプを予め指定し、neaby_searchのパラメータに含めるようにする。
  - 検索範囲を動的に変更できるようにする。
  - 出発地を名称から選択できるようにする。可能であれば、リアルタイムでの検索結果を取得する。
  - 現在地を取得して出発地にセットするようにする。

### 出発地の緯度経度を取得する。