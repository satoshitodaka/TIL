## GitHub(日本語)
https://github.com/slim-template/slim/blob/master/README.jp.md

## Qiita
- https://qiita.com/yterajima/items/53fd0387279510ff082a
- https://qiita.com/pugiemonn/items/b64171952d958dc4d6be

## 記法メモ
#### 共通
- タグを記述する際は<>が不要
- 閉じタグ（`</p>`など）は不要
#### -
- ダッシュは制御コードを示し、ifやループなどに使われる。
- ブロックはインデントで表記し、`end`は用いない。
- ERBでは`<% %>`と表すもの
#### = 
- イコールは出力を示す。
- ERBでは`<%= %>`と表すもの

#### /
- スラッシュ`/`はコメントを表し、以降のテキストがコメントとなる。
- `/!`はHTMLコメント

#### テキストと|（パイプ）
- <>を除いたタグの後に半角スペースを空け、記述したものがテキストとなる。
```
p Hello, World!!
```
- |(パイプ)の後の記述はテキストとして扱われる。複数行にまたがるテキストやインデントなどに用いる
```
p |
  Hello, World!!
  Hello, World!!
  Hello, World!!
```
#### idの指定
- `#`の後に文字列を記述しidを指定する。
```
#user
```
```
<div id='user'></div>
```
#### クラスの指定
- 指定するクラスの前に`.`をつける。
- 複数のクラスを指定する場合は`.`で繋げて書く。
```
.btn.btn-primary
```
```
class= 'btn btn-primary'
```
