# 概要
# 導入
#### インストール
- Gemfileに記載し、bundle installを実行する。
```
gem 'carrierwave', '~> 2.0'
```
#### アップローダーの作成
- ターミナルで実行する。
```
rails generate uploader アップローダー名
```
- 実行すると、app/uploaders/xxxx_uploader.rbというアップローダーが作成される。
- 作成されたアップローダーでは、ファイルの保存先や拡張子など、ファイルアップロードに関する設定ができる。
- デフォルトの設定では、アップロードしたファイルは `public/uploads/モデル名/カラム名/id`に保存される。

#### モデルにファイル用のカラムを作成する。
- マイグレーションファイルを作成し、モデルにカラムを追加する。
- カラムの型はStringでOK。
```
rails g migration add_images_to_posts images:string
```


#### モデルとアップローダーの紐付け
- モデルにアップローダーとカラムの紐付けを記述する。<br>複数ファイルをアップロード可能にする場合、`mount_uploaders`とすること。<br>一つのファイルの場合は`mount_uploader`を使う。
```
class Post < ApplicationRecord
  mount_uploaders :images, ImageUploader
  serialize :images, JSON # If you use SQLite, add this line.
end
```
- 型の扱いについては理解が曖昧なので改めて追記します。

#### ビューとコントローラの修正

- ビューのフォームにアップロード用のメソッドを追加する。複数ファイルのアップロードを可能にする場合は、`multiple: true`を追記する。
```
form.file_field :images, multiple: true
```
- コントローラの　paramsメソッドに画像用のカラムを追記する。<br>複数ファイルをアップロードする場合、空のhashを指定する。
```
def post_params
 params.require(:post).permit(:body, images:[])
end
```
# 参考
- https://pikawaka.com/rails/carrierwave
- https://github.com/carrierwaveuploader/carrierwave
