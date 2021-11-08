- データの値の制限をDB側だけで定義するのは少々不都合があるため、モデル側でも制限や検証を行う必要がある。DB側で保存不可の値を保存する場合、DB側では例外が発生し保存できないが、ユーザーはその内容を知ることができない。データの細かな制約などはモデルで定義する方が扱いやすい、など。
- データの内容が正しいかチェックする仕組みを検証（validation）といい、Railsにもこれが備わっている。

## 1 モデルの検証の仕組み
- Railsにおける検証は、モデルオブジェクトの登録や更新の際に検証を行い、エラーがあれば差し戻すというもの。この仕組みはsaveメソッドにより実行される。
- saveメソッドは実行の際（オブジェクトの登録や更新）の際に検証を行い、検証エラーがあればfalse、問題なければtrueを返す。検証エラーの内容はerrorsを使って確認できる。予期せぬエラーが発生した場合は例外を発生させる。
- 検証を単体で行う場合はvalid?メソッドを使う。
- saveメソッドに`(validate: false)`をつけることで、意図的に検証を回避できる。

## 2 検証の書き方

## 7 検証が行われない操作
- 検証はモデルの登録や更新の際に自動で実行されるが、検証が行われない登録や更新メソッドもある。