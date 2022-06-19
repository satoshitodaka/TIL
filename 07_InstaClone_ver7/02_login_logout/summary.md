# 02_login_logout
### ログイン機能の実装
- ユーザー作成に失敗した時に、404を返すことができる？？
```rb
class UsersController < ApplicationController
  def create
    @user = User.new(user_params)
    if @user.save
      auto_login(@user)
      redirect_to '/', success: 'ユーザー登録が完了しました'
    else
      # これ
      render :new, status: :unprocessable_entity
    end
  end
end
```
> [422 Unprocessable Entity](https://developer.mozilla.org/ja/docs/Web/HTTP/Status/422)
> [Railsアプリケーションの422エラーページの文言をどうするといいのか調べてみた - Qiita](https://qiita.com/hrtkmztn/items/d6a1b504c4d1a5288d73)

### 

> [【Rails】Rspecでマクロを定義して処理を共通化する方法 - Qiita](https://qiita.com/mattan5271/items/6df9f14e5daa5e0dc1c8)