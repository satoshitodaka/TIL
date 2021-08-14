# Web技術とは

## 01 Webとは
文書の公開・閲覧のためのシステム
- Www
webの正式名称はwwwでworld wide webという。
Web上の文書はハイパーテキストと呼ばれる言語で構成されている。
ハイパーテキストは、Webページの中に別のwebページへの参照（ハイパーリンク）を埋め込む事ができる。

- 世界中の情報を蜘蛛の巣のようにつなげる事ができる。
大きな特徴として、Webページ同士がリンクで別のwebページに繋がっている事
最終的には世界中のあらゆるページと繋がることになる。
	
## 02 インターネットとWeb
インターネットとWebは元来別々の目的のために開発された
- CERNで開発
webはCERNで開発された。元は各国の研究者が素早く情報にアクセスできるようにするために製作された。
ENQUIREという名称で開発されたが、改良されてwwwとなった。

- インターネットの普及に貢献
インターネットは米国防省で制作されたARPANETが原型
ARPANETは徐々に拡大、通信方法（プロトコル）を見直し、インターネットとして世界に広まるようになった。
当初は高価であったため研究機関等で利用されていたが、徐々に安価になっていった。

この頃。webは文書だけでなく画像等も使えるようになり、webブラウザの普及もあり利用が広まっていった。
	
## 03 Webの様々な活用
Webには様々な用途がある。
- 文書の閲覧
一つのドメインがあるサイトの集まりをWebサイトという
ページ同士はハイパーリンクで繋がり、相互に行き来できる。

- ユーザーインターフェース
ユーザーの操作とコンピュータの機能を橋渡しする。

- プログラム用API
ソフトウェア同士のやり取りの橋渡しをする機能をAPIという。
	
## 04 HTMLとWebブラウザ
- 記述言語HTML
ハイパーテキストを記述するための言語がHTML（Hyper Text Markup Languege）
HTMLでは、表示方法やハイパーリンクをタグと呼ばれるマークによって表現する。
<タグの種類>表示する内容</タグの種類>といった形で表現する。

- 表示プログラムWebブラウザ
ハイパーテキストは文書にタグづけしたもので、人間が読むには適さない
ブラウザを通すことで、人間が読めるようになる。

## 05 WebサーバーとHTTP
- 配信プログラムとWebサーバー
Webサーバーは、Webブラウザからリクエストがあると、必要なコンテンツをネットワークを通してブラウザに送信する役割を持つ。
コンテンツは、Webサーバーをより配信されることで、Webページと呼ばれるようになる。

- やり取りの手順　HTTP
Webページが表示される手順として、ブラウザがWebサーバーにリクエストを送信し、WebサーバーがWebブラウザにコンテンツを送信することで表示される。
このコンテンツのやり取りの手順やメッセージの書式は世界共通のものが定められていて、HTTP（HyperText Transfer Protocol）という。
HTTPでは、ハイパーテキストの要求手順、送信手順の他に、要求されたコンテンツを持っていなかった場合の応答や
Webサイトが移転した場合のブラウザへ伝える方法など、様々な事柄が決められている。
HTTPが使用されていることで、あらゆるブラウザであらゆるページにアクセスできる。

## 06 Webページが表示される流れ
- URLを使ってWebページにアクセス
ユーザーはWebページを取得する際、URL（Uniform Resouce Locator）で表示したいページをブラウザに伝える。
URLに含まれる情報は、どのやり取りの手順で」「どのWebサーバーに」「どのコンテンツを表示するか」を含む。
Webブラウザは、URLを元にWebブラウザからコンテンツを取得し表示する。
Webで使われる手順は、HTTPとHTTPS（HTTPのセキュリティを高めたもの）がある。

- Webブラウザで解釈
Webブラウザは転送されたコンテンツを解釈し、タグの内容を元に人間が読みやすいように表示する。

- さらに他の画像などを転送
一回のコンテンツ転送で送れるファイルは一つであるため、画像等がある場合は
何度かWebサーバーとやり取りし、Webページを完成させる。

## 07 静的ページと動的ページ
- 静的ぺーじ
何度アクセスしても同じ内容が表示されるページ（企業のHPなど）
元は研究用資料の共有のためであり、静的ページで十分であったが、
Webの普及と利用機会拡大に伴って豊かな表現が求められるようになり、動的ページの技術が生まれた。

- 動的ページ
アクセスの際に状況に応じて異なる内容が表示されるページを動的ページという。（検索サイト、会員制サイト、SNSなど）

## 08 動的ページの仕組み
- CGI(Common Gateway Interface)
WebサーバーがWebブラウザからのリクエストに応じてプログラムを立ち上げるための仕組みをCGIという。
動的ページを利用する際、WebブラウザはOGIが用意された場所を示すURLにアクセスする。
要求を受信したWebサーバーはCGIによってプログラムを起動する。
ブラウザから送信されたデータや、プログラムが持つデータからHTMLファイルを生成し
Webサーバーを通してWebブラウザに送信する。

- サーバーサイドスクリプト
CGIから呼び出されるプログラムはサーバーサイドスクリプトと呼ばれる。
基本的にはなんでもOKだが、文字列の扱いに長けたスクリプト言語が用いられる。PerlやRuby、PHP、Pythonなど

- クライアントサイド・スクリプト
サーバーサイドに対し、HTMLに埋め込まれWebブラウザで読み込まれる際に実行されるすクリプトとして、クライアントサイト・スクリプトがある。
JavaScriptなど

## 09 Webの標準化
- 標準化とは
Webで用いられる技術は、Webの発展に伴って機能拡張や新技術の開発が行われてきた。
この開発を各々が独自で行うと利用・開発に支障が出るため、規格を決める必要がある。この作業を標準化という。

- 標準化を進める団体
Webの標準化を進める団体はW3Cと呼ばれる団体。
W3CはHTMLのほか、CSSやXML、XTMLの標準化を行っている。
標準化したものは「勧告」という形で発表される。強制力はないが、ほとんどのWebサイトが準拠する。

## 10 Webの設計思考
標準化されたもの意外にも、Web技術については様々な設計についての思考がある。
- RESTful
RESTfulとは4つの原則からなるシンプルな設計を指す。(Representational State Transfer)
この原則に基づいて作られたシステムをRESTfulなシステムという。
シンプルな構造であり、やり取りの方法や情報の示し方が統一されており、一つのデータに別のデータをリンクさせられるため
RESTfulなシステム同士では円滑な情報の連携が行える。

- RESTfulの原則
・統一インターフェース　予め統一・共有された方法で情報をやり取りする。
・アドレス可読性　全ての情報が一意なURLの構文で示される。
・接続性　やり取りする情報にはリンクを含める事ができる。
・ステートレス性　やり取りは1回で完結、前後のやり取りの影響を受けない。

- セマンティックWeb
Webページの情報に意味（セマンティック）を付与すること。
これにより、コンピューターがページに付与された意味を理解し、処理できる事が期待されている。
セマンティックWebでは、XMLという言語によってページを構成する。
XML文書の中ではRDFという言語で意味を記述し、OWLで言語の相関関係を記述する。
これらの情報に意味を付与する情報をメタデータと呼ぶ。
既存のページへのメタデータ付与は、まだまだ先になりそう。