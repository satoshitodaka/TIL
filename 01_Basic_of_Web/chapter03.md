# 3. HTTPでやり取りする仕組み

## 01 HTTPメッセージ
- HTTPメッセージによる要求と応答<br>
HTTPにおいて、クライアントとサーバー間のやり取りに利用されるのがHTTPメッセージ

- 2種類のHTTPメッセージ<br>
HTTPメッセージは、クライアントからの要求「HTTPリクエスト」とサーバーからの応答である「HTTPレスポンス」に分類される。<br>
HTTPメッセージの構成　開始行は、リクエスト時とレスポンス時で構文が異なる。

## 02 HTTPリクエストとHTTPレスポンス
- HTTPリクエスト
リクエスト行、メッセージヘッダー、メッセージボディの3つに分ける事ができる。
リクエスト行は、Webサーバーにどのような処理をして欲しいかを伝える。
メッセージヘッダーは、ブラウザの種類やバージョン、対応するデータ形式など付加的な情報を伝える。
メッセージボディは、Webページのフォーム欄等に入力したテキストデータなどをWebサーバーに伝える。

- HTTPレスポンス
Webサーバーがクライアントのリクエストを処理し、その結果をHTTPレスポンスとして返す。
HTTPレスポンスは「ステータス行」「メッセージヘッダー」「メッセージボディ」に分けられる。
ステータス行では、HTTPリクエストに対して、Webサーバー内での処理の結果を伝える。
メッセージヘッダーでは、Webサーバーの種類やデータ形式など、付加的な情報を伝える。
メッセージボディでは、クライアントからリクエストされたHTMLなどのデータが含まれる。

- HTTPレスポンスのHTMLに画像などのリンクが含まれる場合は
再度HTTPリクエストを送信、画像等を取得し、このやりとりの繰り返しでWebページを表示する。

## 03 HTTPメソッド
HTTPリクエストにて、Webサーバーに具体的な処理を伝えるのはHTTPメソッド
HTTPメソッドは複数定義されており、閲覧する場合はGET、データ送信をする場合はPOSTを用いる。
PUTやDELETEメソッドはWebページの改竄につながるため、サーバー側で使えなくしている場合が多い。

- GETとPOSTの違い
GETメソッドは（通常はデータの取得に使うものの）送信に使うこともできる。
GETが送信内容をURLの後ろに組み込む。→閲覧履歴に残る。
POSTメソッドの場合、HTTPリクエストのメッセージボディに送信内容を組み込むため、閲覧履歴に残らない。

## 04 ステータスコード
HTTPレスポンス内には、Webサーバーでの処理結果が含まれ、これをステータスコードという。
ステータスコードは３桁の数字から成り、100番台から500番台までの5つに分類されている。

- ステータスコードの分類
100番台：HTTPリクエストが処理中であることを示す
200番台：HTTPメソッドが正常に処理された場合
300番台：HTTPリクエストに対して、Webブラウザ側で追加の処理が必要な場合。URLが変更された場合など。
400番台：クライアント（ブラウザ）のエラーが発生した場合。要求したHTMLが存在しないなど。
500番台：Webサーバーのエラーが発生した場合。何らかのエラーでリクエストに応えられない、高負荷のため一時的にレスポンスできないなど。

## 05 メッセージヘッダー
HTTPリクエストとレスポンスはそれぞれ、メッセージヘッダーを利用し、HTTPメッセージに関する詳細な情報を伝える。
メッセージヘッダーは複数のヘッダーフィールドから成り立つ。
ヘッダーフィールドは、「フィールド名: 値」で成り立つ。

ヘッダーフィールドは４種類ある
- 一般ヘッダーフィールド
HTTPリクエストとレスポンス両方に含まれる。日付など
- リクエストヘッダーフィールド
HTTPリクエストのみに含まれる。クライアントの固有情報など。
- レスポンスヘッダーフィールド
HTTPレスポンスに含まれる。Webサーバー機能を提供するプロダクト情報を表す「Server」など。
- エンティティヘッダーフィールド
HTTPリクエストとレスポンスいずれにも含まれる。
メッセージボディの付加情報など。Content_typeなど。

