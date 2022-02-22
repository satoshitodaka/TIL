# enum
### enumとは
- enumとは、一つのカラムに複数個の定数を保存できる仕組み
  - これを使うと、指定した定数以外の値は保存できなくなる。
  - カラムに定数が入っているレコードを取り出すのが容易になる
```rb
enum blood_type: { A: 0, B: 1, O: 2, AB: 3 }
```
### 実装について（integer型)
- enum用のカラムは、integer,booleanいずれかの型となる。
  - ２個以上の定数を保存する場合はinteger型を使用すること。
- その他、enumを配列で定義することもできる。

### enumのメソッドについて
#### 確認メソッド
```
インスタンス.定数（カラム） #=> 定数名を返す。
インスタンス.定数名　#=> 定数名が入っていればtrue、入っていなければfalseを返す。
```
#### 更新メソッド
```
インスタンス.定数名! #=> 定数名を更新する。
```
#### 検索メソッド
```
モデルクラス.変数名　#=> 定数名をもつインスタンスの全てのデータを取得できる。
```


### 参考
[【Rails】 enumチュートリアル- pikawaka](https://pikawaka.com/rails/enum)

[ActiveRecord::Enum](https://api.rubyonrails.org/v5.2.4.4/classes/ActiveRecord/Enum.html) ※あまり読めてないので、使うときに読んでみる。
