# Notification
- 複数名への通知方法を考えあぐねたので、インスタクローンRails7verを参考にさせてもらう。
- NotificationとUserの間に中間テーブルUserNotificationモデルを作成することで、複数名に通知を作成できる。
##　モデルをつくる
```
$ docker-compose run web rails g model notification title:string url:string
$ docker-compose run web rails g model user_notification user:references notification:references read:boolean
```
- UserNotificationモデルにおいて、ユーザーと通知は一意なので、インデックスを貼る。
```rb
class CreateUserNotifications < ActiveRecord::Migration[7.0]
  def change
    create_table :user_notifications do |t|
      t.references :user, null: false, foreign_key: true
      t.references :notification, null: false, foreign_key: true
      t.boolean :read, default: false, null: false

      t.timestamps
    end
    add_index :user_notifications, [:notification_id, :user_id], unique: true
  end
end
```
### Notificationモデルにモデルメソッド`notify`を作成する
```rb
# 引数にアスタリスクを1つつけると配列を受け取らせることができる。
def notify(*user)
  # flattenメソッドで受け取った引数を配列にする
  users = user.flatten
  # usersのidをpluckメソッドで取得し、user_notificationsというハッシュを作成する。
  user_notifications = users.pluck(:id).map do |user_id|
    # Ruby3.1より、以下のケースではuser_idの値を省略できるようになった
    { notification_id: id, user_id: }
  end
  # 作成したuser_notificationsをUserNotificationsに一括登録する。
  UserNotification.insert_all(user_notifications, record_timestamps: true)
end
```
- 引数の頭にアスタリスクをつけることで、引数に配列を許容する。ちなみに、アスタリスクを1つつけると配列、2つつけるとハッシュ、アスタリスクのみ渡すと、過剰な引数が無視される。
> [メソッドの引数にアスタリスク - Qiita](https://qiita.com/super-mana-chan/items/ca728d90db7c53295b15)
> [メソッド定義](https://docs.ruby-lang.org/ja/latest/doc/spec=2fdef.html#method:~:text=%E3%82%92%E8%BF%94%E3%81%97%E3%81%BE%E3%81%99%E3%80%82-,%E3%83%A1%E3%82%BD%E3%83%83%E3%83%89%E5%AE%9A%E7%BE%A9,-%E4%BE%8B)
> [instance method Array#flatten](https://docs.ruby-lang.org/ja/latest/method/Array/i/flatten.html)
> [Ruby 3.1: ES6風のハッシュリテラル省略記法が追加（翻訳）](https://techracho.bpsinc.jp/hachi8833/2021_11_24/113383)
- UserNotificationのモデルに記述
```rb
class UserNotification < ApplicationRecord
  belongs_to :notification
  belongs_to :user

  validates :read, inclusion: { in: [true, false] }
  validates :notification_id, uniqueness: { scope: :user_id }

  # これは実際に使ってみてから追記する
  delegate :title, to: :notification, prefix: true
  delegate :url, to: :notification, prefix: true
end
```
> [3.4.1 delegate](https://railsguides.jp/active_support_core_extensions.html#delegate)

- `#notify`についてモデルスペックを記述
 - `create_list`を使うことで、ファクトリをまとめて作成することができる
```rb
RSpec.describe Notification, type: :model do
  describe '#notify' do
    let!(:notification) { create(:notification) }

    context '通知先が1名の場合' do
      let!(:user) { create(:user) }
      it '通知が作成されること' do
        expect { notification.notify(user)}.to change { UserNotification.count }.by(1)
      end
    end

    context '通知先が複数名の場合' do
      let!(:users) { create_list(:user, 10) }
      it '通知が作成されること' do
        expect { notification.notify(users)}.to change { UserNotification.count }.by(10)
      end

    end
  end
end
```
> [RSpecでFactoryBotから複数のインスタンスをまとめて作成する【create_listを使用】 - Qiita](https://qiita.com/kodai_0122/items/e755a128f1dade3f53c6)

- 通知作成のアクションを追加し、`mypage/activities#create`の中で呼び出す
```rb
def create_notification_about_activity_created(activity)
  notification = Notification.create!(
    title: '新規作成されたアクティビティを承認してください。',
    url: rails_admin_path
  )
  notification.notify(User.admin)
end
```
### 既読管理
- 既読機能を考えるにあたって、ルーティングを厳密にリソースフルにするため、要件を以下のようにする
  - URLは`notifications/:notification_id/read`とする
  - 既読にするメソッドを`notifications#read`とせず、`notifications/reads#create`とする。
```
$ docker-compose run web rails g controller notifications/read create
```
- ルーティングを記述する。
  - URLを`notifications/:notification_id/read`とするために、`resources :notifications, only: [] do`のブロックの中に`reads#create`を定義する。
  - ディレクトリ構造を`notifications/reads`とするために、moduleオプションを付ける。
```rb
resources :notifications, only: [] do
  resource :read, only: %i[create], module: :notifications
end
```
- コントローラー
```rb
class Notifications::ReadsController < ApplicationController
  before_action :require_login

  def create
    # 確認する通知をパラメータから取得する
    notification = current_user.notifications.find(params[:notification_id])
    debug.logger("通知は#{notification.title}")
    # 既読にするUserNotificationを、通知のidから取得する
    user_notification = current_user.user_notifications.find_by!(notification_id: params[:notification_id])
    # user_notificationが未読であれば、既読に更新する
    user_notification.update!(read: true) unless user_notification.read?
    # 通知のURLに遷移する
    redirect_to notification.url
  end
end
```
### ビューの実装
- headerでNotificationをrenderする際に、未読通知に絞って渡すようにする。
```rb
<%= render 'shared/notifications', user_notifications: current_user.user_notifications.where(read: false).includes(:notification).limit(3) %>
```
- 未読の通知があれば、バッジを表示する。
```html
<label tabindex="0" class="btn btn-ghost btn-circle">
  <div class="indicator">
    <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 17h5l-1.405-1.405A2.032 2.032 0 0118 14.158V11a6.002 6.002 0 00-4-5.659V5a2 2 0 10-4 0v.341C7.67 6.165 6 8.388 6 11v3.159c0 .538-.214 1.055-.595 1.436L4 17h5m6 0v1a3 3 0 11-6 0v-1m6 0H9" /></svg>
    <% if notifications.size > 0 %>
      <span class="badge badge-xs badge-primary indicator-item"></span>
    <% end %>
  </div>
</label>
```
- 通知の一覧では、未読であればnewのバッジを表示する。
```html
<% @user_notifications.each do |user_notification| %>
  <%= link_to notification_read_path(user_notification.notification), data: { turbo_method: :post } do %>
    <div class="indicator w-full">
      <!-- indicator for new notification -->
      <% unless user_notification.read? %>
        <span class="indicator-item badge badge-primary">new</span> 
      <% end %>
      <!-- notification card -->
      <div class="card w-full bg-base-100 shadow-xl mb-5 mx-auto">
        <div class="card-body">  
          <p class="card-title"><%= user_notification.notification.title %></p>
          <div class="text-right">
            <p><%= time_ago_in_words(user_notification.created_at) %>前</p>
          </div>
        </div>
      </div>
    </div>
  <% end %>
<% end %>
```