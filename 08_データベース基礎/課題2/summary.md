# 課題
「スタッフによる餌やりイベント」をテーブルに落とし込んでください

### 要件
- イベントでは一人のスタッフが複数の動物に餌を与えます
- 一回の餌やりではいろんな餌（にんじんとキャベツ etc）を与えることがあります
- 餌やりイベントは一日のうちに複数回あります

# 設計メモ
### 要素を書き出してみる
- 動物園
- スタッフ
- 餌やりイベント
- 動物
- えさ

### 概念のパズルを作る

[![Image from Gyazo](https://i.gyazo.com/c67935c5130a4fa680d4481e041c5cfb.jpg)](https://gyazo.com/c67935c5130a4fa680d4481e041c5cfb)

### ER図に書き出す・正規化する

[![Image from Gyazo](https://i.gyazo.com/d35e97972c838117b685c40ec703c67c.png)](https://gyazo.com/d35e97972c838117b685c40ec703c67c)

### だいそんさんからの指摘（提出2回目）
[![Image from Gyazo](https://i.gyazo.com/7b1a401d2dd9ddf058c3cc125ec1a494.png)](https://gyazo.com/7b1a401d2dd9ddf058c3cc125ec1a494)

最初に提出した内容だと、イベントごとに動物のデータを登録する必要があり、重複する。
イベントと動物は多対多の関係なので、中間テーブルを作る必要がありそう。

動物とイベントのほか、スタッフと餌についても中間テーブルを作成してみた。
[![Image from Gyazo](https://i.gyazo.com/12b23520bcbd390b9e2e2ce839abd99e.png)](https://gyazo.com/12b23520bcbd390b9e2e2ce839abd99e)

上記の書き方で問題なく管理できると思うが、実際の運用では「動物〇〇に餌〇〇を与える」と管理すると想像するので、動物と餌の中間テーブルを作成し、これをイベントに紐付けるのが良い気がする。（[Shunさんの解答](https://tech-essentials.work/courses/1438/tasks/6/outputs/1342)が参考になった）

以下は更に書き直したもの
[![Image from Gyazo](https://i.gyazo.com/1872c14db80b5e170daf41341727740f.png)](https://gyazo.com/1872c14db80b5e170daf41341727740f)

### 提出3回目
結局event_animalとevent_feed は分けて管理する。
また、不要なカラムを削除したもの。
[![Image from Gyazo](https://i.gyazo.com/cfc146a8e3e746037a3d6212ec6cdd41.png)](https://gyazo.com/cfc146a8e3e746037a3d6212ec6cdd41)

