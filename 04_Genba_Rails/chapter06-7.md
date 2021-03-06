## 1 Strong Parameters
- モデルの機能に、マスアサインメントという機能がある。複数の属性を一括で代入できるもの。この属性の代入の際に、悪意あるユーザーが意図しない属性を送信した場合、不適切な挙動をする場合がある。これを防ぐため、Strong Parametersとういう機能があり、属性の許可・不許可を設定することができる。パラメータを扱うのはコントローラの役目のため、コントローラに記述する。
- paramsの後に続くrequireでモデルを指定、permitで許可する属性を指定する。permitで指定した属性について、不足（送信されなかった）の場合は例外は発生しない。
## 2 CSRF対策
- Cross Site Request Forgeryといい、別サイトのリンクや画像表示をきっかけに、リクエストを偽造することで悪意ある操作をすること。
- railsアプリではCookieを使って本人確認を行う。Cookieは本来、ホストとなるアプリしか取得（ブラウザからの送信を受け取る）できないが、不正なリンクを踏むなどの操作から、Cookieを取り出すことができ、これが悪用される場合がある。
- CSRFの対策として、本人の操作かどうかを確認する必要があるため、セキュリティトークンを発行するなどの対策が取られる。
- railsアプリではform_withなどのヘルパーメソッドを使うことで自動的に対策がなされるが、Getメソッドでは対策されない仕組みとなっている。情報を書き換える操作の場合、GETは使用を避ける必要がある。
#### 2-1 ajax
- Railsでは、ajaxリクエストでもセキュリティトークンをサポートしている。
- 仕組みとして、JSが動く画面で予めセキュリティトークンを用意しておき、JSへの引き渡しと送信が行われる。

## 3 インジェクション
#### 3-1 XSS(cross site scripting)
#### 3-2 SLインジェクション
- whereメソッドの引数にユーザーの入力値を含めると、SQLインジェクションに対する脆弱性となる可能性がある（避けること）
- SQLインジェクションを回避するためには、入力値をエスケープするなどの処理をして使う必要がある。railsでは、クエリメソッドに対してハッシュで条件を指定すると、自動的に安全化の処理を実行する。
- ハッシュでは作れないSQLを使用する場合は、プレースホルダーを利用するようにする。
