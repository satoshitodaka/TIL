## 4-1 オブジェクトをまとめて扱う
#### 配列とは
- オブジェクトをまとめて扱う部品。
- 配列自身もオブジェクトで、Arrayオブジェクト。
#### 配列を代入する変数名は複数形にする。
- 慣習として、配列を代入する変数は複数形をつける。
## 4-2 要素を取得する
- 変数の後に[n]とつけることで、配列のn番目の要素を指定できる。
- 配列の中身を指す番号は0からカウントする。
- 後ろからカウントする場合はマイナスを使う。
- Firstメソッドは[0]、lastメソッドは[-1]を同等。
##  4-3 要素を追加・削除する。
#### 要素を追加する。
- 要素を末尾に追加する場合はpush、先頭に追加する場合はunshiftメソッドを使う。
- pushメソッドは<<と同等。（末尾に追加される）
#### 要素を削除する
- 要素を末尾から削除する場合はpop、先頭から削除する場合はshiftメソッドを使う。
#### 配列を足し算する
- 配列同士は足し算ができる。
#### 配列を引き算する
- 配列の引き算においては、引かれる配列から、引く配列の要素を除き、残った要素が返される。
## 4-4 配列を繰り返し処理する
#### 繰り返しを途中で終わらせる break
- breakを使うと繰り返し処理を途中で終わらせることができる。
```
[1, 2, 3].each do |x|
Break if x == 2
Puts x
```
#### 繰り返しの次の回へ進む next
- nextを使うと、その回での処理を終了し、次の回へ進む。
```
[1, 2, 3] .each do |x|
Next if x == 2
Puts x
end
```
#### 範囲を指定して繰り返す
- 配列で範囲を示す場合、3..5と記述することができる。これはRangeオブジェクトともいう。
