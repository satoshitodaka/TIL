# 02_Spaceやその他モデル
## spaceモデル
```
$ rails g model space name:string description:text address:string nearest_station:string latitude:float longitude:float
```
### spacesのコントローラとビューを簡単に実装



### ダミーデータを作成する
```rb
# space
100.times.each do |n|
  # ランダムの緯度経度を作成
  latitude = Random.rand(35.6..35.7)
  longitude = Random.rand(139.70..139.78)

  space = Space.create!(
    name: Faker::Lorem.sentence,
    description: Faker::Lorem.paragraph(
      sentence_count: 30,
      supplemental: false,
      random_sentences_to_add: 4,
    ),
    address: Faker::Address.full_address,
    nearest_station: Faker::Address.city,
    latitude: latitude,
    longitude: longitude,
  )

  # 3.times.each do |n|
  #   画像をランダムで取得するようにする
  # end

  # スペースのfeaturesについての処理
  # スペースのtypesについての処理
end
```

### space_types
- レンタルスペースの種類を管理するモデル。
- レンタルスペースの種類自体を管理するモデル`space_type`とスペースとの中間テーブル`space_type_mapping`を使って実装する。
#### space_typesモデルを作る
```
$ rails g model space_type name:string:uniq
```
#### space_type_mappingsモデルを作る
```
$ rails g model space_type_mapping space:references space_type:references
rails g model space_type_mapping space:references:uniq space_type:references:uniq
```
#### spaceモデルの修正
- models/space.rb に追記する。これにより、space_type_mappings経由でspace_typeを取得できるようになる。
```rb
class Space < ApplicationRecord

  has_many :space_type_mappings, dependent: :destroy # 追記
  has_many :space_types, through: :space_type_mappings # 追記

  validates :name, presence: true
  validates :description, presence: true
end
```
- Space の入力フォームを整える
  - `collection_check_boxes`ヘルパーを使うことでspace_typeのチェックボックスが生成される。
    - ラベルの表記自体は省略しても動作するが、DaisyUIを適用させるために？クラスをきちんと記述する。
```html
<div class="form-control w-full max-w-xs mb-5">
    <%= f.label :space_types, class: 'label' %>
    <div class="flex flex-wrap gap-5">
      <%= f.collection_check_boxes :space_type_ids, SpaceType.all, :id, :name do |b| %>
        <%= b.label class: 'label cursor-pointer justify-start gap-3' do %>
          <%= b.check_box class: 'checkbox checkbox-primary' %>
          <span class="label-text"><%= b.text %></span>
        <% end %>
      <% end %>
    </div>
  </div>
```
> [collection_check_boxesメソッドの構造確認 - Qiita](https://qiita.com/sho012b/items/3a595fde14516081dff5)

- db/seeds.rb でダミーデータを作成するようにする。
```rb
# space_type
# テキストの配列よりSpaceTypeを作成する
%w[レンタルスペース 貸し会議室 セミナー会場 パーティールーム コワーキングスペース カフェ].each do |name|
  SpaceType.create!(name: name)
end

# space
100.times.each do |n|
  
  # 省略

  # スペースタイプを指定
  # 3つのランダムなSpaceTypeのidの配列を、space_type_idsに格納する
  space.space_type_ids = SpaceType.all.sample(3).pluck(:id)
end
```
> [pluck 指定したカラムのレコードの配列を取得](https://railsdoc.com/page/model_pluck)

#### Space作成時にspace_typeを送信できるようにする

### features
#### モデルを作る
```
rails g model feature name:string:uniq
```
#### features_mappingsモデルを作る
```
rails g model feature_mapping space:references feature:references
```
### ActiveStrageを使って画像をアップロードする
- 以下コマンドでセットアップをする。マイグレートすることにより、`active_storage_blobs`、`active_storage_variant_records`、`active_storage_attachments`の3つのテーブルが作られる。
```
$ docker-compose run web rails active_storage:install
$ docker-compose run web rails db:migrate
```
- models/space.rbに複数枚のファイルを紐づけるために追記する
```rb
class Space < ApplicationRecord

  has_many_attached :images # 追記

  # 省略

end
```
- Space作成時に画像を含めることができるようにする。
```rb
# spaces_controller.rb
class SpacesController < ApplicationController
  
  # 省略

  private

  def space_params
    params.require(:space).permit(:name, :description, :address, :nearest_station, { space_type_ids: [] },
                                  { feature_ids: [] }, :longitude, :latitude, { images: [] })
  end
end
```

#### ダミーデータ生成時に画像を付加する。
```rb
100.times.each do |n|
  # 省略

  3.times.each do
    fixture_image_name = "#{Random.rand(1..10)}.jpg"
    space.images.attach(io: File.open(Rails.root.join("db/fixtures/spaces/#{fixture_image_name}")), filename: fixture_image_name, content_type: 'image/jpeg')
  end

  # 省略
end
```
