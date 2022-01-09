# Flash
## 概要
- セッション的な要素を持ち、リクエスト毎にクリアされる。そのため、直後のリクエストでのみ参照できる。
- flashにアクセスするには`flash` メソッドを使用する。

### 一般的な使い方
```
class LoginsController < ApplicationController
  def destroy
    session.delete(:current_user_id)
    flash[:notice] = "ログアウトしました"
    redirect_to root_url
  end
end
```

### リダイレクトのオプションとして使う方法
```
redirect_to root_url, notice: "ログアウトしました"
redirect_to root_url, alert: "問題が発生しました！"
redirect_to root_url, flash: { referral_code: 1234 }
```
### flashの保存
- `flash.keep` メソッドを使うことでflashを保存できる。
```
  def index
    # すべてのflash値を保持する
    flash.keep

    # キーを指定して特定の値だけをkeepすることも可能
    # flash.keep(:notice)
    redirect_to users_url
  end
```
## flash.now
### 概要
- 通常のflashでは、次のリクエストでflashメッセージを利用できるが、同じリクエスト内でflashを参照したいとき（入力の失敗を知らせるflashなど）がある。
- この場合に　`flash.now` を使うと、通常のflashと同様にflashの値を取得できる。
```
class ClientsController < ApplicationController
  def create
    @client = Client.new(client_params)
    if @client.save
      # ...
    else
      flash.now[:error] = "クライアントを保存できませんでした"
      render action: "new"
    end
  end
end

```

## 参考
- https://railsguides.jp/action_controller_overview.html#:~:text=%E3%82%92%E4%BD%BF%E3%81%84%E3%81%BE%E3%81%99%E3%80%82-,5.2%20Flash,-flash%E3%81%AF%E3%82%BB%E3%83%83%E3%82%B7%E3%83%A7%E3%83%B3

# エラーメッセージとの違い
- エラーメッセージはバリデーションエラーなどを扱うもの。
- flashは実行したアクションに対する通知を行うもの。
