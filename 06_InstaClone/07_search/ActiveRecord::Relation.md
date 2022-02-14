# ActiveRecordとは
- MVCのModelに該当する部分で、DBとのやりとりに関する機能を提供する。
- オブジェクト/リレーショナルマッピングを用いることにより、わずかなコードを書くことでDBを扱うことができる。
- 重要な機能は以下の通り(Railsガイドより抜粋)
  - モデルおよびモデル内のデータを表現する
  - モデル同士の関連付け(アソシエーション)を表現する
  - 関連付けられているモデル間の継承階層を表現する
  - データをデータベースで永続化する前にバリデーション(検証)を行なう
  - データベースをオブジェクト指向スタイルで操作する

# ActiveRecordのメソッドは大きく２種類に分類できる。
## 1.すぐにSQLを発行し、DBにアクセスしてレコードを取得するメソッド
- `find, find_by, take, first, last, exists?`　など
## 2.それ以外の検索メソッド
- `where, limit`など。メソッドの実行時点ではSQLを発行せず、ActiveRecord::Relation を返す。
- 実際にデータが必要になった時点でSQLを発行する。遅延評価（Lazy Lazy Evaluation）という。

# ActiveRecord::Relationとは
- クエリを発行するための情報を保持するもの。
- メソッドチェーンでつなげることができるため、再利用性が高い。
- 必要になった時にクエリが発行され、必要なオブジェクトを取得することができる。

# 参考
[Active Record の基礎](https://railsguides.jp/active_record_basics.html)

[ActiveRecord各メソッドのクエリ実行タイミングについて](https://qiita.com/ykamez/items/0c81a33ec1b90219d541)

[ActiveRecord::Relationとは一体なんなのか](https://spirits.appirits.com/doruby/8831/?cn-reloaded=1)
