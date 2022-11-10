# Mypage
## 方針など
- 基本的な実装はインスタクローンを参考にする。
  - mypageのレイアウトは別に用意する。
  - サイドメニューの実装は一旦保留とし、ヘッダーのメニューに含めるようにする。

## プロフィール画像を登録できるようにする
- Active Strageを有効にする
```
$ docker-compose run web rails active_storage:install
$ docker-compose run web rails db:migrate
```
- ActiveStrageのバリデーションは標準で用意されていないので、Gemを導入する
```Gemfile
gem 'activestorage-validator'
```
> [ActiveStorage Validator](https://github.com/aki77/activestorage-validator)
- Userモデルにアバター紐付けとバリデーションの記述をする。
```rb
class User < ApplicationRecord
  # ここはRailsガイド通り
  has_one_attached :avatar
  # Gemの公式通りにコンテンツタイプとサイズを指定する
  validates :avatar, blob: {content_type: ['image/png', 'image/jpg', 'image/ipeg'], size:_range: 1..(5.megabytes) }
end
```
> [Active Storage の概要](https://railsguides.jp/active_storage_overview.html)