## 06 TCPによるデータ通信
HTTPのやり取り（データ転送）はTCPが行う。
TCPはまず最初に、コネクションの確立を行なう。（クライアントとサーバーの通信を確立）
コネクションには3回のやり取りが必要となる。

- クライアントからの接続要求（SYN）
接続を要求するため、クライアントはサーバーに「SYNパケット」を送信する。
- クライアントに対しての確認応答、及びサーバーからの接続要求
サーバーはクライアントにACKパケットを送信、同時にSYNパケットを送信して接続要求
- サーバーに対しての確認応答
クライアントはACKパケットを送信、サーバーに確認応答を伝える。

## 07 HTTP/1.1のやり取り
現在はHTTP1.1が最も広く普及している。
- HTTPキープアライブ
HTTP1.1以降、コネクションの確立後にコネクションを継続して利用するようになった。
それ以前では、確立後に一度コネクションを終了していたため、画像のやり取りのために再度接続する場合に不便だった。

- HTTPパイプライン
複数のHTTPリクエスト送信を可能にする機能。
これがない場合、リクエストの対となるレスポンスを待つ必要があり、効率的な通信ができない。

## 08 HTTP/2のやり取り
HTTP/1.1でも多くの機能改善があったが、データ通信量の増大に伴いHTTP/2が登場した。
Googleが提案したSPDYと呼ばれる通信の高速化を目的としたプロトコルが原型となっている。
現在は標準化されており、HTTP/1.1と比べると多くの改善点がある。

- ストリームによる多重化
HTTPパイプラインの問題点として、リクエストを順に処理するため、一つの処理に時間を要する場合に全体のレスポンスが落ちるという点があった。
ストリームでは、一つのコネクション上に仮想の通信経路を複数作成することにした。
これにより、従来あったリクエスト順番に処理するという制限を回避、全体の処理速度を短縮できるようにした。

## 09 HTTP/2の改良点
- バイナリ形式の利用
バイナリ形式を利用することにより、クライアントやサーバーの処理時間を短縮した。
HTTP/1.1以前はテキスト形式を利用していた。

- ヘッダー圧縮
ヘッダー情報を圧縮できるようになった。HPACKを利用し、差分のみ送信している。
- 従来はリクエストとレスポンスが繰り返されると、データの重複が発生していた。

- サーバープッシュ
リクエストに対し、サーバーが必要な情報を判断し（画像を含むページであれば画像）、（続きの）リクエストがなくても先に送信する機能。
従来は、画像は後からクライアントがリクエストを発し、それに対してサーバーがレスポンスするという仕組みだった。

## 10 HTTPSの仕組み
HTTP Sとは、インターネット上の脅威（盗聴や改ざんなどからデータを守る仕組み。
HTTP over SSL（Secure Sockets Layer）/TLS（Transport Layer Security）の略称。

- 安全性を単t歩する仕組み
3つの仕組みで安全性を担保する。
- 盗聴防止（暗号化通信）
データを暗号化することで第3者のからの盗聴を防ぐ。
Web閲覧はいくつものサーバーを経由するため、実際は通信内容を傍受することは比較的容易。

- 改ざん防止
改ざん防止としてメッセージダイジェストが用いられる。
特定のハッシュ値を抜き出し、送信の前後で相違がないか比較するもの。

- なりすまし防止（Webサイト運用元の確認）
WebサーバーにSSL認証証明書を配置しておき、接続時に検証することで運用会社の身元を確認する。
このSSL認証証明書の発行には、認証局による運用元の認証作業をパスする必要がある。
この証明書を持たないサーバーにアクセスした場合、Web閲覧時に警告が出る。

