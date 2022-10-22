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
> [Rails の UUIDプライマリキーを試す - Qiita](https://qiita.com/HeRo/items/2816e27fb3066db6c4e6)

- MySQLだとパフォーマンスの問題があるらしいので、後々影響するかもしれない。
> [MySQLでプライマリキーをUUIDにする前に知っておいて欲しいこと](https://techblog.raccoon.ne.jp/archives/1627262796.html)

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
  - 取得したJsonから行先を設定できるようにする。
  - 行先のタイプを予め指定し、neaby_searchのパラメータに含めるようにする。
  - 出発地を名称から検索できるようにする。可能であれば、リアルタイムでの検索結果を取得する。
  - 現在地を取得して出発地にセットするようにする。

### 出発地の緯度経度を取得する。
- ひとまず地図をクリックして出発地点を設定する。[Rental Space](https://github.com/satoshitodaka/TIL/blob/main/09_%E3%81%B2%E3%81%9F%E3%81%99%E3%82%89Rails%E3%82%A2%E3%83%97%E3%83%AA%E3%82%92%E4%BD%9C%E3%82%8B/rental_space/03_%E3%81%9D%E3%81%AE%E4%BB%96%E6%A9%9F%E8%83%BD%E3%81%AE%E5%AE%9F%E8%A3%85.md)の課題を参考にさせてもらった。

### 出発地点の緯度経度をもとにnearby_searchを取得できるようにする
- nearby_seachのメソッドは、モデルに記述し、コールバックで使用するようにする。
- nearby_seachでJsonデータを取得するメソッドを作成する。
```rb
  def get_neaby_locations
    # 動的な要素を先にローカル変数に代入する。
    google_map_api_key = Rails.application.credentials.google_map_api_key
    start_point = "#{self.start_point_latitude}" + ',' + "#{self.start_point_longitude}"
    # 取得に使用するURLを用意する。言語は日本語とし、距離は半径3km以内
    url = "https://maps.googleapis.com/maps/api/place/nearbysearch/json?location=#{ start_point }&radius=3000&language=ja&key=#{ google_map_api_key }"
    # URIを生成する
    uri = URI.parse(url)
    # URIを使ってリクエストを送り、結果をresponseに代入
    response = Net::HTTP.get(uri)
    # responseを解析し、Json形式に整えてresultに代入
    result = JSON.parse(response)
  end
```
- Lotのnearby_searchにセットするメソッドを作成する。
```rb
def set_neaby_locations
  # 上記のメソッドの戻り値を代入する
  self.neaby_locations = get_neaby_locations
end
```
- 作成したメソッドはコールバックで呼び出す。複数呼び出す場合は上から順に実行される。
```rb
before_create :get_neaby_locations
before_create :set_neaby_locations
```
> [【Rails】入力された値を元にGETリクエストを送ってデータを取得し、DBに保存する - Qiita](https://qiita.com/aiandrox/items/ba82db7c12d413cb4cef)
> [RubyでJSON形式の結果が返ってくるURLをParseする - Qiita](https://qiita.com/awakia/items/bd8c1385115df27c15fa)
> [Net::HTTP#get](https://docs.ruby-lang.org/ja/latest/method/Net=3a=3aHTTP/i/get.html)
> [JSON.parse()](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/JSON/parse)
> [Railsのコールバックまとめ](https://www.techscore.com/blog/2012/12/25/rails%E3%81%AE%E3%82%B3%E3%83%BC%E3%83%AB%E3%83%90%E3%83%83%E3%82%AF%E3%81%BE%E3%81%A8%E3%82%81/)

### 取得したJsonから行先を設定できるようにする
- Jsonの結果は20件取得できるが、0番目と19番目は広域の情報らしいので、これを除いてランダムに取得する。
- これをJson.perseでハッシュに変換し、行先を代入するようにする。
- 先程と同様にコールバックで使用する。
```rb
before_create :set_destination

def set_destination
  # ハッシュのN番目を取得するため、1から18の範囲からランダムな数値を取得する
  order_number = Random.rand(1 .. 18)
  # nearby_locationsより、N番目の値を取り出して代入する
  destination_infomations = self.neaby_locations["results"][order_number]

  # Lotのそれぞれの値に代入する
  self.destination_name = destination_infomations["name"]
  self.destination_address = destination_infomations["vicinity"]
  self.destination_latitude = destination_infomations["geometry"]["location"]["lat"]
  self.destination_longitude = destination_infomations["geometry"]["location"]["lng"]
end
```
> [【Ruby】 JSON形式のデータをRubyで扱う方法とは？](https://pikawaka.com/ruby/json#JSON%E5%BD%A2%E5%BC%8F%E3%81%AE%E6%96%87%E5%AD%97%E5%88%97%E3%81%A8Ruby%E3%81%AEHash%E3%81%B8%E3%81%AE%E5%A4%89%E6%8F%9B%E6%96%B9%E6%B3%95)

### 行先のタイプを予め指定し、neaby_searchのパラメータに含めるようにする。
- Location_typeを作成する
```
$ rails g model Location_type type:integer name:string
```
- Location_typeのlocation_typeはenumで値を設定する。
  - `store`を設定したかったが、ActiveRecordのメソッドと重複するため不可らしい。後日、対応できるか詳しく調べるが、ちょっと無理そう。
```rb
class LocationType < ApplicationRecord
  validates :location_type, presence: true, uniqueness: true
  validates :name, presence: true

  enum location_type: { anywhere: 0,
                        cafe: 1,
                        park: 2,
                        tourist_attraction: 3,
                        spa: 4,
                        bakery: 5,
                        book_store: 6,
                        hindu_temple: 7
  }
end
```
- LocationTypeのアソシエーションを追記する。Lotとの関係については、Lotからlocation_typeの値を参照するのみなのでbelongs_toのみ記述する。
```rb
class Lot < ApplicationRecord
  include IdGenerator
  belongs_to :user, optional: true
  belongs_to :location_type
```
> [belongs_to関連付け](https://railsguides.jp/association_basics.html#belongs-to%E9%96%A2%E9%80%A3%E4%BB%98%E3%81%91)
- LocationTypeモデルを作成後、DB側に制約を設けたほうが良いのでは、と思い至ったので追記する。
  - location_typeは重複しては困るので、uniqueのindexを貼る。
```rb
class AddIndexLocationTypeOnLocationTypes < ActiveRecord::Migration[7.0]
  def change
    add_index :location_types, :location_type, unique: true
  end
end
```
  - デフォルト値の要否は迷いどころで、APIリクエスト送信時にtypeが無効な値orブランクだと検索結果に影響しないようなので、デフォルト値を指定しても意味がなさそうではあるが、デフォルト値をanywhereとすると分かりやすい気もするので、今回はあえて設定した。[公式Doc](https://developers.google.com/maps/documentation/places/web-service/search-nearby?hl=ja#type)に厳密にそうは書いてないが、無効な値は無視される旨の記述がある。
```rb
class ChangeDefaultLocationTypeOnLocationTypes < ActiveRecord::Migration[7.0]
  def change
    change_column_default :location_types, :location_type, from: nil, to: 0
  end
end
```
> [Changing Columns](https://guides.rubyonrails.org/active_record_migrations.html#changing-columns)
- LocationTypeのダミーデータを追加する。
```rb
type = [0, 1, 2, 3, 4, 5, 6, 7] # ここはダサい気がするので修正すること！
type_name = %w[どこでも カフェ 公園 観光スポット 温泉 パン屋 本屋 神社・寺]

puts 'Start inserting seed "location_type"...'

type.zip(type_name) do | type, name|
  location_type = LocationType.create!(
    location_type: type,
    name: name
  )
  puts "\"#{location_type.name}\" has created!"
end
```
- Lotにlocation_type_idを保存するカラムを追加する
```rb
class AddLocatiouTypeIdToLots < ActiveRecord::Migration[7.0]
  def change
    add_column :lots, :location_type_id, :bigint, null: false
  end
end
```
- ビューにセレクトボックスを追記する。
```rb
<%= form_with model: lot, data: { controller: 'set-start-point' } do |f| %>
  <%= f.label :location_type %>
  <%= f.collection_select :location_type_id, LocationType.all, :id, :name, class: 'select select-bordered w-full max-w-xss' %>
```
- コントローラのストロングパラメータに追記
```rb
private
  def lot_params
    params.require(:lot).permit(
      :location_type_id,
      :start_point_name,
      :start_point_address,
      :start_point_latitude,
      :start_point_longitude,
    )
  end
```
- URLにLocationTypeを代入できるようにする。
```rb
def get_neaby_locations
  google_map_api_key = Rails.application.credentials.google_map_api_key
  start_point = "#{self.start_point_latitude}" + ',' + "#{self.start_point_longitude}"
  url = "https://maps.googleapis.com/maps/api/place/nearbysearch/json?location=#{ start_point }&radius=3000&type=#{ self.location_type.location_type }&language=ja&key=#{ google_map_api_key }"
  uri = URI.parse(url)
  response = Net::HTTP.get(uri)
  JSON.parse(response)
end
```
### 出発地を名称から検索できるようにする。可能であれば、リアルタイムでの検索結果を取得する。
- 難しそうなので後日

### 現在地を取得して出発地にセットするようにする。
- 難しそうなので後日