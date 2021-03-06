## 7-1 メソッドを作って呼び出す
#### メソッドとは
- 特定の処理に名前をつけて、利用できるもの（部品のようなイメージ）。
- 既にあるメソッドを利用することで、ゼロから処理を書く必要がなくなる。
#### メソッドを定義する
- 自分でメソッドを作ることを、定義するという。
- def の後にメソッド名を書き、endを書いて終わる。ブロックではないのでdoは不要。

## 7-2 メソッドへオブジェクトを渡す
#### 引数を使ってオブジェクトを渡せるメソッドを定義する
- メソッド名の後の（）の中に変数を書くと、引数を渡せる。
- 渡した変数はメソッドの中で使用できる。
- メソッドを呼び出す際は、area(2)のようにメソッド名の後に括弧書きで引数を書く。
#### メソッドを途中で終わらせる -return
-  メソッドの途中にreturnを記述すると、処理がそこで終了する。
- returnに条件を付与することで、処理を分岐することができる。
```
def thanks_and_receipt(receipt)
  puts 'ありがとうございました。'
  return unless receipt
  puts 'こちらがレシートです。'
end
``` 
```
thanks_and_receipt(false)
```
- returnには戻り値を指定する機能もある。Returnの後に戻り値を書くと、表示ささせることができる。
#### メソッドの()は省略できる。
- 曖昧でない（他の解釈ができない）場合は（）を省略できる。
- 定義する場合も（）は省略できるが、慣習的に省略しないことが多い。ただし、引数が０個の場合は省略することが多い。

## 7-3 引数の便利な機能を使う
#### `引数を省略したときのデフォルト値
- メソッドの引数はデフォルト値を指定することができる。
```
def order(item = 'コーヒー')
  "#{item}をください"
end
puts order
puts order('カフェラテ')
puts order('モカ')
```
##### 引数の順番を変えられるキーワード引数
- キーワード引数を使うと、引数に意味を付与するため、メソッドに渡す引数の順番の影響を受けなくなる。
```
def order(item:, size:)
  "#{item}を#{size}サイズでください"
end
puts order(item: 'カフェラテ' , size: 'ベンティ')
```
- 引数が複数ある場合はキーワード引数の利用を検討するのが良いが、呼び出し時の記述量も増えるというデメリットがある。
#### キーワード引数でのデフォルト値
- キーワード引数でもデフォルト値を指定できる。記述する際は、引数名: デフォルト値と書く。通常の引数の時のように=と書かないこと。

## 変数には見える範囲がある
#### ローカル変数とスコープ
- うっかり同じ名前の変数を設定した場合、互いに影響しない仕組みが必要。
- メソッドの中で定義した変数は、メソッドの外では使えない。
- 同様に、メソッドの外で定義した変数は、メソッドの中では使えない。
- 変数の利用可能範囲（見える範囲）と寿命には制限があり、これをスコープという。 
