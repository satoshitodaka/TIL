# Scopeとは
- 頻繁に使うクエリを、関連オブジェクトやモデルへのメソッド呼び出しとして使用できるようにするもの。
- `where, includes, joins`など、様々なクエリメソッドが使用できる。
- どのscopeメソッドも、ActiveRecord：：Relationオブジェクトを返す。
  - scopeでは別のscopeメソッドを呼び出すため、`ActiveRecord::Relation`もしくは`nil`を返すようにすべき

# ActiveRecord::Relation

# 参考
[Railsガイド　スコープ](https://railsguides.jp/active_record_querying.html#%E3%82%B9%E3%82%B3%E3%83%BC%E3%83%97)
