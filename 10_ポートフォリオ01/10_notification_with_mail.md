# Notification with Mail
## letter_opener_web
> [letter_opener_web](https://github.com/fgrehm/letter_opener_web)
## Mailer実装の方針
- 今回メール配信の内容は以下の通り
  - アクティビティが新規作成されて、管理者に承認・公開依頼の通知をする
  - アクティビティが承認・公開されたとき、作成者に通知する。
- 前者については、複数名に送信すること、またロール的に分離するためにもAdminMailerとして実装する。
##　メールの設定を定数に設定
- メイラーのインスタンスは、HTTPリクエストのコンテキストと無関係であるため、`_url`ヘルパーを使うためには`:host`パラメータを明示的に伝える必要がある。
```rb
# config/environments/development.rb
config.action_mailer.default_url_options = Settings.default_url_options.to_h
config.action_mailer.delivery_method = :letter_opener_web

```
コンソールで確認する
```
irb(main):001:0> Settings.default_url_options.to_h
=> {:host=>"localhost:3000"}
```
> [2.7 Action MailerのビューでURLを生成する](https://railsguides.jp/action_mailer_basics.html#action-mailer%E3%81%AE%E3%83%93%E3%83%A5%E3%83%BC%E3%81%A7url%E3%82%92%E7%94%9F%E6%88%90%E3%81%99%E3%82%8B)

## AdminMailerをつくる
```
$ docker-compose run web rails g mailer Admin
```
- AdminMailerを作成。管理者向けのメールなどは、デフォルトで送信先を設定することで複数名に送信できる。
```rb
class AdminMailer < ApplicationMailer
  default to: -> { User.admin.pluck(:email) },
          from: 'admin_notification@walking_lot.com'

  def new_activity
    @user = params[:user]
    @activity = params[:activity]
    mail(subject: '作成されたアクティビティを承認してください')
  end
end
```
> [2.3.3 メールを複数の相手に送信する](https://railsguides.jp/action_mailer_basics.html#%E3%83%A1%E3%83%BC%E3%83%AB%E3%82%92%E8%A4%87%E6%95%B0%E3%81%AE%E7%9B%B8%E6%89%8B%E3%81%AB%E9%80%81%E4%BF%A1%E3%81%99%E3%82%8B)
- メールの作成をmypage/activitiesコントローラーに記述
```rb
def create
  @activity = current_user.activities.new(activity_params)

  if @activity.save
    create_notification_about_activity_created(@activity)
    AdminMailer.with(user: @activity.user, activity: @activity).new_activity.deliver_later # ここでメールを作成
    redirect_to mypage_activity_path(@activity), success: 'アクティビティを作成しました。公開まで少々お待ちください'
  else
    flash.now[:danger] = 'アクティビティの作成に失敗しました'
    render :new, status: :unprocessable_entity
  end
end
```

## メールのスペックを書く
- InstaCloneやEveryDayRailsを参考にしながら実装
```rb
RSpec.describe AdminMailer, type: :mailer do
  describe 'アクティビティ新規作成時' do
    let!(:user) { create(:user) }
    let!(:admin_user) { create(:user, admin: true) }
    let!(:activity) { create(:activity, user: user) }
    let!(:mail) { AdminMailer.with(user: user, activity: activity).new_activity }
    it '管理者に承認依頼通知メールが送信される' do
      expect(mail.subject).to eq '作成されたアクティビティを承認してください'
      expect(mail.to).to eq [admin_user.email]
      expect(mail.from).to eq ['admin_notification@walking_lot.com']
      expect(mail.body).to match("#{user.name}さんがアクティビティを新規作成しました。ログインして承認・公開してください。")
    end
  end
end
```
- テスト環境にホスト情報を伝える必要があるので、開発環境と同様に設定する。
```yml
# config/settings/test.yml
default_url_options:
  host: 'localhost:3000'
```
```rb
# config/environments/test.rb
Rails.application.configure do
  config.action_mailer.default_url_options = Settings.default_url_options.to_h
end
```
