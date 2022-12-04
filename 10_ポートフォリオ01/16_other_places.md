# other_places
### モデルを作る
```
$ docker-compose run web rails g model OtherPlace place_number:integer lot_id:string name:string address:string latitu
de:float longitude:float photo_url:string
```
- lot_idとplace_numberは一意である必要があるため、インデックスを貼る
```rb
class CreateOtherPlaces < ActiveRecord::Migration[7.0]
  def change
    create_table :other_places do |t|
      t.integer :place_number, null: false
      t.references :lot, null: false, foreign_key: true

      t.timestamps
    end
    add_index :other_places, [:lot_id, :place_number], unique: true
  end
end
```
- Lotモデル
```rb
class Lot < ApplicationRecord
  include IdGenerator
  belongs_to :user, optional: true
  belongs_to :location_type
  has_one :lot_activity, dependent: :destroy
  has_one :activity, through: :lot_activity
  has_many :other_places, dependent: :destroy
```
- OtherPlaceモデル
```rb
class OtherPlace < ApplicationRecord
  belongs_to :lot

  validates :lot_id, presence: true
  validates :place_number, presence: true, uniqueness: { scope: :lot_id }
end
```
### OtherPlacesの作成について
- LotControllerのなかで、クラスメソッドとして作成する
```rb
class OtherPlace < ApplicationRecord
  private

  def self.create_with_lot(lot, place_order_numbers)
    # 寄り道スポットの数を取得
    other_place_count = place_order_numbers.size - 1
    return if other_place_count == 0
    [*1 .. other_place_count].each do |i|
      other_place_informations = lot.nearby_locations['results'][place_order_numbers[i]]
      other_place = OtherPlace.create(
        lot_id: lot.id,
        place_number: place_order_numbers[i],
        name: other_place_informations['name'],
        address: other_place_informations['vicinity'],
        latitude: other_place_informations['geometry']['location']['lat'],
        longitude: other_place_informations['geometry']['location']['lng']
      )

      if other_place_informations['photos']
        other_place.update(photo_url: other_place_informations['photos'][0]['photo_reference'])
      elsif other_place_informations['photos'].nil?
        other_place.update(photo_url: 'no_image')
      end
    end
  end
end
```
- app/controllers/lots_controller.rb
```rb
def create
    ActiveRecord::Base.transaction do
      if @lot.valid?
        @lot.get_nearby_locations

        if @lot.nearby_locations['status'] == 'OK'
          @place_order_numbers = set_place_order_number(@lot)
          @lot.set_destination(@place_order_numbers)
          @lot.save

          LotActivity.create_with_lot(@lot) # 従属するモデルLotActivityを生成
          OtherPlace.create_with_lot(@lot, @place_order_numbers) # 従属するモデルOtherPlaceを生成

          redirect_to lot_path(@lot), success: 'くじを作成しました'
        elsif @lot.nearby_locations['status'] = 'ZERO_RESULTS'
          flash.now[:info] = '近くにスポットがありませんでした。条件を変えてもう一度引いてください。'
          render :new, status: :unprocessable_entity
        end
      else
        render :new, status: :unprocessable_entity
      end
    end
  end