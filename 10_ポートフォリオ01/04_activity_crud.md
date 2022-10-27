# Activity
## Activityモデルの作成
- UserはActivityを作ることができる。
- User削除後にActivityを消すのはもったいないので、関連付けは`dependent: :nullify`が良いか。
- 公開状況と承認状況のフラグを作成する。
- 優先度は高く無いが、承認制にするなら差し戻しの管理（コメント含む）を実装できるとよい。
```
$ docker-compose run web rails g model Activity content:text released:boolean approved:boolean user:references
```
- Activityのuser_id はnull を許容したいので、`null:false`制約を外して設定
```rb
class CreateActivities < ActiveRecord::Migration[7.0]
  def change
    create_table :activities do |t|
      t.text :content, null: false
      t.boolean :released, default: false, null: false
      t.boolean :approved, default: false, null: false
      t.references :user, foreign_key: true

      t.timestamps
    end
  end
end
```
- ActivityとLocationTypeの中間テーブルを作成する。
```
$ rails g model ActivityLocationType activity:references location_type:references
```
- 中間テーブルにおいて、ActivityとLocationTypeの組み合わせを一意とするため、indexを追加する。
```rb
class CreateActivityLocationTypes < ActiveRecord::Migration[7.0]
  def change
    create_table :activity_location_types do |t|
      t.references :activity, null: false, foreign_key: true
      t.references :location_type, null: false, foreign_key: true

      t.timestamps
    end
    add_index :activity_location_types, [:activity_id, :location_type_id], unique: true, name: 'unique_index_on_activity_location_types'
  end
end
```
- index名が長いと怒られるので、上記の通りnameオプションをつける(公式ドキュメントはわからなかった)
```
StandardError: An error has occurred, all later migrations canceled:

Index name 'index_activity_location_types_on_activity_id_and_location_type_id' on table 'activity_location_types' is too long; the limit is 64 characters
/walking_lot/db/migrate/20221023231329_create_activity_location_types.rb:9:in `change'

Caused by:
ArgumentError: Index name 'index_activity_location_types_on_activity_id_and_location_type_id' on table 'activity_location_types' is too long; the limit is 64 characters
/walking_lot/db/migrate/20221023231329_create_activity_location_types.rb:9:in `change'
Tasks: TOP => db:migrate
(See full trace by running task with --trace)
```
> [RailsのMigrationで index name ... too long と怒られた - Qiita](https://qiita.com/ezawa800/items/9a63a96fb36a7c1de04d)

- ActivityとLotの中間テーブルを作る
```
$ rails g model LotActivity activity:references lot_id:string
```
- 今回LotのidをUUID（string）で作っているので、マイグレーションファイルは以下のようになる。
```rb
class CreateLotActivities < ActiveRecord::Migration[7.0]
  def change
    create_table :lot_activities do |t|
      t.references :activity, null: false, foreign_key: true
      t.string :lot_id, null:false, foreign_key: true

      t.timestamps
    end
    add_index :lot_activities, [:lot_id, :activity_id], unique: true
  end
end
```



[【Rails】ActiveRecordでJOIN先のテーブルのカラムで絞り込む](https://mogulla3.tech/articles/2020-04-11-01)
### その他