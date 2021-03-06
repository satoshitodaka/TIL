#### 2-7 ゲッターやセッターを設定する。
- Attr_accessorメソッドでシンボルを指定すると簡単に設定できる。
- Attr_readerではゲッターを、attr_writerではセッターだけ設定できる。

#### 2-8 ローカル変数とメソッドの見分け方
- メソッドの中において、ローカル変数とメソッドの名前が重複している場合、それはローカル変数として扱われる。そのメソッド？ローカル変数？がメソッド内で定義されていればローカル変数、そうでなければ他のメソッド、という判断が良い。
## 6 プログラムの異常を検知する（例外補足）
- プログラムが意図通り動作しない時、例外が発生することがある。
## 7 Rubyっぽい書き方
#### 7-1 nilガード
- 以下は同じ働きをする。
- もしnumberの値があればそれで、なければ10を代入したnumber、という意味。変数にnilが入っているかもしれない状況で非常に便利に使える。
```
Number ||= 10
Number || (number = 10)
```
#### 7-2 ぼっち演算子
- &.という演算子を使うと、レシーバがnilの時にエラーが出なくなる。
- 正式名称はsafe navigation operaterという。
- ぼっち演算子は、ifや三項演算子で表す内容を簡潔に表すことができる。
```
Name = if object
Object.name
Else
Nil
```
```
Name = object ? bject.name : nil
```
```
Name = object&.name
```
#### 7-3 %記法
- 全ての要素が文字列の配列の場合、%wを使って表記できる。括弧の中にはカンマやクオーテーションも要らない。
```
Array = %w(apple banana orange)
```
- 全ての要素がシンボルである配列は、%iを使って表記できる。同様に、括弧の中にはカンマやクオーテーションは要らない。
```
Array = %i(dog bird cat)
```
- これらを％記法という。これのメリットとしては、コードの記述量が少し減ること、可読性が上がる場合がある、という点。
#### 配列の各要素から特定の属性だけ取り出す
- 配列の要素から特定の属性を取り出すには、eachやmapメソッドが有効である。
```
Names = []
Users.each do |user|
Names << user.name
end
```
```
Names = users.map do |user|
User.name
end
```
- ブロックはdo〜endを{}に書き換えられるので、以下と同じ動作をする。
```
Names = users.map { |user| user.name }
```
さらに、＆とメソッドのシンボルを使って簡潔にできる。
```
Names = users.map(&:name)
```
