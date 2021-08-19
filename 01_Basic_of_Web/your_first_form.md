# 初めてのフォーム

## Webフォームとは何か
- ユーザーがデータをWebサイトに送（伝達）する仕組み
- Webフォームは一つ以上のフォームコントロールと、フォーム全体を構成するのに役立つ構成要素（HTMLフォーム）を持つ。コントロールには、テキストフィールド、ドロップダウン、ボタン、チェックボックスがある。

## フォームを設計する
まず最初に、シンプルなモデルを作り、ユーザーに入力を依頼したい内容を検討する。<br>
UXが大規模になると、ユーザーはフォームから離れてしまう。必要最低限のことに集中すること。

## HTMLを書く
- フォームを作るためには以下の要素が必要<br>
<form><label><input><textarea><button>

### <form>要素
- 全ての要素はform要素から始まる。
- divやpと同様にコンテナ要素
- 全ての要素は省略可能だが、action(URLを指定)とmethod(HTTPメソッドを指定)は記述するのがよい。

### <label><input><textarea>でウィジェットを追加する。
サンプルのテキストフィールドについて。<br>
- liタグはコードを扱いやすく、構造化するもの。またスタイル設定にも便利。
- labelタグのfor属性は、ラベルとフォームウィジェットを関連づける。
- inputタグで最も重要なものはtype属性で、見た目や動作を定義するもの。<br>
nameを指定すれば単一行でなんでも入力できるフィールド<input/text>を示す。<br>
emailを指定すれば単一行の<input/email>を示し、データの入力・チェックがemailに最適化される。

<input>は空要素のため、終了タグは不要。<br>
<textarea>は空要素ではないので、終了タグが必要。<br>
<input>タグにデフォルトの値を定義したい場合、inputタグ内でvalueの値を定義する。<br>

### <button>を追加する。
ユーザーが入力したフォームの送信のためには、buttonタグを追加する。<br>
button要素は3種類のtypeを指定できる。Submit,reset, button<br>
- submitでは、form要素のaction属性で定義したサイトに送信する。<br>
- resetでは、全てのフォームの入力値をデフォルト値に戻す。UX的には良くないため、非推奨。
- buttonは何もしないが、JSでカスタムボタンを作るときに役立つ。

## CSSで見栄えを良くする。
HTMLのheadタグ内にCSSを追加できる。

## データをWebサーバーに送信する。
- form要素は、action属性とmethod属性により、どこへどのようにデータを送信するか定義する。
- フォームコントロールに名前をつけるが、これはクライアントとサーバーどちらにとっても重要。<br>
名前をつけるには、フォームウィジェットのname属性を使用する。

## イマイチピンと来なかったので自分メモ
```
<form action="/my-handling-form-page" method="post">
 <ul>
  <li>
    <label for="name">Name:</label>
    <input type="text" id="name" name="user_name">
  </li>
  <li>
    <label for="mail">E-mail:</label>
    <input type="email" id="mail" name="user_email">
  </li>
  <li>
    <label for="msg">Message:</label>
    <textarea id="msg" name="user_message"></textarea>
  </li>
 </ul>
</form>
```
- 全てのフォームは<form></form>で囲む。
- actionでURLを指定できる。（どこの場所に対して処理するか）
- methodでHTTPメソッドを指定する（どんな処理をするか）
- <ul><li>タグは構造を整理するのに便利
- <label>タグで囲むと、フォームをラベリングできる。（見出しをつけれる）<br>
  for属性を指定することで、<input>や<textarea>と紐付けできる。<br>
  <input><textarea>側ではidを指定し、<label>と紐付ける。<br>
- <input>は単一行のフィールド。type属性を指定することで、入力フォームが最適化される。（見た目や入力の制約、呼び出されるキーボードなど。例：メールの入力）
- <textarea>は複数行の入力フィールドで、あらゆるテキストを入力できる。
- idは、labelとフォームウィジェット紐付けるために指定するもの。
- nameは、入力されたデータの名前を示すもの（モデルのカラムに該当？）

