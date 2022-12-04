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
  # メソッド名にselfをつけると、クラスメソッドになる。
  # classが使えないため、klassを使うらしい
  def self.included(klass)
    # klass = Userがcreateされる前にfill_idを実行する
    klass.before_create :fill_id
  end

  def fill_id
    # 繰り返し処理をし、結果をself.idに代入する
    self.id = loop do
      # uuidを生成してuuidに代入する
      uuid = SecureRandom.uuid
      # 繰り返し終了の条件は、uuidが有効で、idが生成したuuidとなるデータがself.class=Lotに存在しない場合
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
> [instance method Module#included](https://docs.ruby-lang.org/ja/latest/method/Module/i/included.html)
> [module SecureRandom](https://docs.ruby-lang.org/ja/latest/class/SecureRandom.html)
> [klassってなんやねんってお話 - Qiita](https://qiita.com/chatrate/items/54d9226fa98dd1c887ed)

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

### 現在地を取得して出発地にセットするようにする。
- 基本的にはドキュメント通りに書いて実装。
```js
// 現在地を取得してピンを立てる
// 定数locationButtonを定義し、要素buttonを代入する。これにより、地図の中にbuttonという要素が作られる。
const locationButton = document.createElement("button");
// ボタンのテキスト表示を設定
locationButton.textContent = "現在地から歩く";
// ボタンのCSSクラスを定義する
locationButton.classList.add("custom-map-control-button");
// マップ内でのボタンの位置を指定する。
map.controls[google.maps.ControlPosition.TOP_RIGHT].push(locationButton);
// ボタンが押された時の処理を記述する
locationButton.addEventListener("click", (e) => {
  // フォームのデフォルトの動作をキャンセルする
  e.preventDefault();
  // マーカーがあれば削除する
  if (currentMarker != null) {
    currentMarker.setMap(null)
  }
  // ブラウザの位置情報が有効である場合の処理
  if (navigator.geolocation) {
    // Geolocationオブジェクトから現在位置を取得する
    navigator.geolocation.getCurrentPosition(
      // 取得した情報をpositionとし、posに緯度経度の情報を代入する
      (position) => {
        const pos = {
          // coordsプロパティに位置情報が含まれるらしい
          lat: position.coords.latitude,
          lng: position.coords.longitude,
        };
        // mapの中心に取得したposをセットする
        map.setCenter(pos);
        // 取得した現在地でマーカーを作成する。
        currentMarker = new google.maps.Marker({
          position: pos,
          map: map
        });
        // 設定したTargetの値に、イベントで取得した緯度経度情報を代入する。これにより、Form側で紐付けた入力欄に渡される
        this.latitudeTarget.value = pos.lat
        this.longitudeTarget.value = pos.lng
      }
    );
  } else {
    // 位置情報を取得できなければ離脱する
    return false
  }
});
```
> [地図上にユーザーやデバイスの位置を表示する](https://developers.google.com/maps/documentation/javascript/geolocation?hl=ja)
> [Navigator.geolocation](https://developer.mozilla.org/ja/docs/Web/API/Navigator/geolocation)
> [位置の表現](https://developer.mozilla.org/ja/docs/Web/API/Geolocation_API/Using_the_Geolocation_API#%E4%BD%8D%E7%BD%AE%E3%81%AE%E8%A1%A8%E7%8F%BE)

### 出発地を名称から検索できるようにする。可能であれば、リアルタイムでの検索結果を取得する。
- GoogleMaps APIでPlaces Search Boxが用意されているのでこれを使う。
- 取得した検索結果にマーカーを設置し、infoWindowsを表示させて位置情報をセットする。
```js
// SearchFormで出発地を指定してピンを立てる
// 検索ボックスを作成する
const input = document.getElementById("pac-input");
const searchBox = new google.maps.places.SearchBox(input);
map.controls[google.maps.ControlPosition.TOP_LEFT].push(input);

// 検索結果を現在のマップのビューに偏らせる
map.addListener("bounds_changed", () => {
  searchBox.setBounds(map.getBounds());
});

let markers = [];

// 検索結果をユーザーが選択すると、イベントが発生
searchBox.addListener("places_changed", () => {
  const places = searchBox.getPlaces();

  // 検索結果がなければ離脱
  if (places.length == 0) {
    return;
  }

  // 古いマーカーを削除する
  markers.forEach((marker) => {
    marker.setMap(null);
  });
  markers = [];

  const service = new google.maps.places.PlacesService(map);

  // 取得した情報に対し、アイコンと場所の情報を取得する
  const bounds = new google.maps.LatLngBounds();

  places.forEach((place) => {
    if (!place.geometry || !place.geometry.location){
      console.log("Returned place contains no geometry");
      return;
    }

    // 検索結果にマーカーを設定し、markersに含める
    let marker = new google.maps.Marker({
      map,
      title: place.name, //名称
      position: place.geometry.location, //緯度経度
      address: place.formatted_address, // 住所
    })
    markers.push(marker);

    // 表示するウィンドウのコンテンツを設定する
    const contentString =
      `<div id="content">` +
        `<h3>${ marker.title }</h3>` +
        `<p>${ marker.address }</p>` +
        `<input type="button" id="setStartPoint" value="ここから歩く" class="btn btn-primary">` +
      "</div>";

    // infowindowを作成する
    const infowindow = new google.maps.InfoWindow({
      content: contentString,
    });

    let currentInfoWindow = null;

    // マーカーをクリックすると、マーカーの場所でinfoWindowを表示する
    marker.addListener("click", () => {
      // currentInfoWindowが存在すれば閉じる
      if (currentInfoWindow) {
        currentInfoWindow.close();
      }
      // currentInfoWindowとしてinfowindowを開く
      infowindow.open({
        map,
        anchor: marker
      });
      currentInfoWindow = infowindow;

      // infoWindowのDOMが用意できたら発火する
      currentInfoWindow.addListener('domready', () => {
        // infoWindowのボタンがクリックされたら発火する
        document.getElementById("setStartPoint").addEventListener("click", () => {
          this.startpointnameTarget.value = marker.title;
          this.startpointaddressTarget.value = marker.address;
          this.longitudeTarget.value = marker.position.lng();
          this.latitudeTarget.value = marker.position.lat();
        });   
      });
    });

    if (place.geometry.viewport) {
      bounds.union(place.geometry.viewport);
    } else {
      bounds.extend(place.geometry.location);
    }
  });
  map.fitBounds(bounds);
});
```
> [Places Search Box](https://developers.google.com/maps/documentation/javascript/examples/places-searchbox?hl=ja#maps_places_searchbox-javascript)
> [info Windows](https://developers.google.com/maps/documentation/javascript/infowindows?hl=ja)
> [テンプレートリテラル](https://developer.mozilla.org/ja/docs/Learn/JavaScript/First_steps/Strings#%E3%83%86%E3%83%B3%E3%83%97%E3%83%AC%E3%83%BC%E3%83%88%E3%83%AA%E3%83%86%E3%83%A9%E3%83%AB)

##　Lotのshow画面

- JSを使わないのであれば、料金が発生しないEmbed APIというのがあるらしい。
> [Maps Embed API の概要](https://developers.google.com/maps/documentation/embed/get-started)

## スペック
- 途中までスペックを書いたが、GoogleMapsAPIのスペックは面倒らしいので今回はシステムスペックは書かないことにする。以下は参考のためのメモ書き
### 地図をクリックしてくじを作成するスペック
- 通常のシステムスペックに加え、GoogleMapをクリックする動作が必要となる。様々な書き方があるようだが、シンプルにidで見つけてclickするのが良さそう。
```rb
# やり方1
map = find('#map').native
page.driver.browser.action.move_to(map, 200, 190).click.perform
# やり方2
# clickメソッドは、場所を指定しなければ要素の中央をクリックするらしいが
# このスペックでは通らなかったので、場所を指定する
find('#map').click(x: 30, y: 30)
```
```rb
RSpec.describe 'Lots', type: :system do
  let!(:user) { create(:user) }
  let!(:location_type) { create(:location_type) }
  let!(:activity) { create(:activity) }
  let!(:activity_location_type) { create(:activity_location_type, activity: activity, location_type: location_type) }

  describe '地図をクリックしてくじを作成する' do
    context '未ログインユーザー' do
      it 'くじを作成できる' do
        visit '/'
        within '#header' do
          click_on 'くじを引く'
        end
        select 'どこでも', from: 'lot_location_type_id'
        find('#map').click.click(x: 30, y: 30)
        click_on '登録する'
        expect(page).to have_content '散歩くじの結果'
        puts "#{Rails.application.credentials.google_map_api_key}"
      end
    end

    # 省略
  end
end
```
> [[SOLVED]-TESTING GOOGLE MAP MARKERS WITH CAPYBARA AND SELENIUM IN RAILS-GOOGLEMAPS](https://www.appsloveworld.com/googlemaps/100/59/testing-google-map-markers-with-capybara-and-selenium-in-rails)
> [Method: Capybara::Node::Element#click](https://www.rubydoc.info/gems/capybara/Capybara%2FNode%2FElement:click)
> [Ruby on Rails開発のインターン　（Day 16）](https://programming-shop.hatenablog.com/entry/2018/08/07/085919)

### 現在地からくじを作成するスペック
- Capybaraの環境で現在地の情報を利用する手法として、`execute_script`メソッドで任意のスクリプトを実行するやり方がある。今回は現在地を取得するメソッドとそのレスポンスを予め設定することで、Capybaraが`navigator.geoloation.getCurrentPosition`を利用できるようにする。
```rb
describe '現在地を取得してくじを作成する' do
  context '未ログインユーザー' do
    it 'くじを作成できる' do
      visit '/'
      within '#header' do
        click_on 'くじを引く'
      end
      select 'どこでも', from: 'lot_location_type_id'
      page.execute_script "navigator.geolocation.getCurrentPosition = function(success) { success({coords: {latitude: 35.6895014, longitude: 139.6917337}}); }"
      click_on '現在地から歩く'
      sleep(5)
      click_on '登録する'
      expect(page).to have_content '散歩くじの結果'
    end
  end
end
```
> [How to simulate sharing geolocation with Capybara?](https://stackoverflow.com/questions/7367405/how-to-simulate-sharing-geolocation-with-capybara)
> [Capybara::Session#execute_script](https://www.rubydoc.info/gems/capybara/Capybara%2FSession:execute_script)

###　出発地点を検索してくじを作成するスペック
- 未作成

### VCR
- WebAPIキーのスペックは、VCRなどでモック化すると良いらしい。GoogleMapsAPIのスペックは面倒らしいので今回は実装しないが、リクエストスペックにはかなり有効そうなので、リンクだけ残しておく。
> [vcr](https://github.com/vcr/vcr)
> [VCRを使うとRSpecのWebmockの作成が超絶楽になった！](https://morizyun.github.io/blog/webmock-vcr-gem-rails/)
> [外部APIを叩くRubyのコードのテストに「VCR」を使ってみました](https://tech.synapse.jp/entry/2021/11/01/100000)

## 行先の写真を表示する
### 予め用意した写真をランダムで表示する

### 行先の写真をGoogleより取得して表示する

> [場所の写真](https://developers.google.com/maps/documentation/places/web-service/photos)
> [Google Maps APIでGOTOイートしたつもり](https://techracho.bpsinc.jp/en_tak/2020_12_21/101903)

## エラーメッセージのカスタマイズ
- くじを引く際に、出発地が未登録だとエラーが発生する。但し、緯度経度に対してエラーが出るので少しニュアンスが違う
[![Image from Gyazo](https://i.gyazo.com/1b4c00ff34a8026a5f810742f94502c1.png)](https://gyazo.com/1b4c00ff34a8026a5f810742f94502c1)
- 対応の方針は以下の通り
  - 「出発地を入力してください」は一つだけ出したいので、モデルのバリデーションから片方を外す
  - エラーメッセージをカスタマイズして「登録してください」と表示する
```rb
class Lot < ApplicationRecord
  validates :user, presence: true, if: :user_id?
  validates :start_point_latitude, presence: { message: 'を登録してください' }
  # 経度のバリデーションは意図的に外した。緯度経度にバリデーションがかかり、エラーメッセージの重複を避けるため
end
```

> [Railsのバリデーションエラー時、エラーメッセージのカラム名をカスタマイズする](https://blog.parity-box.com/posts/diary/2022/08/05)
> [3.3 :message](https://railsguides.jp/active_record_validations.html#message:~:text=nil).valid%3F%0A%3D%3E%20true-,3.3%20%3Amessage,-%E6%97%A2%E3%81%AB%E4%BE%8B%E7%A4%BA%E3%81%97%E3%81%9F)

## スペック

## エラーなど不具合
### GoogleMapの表示が不安定で、表示に失敗することがある（以下リサーチはしたが、結局やめた）
- マップの表示が不安定で、ページ読み込み時に失敗するときがある。
- Webでざっと調べた限り、JSの記述による読み込みの順番っぽいが、Stimulus環境ではどうしたらよいかわからなかった。
- Stimulusを使ってGoogleMapを表示する場合、マップのコールバックをカスタムイベントに変換し、コントローラとアクションを接続してイベントをリッスンするという手法があるらしい。
#### 修正内容
- マップのURLにコールバック関数`initMap`を定義する
  - `data-turbolinks-eval`を設定することで、処理の重複を避けることができる。（1回しか適用されない）
```html
<script async
  src="https://maps.googleapis.com/maps/api/js?key=<%= Rails.application.credentials.google_map_api_key %>&libraries=places&callback=initMap" data-turbolinks-eval="false">
</script>
```
- `app/javascript/controllers/application.js`にカスタムイベントを記述する
```js
window.initMap = function(...args) {
  // イベントを作成する
  const event = document.createEvent("Events")
  // イベントの名前を"google-maps-callback"と設定する
  event.initEvent("google-maps-callback", true, true)
  event.args = args
  window.dispatchEvent(event)
}
```
> [スプレッド構文](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Operators/Spread_syntax)
> [Document.createEvent()](https://developer.mozilla.org/ja/docs/Web/API/Document/createEvent#notes)
- `app/views/lots/_form.html.erb`で、data属性でアクションも指定する。
```rb
<%= form_with model: lot, data: { controller: 'set-start-point', action: "google-maps-callback@window->set_start_point#initMap" } do |f| %>
<% end %>
```
- マップを表示するだけのページも同様
```html
<!-- GoogleMap -->
<div
  data-controller="show-map"
  data-action="google-maps-callback@window->show_map#initMap"
  data-show-map-destinationlatitude-value="<%= @lot.destination_latitude %>"
  data-show-map-destinationlongitude-value="<%= @lot.destination_longitude %>"
  data-show-map-startpointlatitude-value="<%= @lot.start_point_latitude %>"
  data-show-map-startpointlongitude-value="<%= @lot.start_point_longitude %>"
  class="h-[400px]">
</div>
```


> [Call Stimulus Function from Google Maps callback](https://discuss.hotwired.dev/t/call-stimulus-function-from-google-maps-callback/191)
> [Google Maps API with StimulusJS](https://www.driftingruby.com/episodes/google-maps-api-with-stimulusjs)

## Markerが地図上で隠れる場合がある
- LatLngBoundにロケーションを追加し、fitBoundsメソッドを使えばOK
> [Auto-center map with multiple markers in Google Maps API v3](https://stackoverflow.com/questions/15719951/auto-center-map-with-multiple-markers-in-google-maps-api-v3)