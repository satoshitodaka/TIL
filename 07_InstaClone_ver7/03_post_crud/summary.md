# post_crud
## Postモデルを作る
```
docker-compose run web rails g model post body:text address:string latitude:float longitude:float user:references
```
- 生成されたマイグレーションファイルを編集する。(null: falseをつけた)
```rb
class CreatePosts < ActiveRecord::Migration[7.0]
  def change
    create_table :posts do |t|
      t.text :body, null: false
      t.string :address, null: false
      t.float :latitude, null: false
      t.float :longitude, null: false
      t.references :user, null: false, foreign_key: true

      t.timestamps
    end
  end
end
```
## seedファイルを作る
- db/seedsディレクトリから読み込むよう、db/seed.rbに記述する。
```rb
require './db/seeds/users'
require './db/seeds/posts'
```
- db/seeds/users.rb
```rb
puts 'Start inserting seed "users"...'
10.times do
  user = User.create(
    email: Faker::Internet.unique.email,
    username: Faker::Internet.unique.user_name,
    password: 'password',
    password_confirmation: 'password'
  )
  puts "\"#{user.username}\" has created!"
end
```
- db/seeds/posts.rb
```rb
puts 'Start inserting seed "posts"...'
User.limit(10).each do |user|
  post = user.posts.create(
    body: Faker::Hacker.say_something_smart,
    address: Faker::Address.full_address,
    latitude: Faker::Address.latitude,
    longitude: Faker::Address.longitude
  )
  puts "post#{post.id} has created!"
end
```
> [Faker::Address](https://github.com/faker-ruby/faker/blob/master/doc/default/address.md)

## GoogleMap利用のためのGemを導入
- Gemfileに記載し、`bundle install`する。
```
gem 'gmaps4rails'
gem 'geocoder'
```
> [gmaps4rails.md]()
> [geocoder.md]()