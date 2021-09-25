## 1 セッションとCookie
#### 1-1 セッション
- HTTPはステートレスなプロトコルなので、前後の処理を引き継ぐことができない。そのため、サーバ側でセッションという仕組みを用意し、ブラウザとの一連の処理情報（状態）を保持している。
- Railsでは、コントローラからSessionメソッドを呼び出すことでセッションにアクセスできる。sessionはハッシュのように扱うことができる。
#### 1-2 Cookie
- Cookieとは、状態をブラウザ側でも保存できる仕組み。最初のHTTPレスポンスの際にサーバーがCookieを付与してレスポンスする。ブラウザがこれを保存し、次のリクエストに付与することにより、状態が保持できる。
- Railsでは`cookie`メソッドを使うことによりCookieにアクセスできる。
- Railsにおいて、セッションはCookieのやり取りに含まれるセッションIDを活用している。また、セッションの保存先は複数あり任意のものを選択できるが、デフォルトではCookieに保存する。

## 2 Userモデルを作る
#### 3 パスワードを受け付けてdigestを保存する
- Userモデルに安全なパスワード認証を導入するため、`has_secure_password`を導入する。パスワードのハッシュ化の際にbcryptを利用するため、Gemfileに記載してインストールする必要がある。
- モデルの中に`has_secure_password`を記述するだけ。記述すると、モデルにpasswordとpassword_confirmationの二つの属性が追加される。
```
class User < ApplicationRecord
has_secure_password
end
```
- ユーザーがパスワードを入力する際は、passwordとpassword_confirmationを入力させ、一致した場合はpasswordをハッシュ化したものをpassword_digestとしてDBに保存する。passwordとpassword_confirmationが一致しない場合は検証に失敗し、登録（認証）できない仕組み。

![Image from Gyazo](https://i.gyazo.com/5c955c4cb2135d44d46213a8607b7f70.png)

## 4 ユーザー管理機能一式を追加する 
- アプリケーションのユーザー登録には２通りある。一つが、ユーザーが新規登録画面で登録する（SNSなど）。もう一つは管理者がユーザーを登録するもの（ユーザーの範囲が限定されるもの）。
- 後者においては、管理者だけにユーザーの改廃機能を付与する。URL`/admin`にアクセスし利用できる機能を実装するやり方がある。
#### 2 ユーザー管理のためのコントローラを実装する。
- 管理系の機能を作るにあたり、Adminというモジュールのnamespaceの中にコントローラを定義する。これにより、コントローラのフォルダ階層の中にまとめて定義されるようになる。
```
$ bin/rails g controller Admin::Users xxx xxx xxx
```
```
Rails.application.routes.draw do
namespace :admin do
  resources :users
end
root to: 'tasks#index'
resources :tasks
end
```
## 5 ログイン機能を実装する

## 8 ログイン情報の取得を簡単にする
- ユーザーがログイン中は以下のコードでログイン情報を簡単に取得できる。
```
User.find_by(id: session[:user_id])
```
- ログイン情報の取得は全てのコントローラにて行いたい、かつビューからも取得したいのでApplication Controllerに定義する。
```
class ApplicationController < ActionController::Base
helper_method :current_user
private
  def current_user
    @current_user ||= User.find_by(id: session[:user_id]) if session[:user_id]
end
```
## 9 ログアウト機能を実装する
- sessionの`user_id`だけを削除するには以下で問題ないが、ユーザーに紐づく様々な情報をsessionに入れている場合、全体をリセットするのが良い。
```
session.delete(:user_id)
```
```
reset_session
```