## 11 HTTPSのやり取り
- SSL/TLSハンドシェイク
HTTPSの通信を開始するには、大きく4つのフェーズが必要。これをSSL/TLSハンドシェイクと呼ぶ
- 暗号化方式の決定
暗号化方式は複数あるため、どれを使うか決定する。
SSLやTLSのバージョン、メッセージダイジェストを確認する。
- 通信相手の証明
クライアントが通信しているサーバーが正しい相手なのか、SSL証明書から確認する。
- 鍵の交換
データ転送に利用する鍵を交換する。この鍵は暗号化やデータの復号に利用する。
- 暗号化方式の確認
暗号化方式の最終確認を行う。

## 12 ステートフルとステートレス
HTTPの特徴として、ステートレスであること。
ステートレス：1回のやり取り（リクエストとレスポンス）は完結しており、前後の処理に影響しない。
ステートフル：1回の処理内容を保持するため、次の処理に情報を反映させる事ができる。1対1のやり取りは問題とならないが、多くのクライアントと通信する場合の負荷は大きい。

- HTTPの弱点
ECサイトでは買い物かごの中身を（ページが遷移しても）保持する必要があるが、HTTPにはその仕組みがない事。

## 13 Cookie
HTTPにおいて、状態を保持するためにCookieが用いられる。
- Cookieのやり取り
Webサーバーに接続してきたクライアントに対し、コンテンツと併せて、ブラウザで保存してもらいたい情報をCookieとして送信する。
クライアントはCookieを保存しておき、次にサーバーと通信する際にCookieを併せて送信する。

- メッセージヘッダーの利用
cookieの送信にはメッセージヘッダーが使われる。
サーバーはHTTPレスポンスに「Set-Cookie」ヘッダーを含め、クライアントは「cookie」ヘッダーに含める。
Set-cookieではcookieの有効期限を設定したり、HTTPSの通信のみで含めるという設定ができる。

- セッションCookie
セッションcookieは有効期限が決められていないcookieで、ブラウザが終了するとcookieは削除される。
有効期限が設定されたCookieはブラウザを閉じても保持され、有効期限到達時に削除される。
セッションcookieはECサイトなどで利用される。

## 14 セッション
クライアントとサーバーのやり取りにおいて、一連の関連性のあるやり取りをセッションと呼ぶ

- セッションの管理
セッションの管理はCookieを用いる。
セッション管理において、サーバーがブラウザを識別するためにセッションIDを使う。
セッションIDはサーバーが生成し、Cookieを含めてクライアントに送信する。
セッションIDを受け取ったクライアントは、次回以降の処理にセッションIDを含める事で、サーバーとセッションを維持できる。
セッション内の処理内容は、セッションデータとしてサーバーに保存される。
クライアントはセッションIDを用いて、この自身のセッションデータを参照できる。

- セッションのやり取り
一部のブラウザではcookieが使えないため、その場合はURLに組み込む、フォームに埋め込む事があるが、セキュリティ上望ましくない。

## 15 URI
情報やソースを識別するための記述方法をURI（Uniform Resource Identifier）という。
URIのうち、場所を示すものをURLという。URLは場所だけでなく、取得方法を示すこともある。

- リクエストURI
HTTPにおいてもURIが利用されており、リクエスト行のメソッドに続いて記述される。リクエストURIとも呼ばれる。
リクエストURIにはURI全てを含める絶対URI、記述を簡略化した相対URIがあり、通常は相対URIが用いられる。

- パーセントエンコーディング
URIに利用できる文字は定められており、予約文字と非予約文字がある。
予約文字（!、#、$など）は特定目的に利用するための文字で、決められた目的以外では使えない。
非予約文字（半角英数字など）は、自由にURIに使える文字のこと。
上記に該当しない文字でURIを記述する場合は、パーセンテージエンコーディングを用いて文字を変換する。
％の後に該当する文字を16進数で表し、「%xx」の形式に変換する。