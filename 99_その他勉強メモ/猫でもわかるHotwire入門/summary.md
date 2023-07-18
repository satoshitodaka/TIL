# Hotwireとは
- [Hotwireについて](/Users/satoshitodaka/my_app/TIL/07_InstaClone_ver7/01_settings_rubocop_rspec_ci/Hotwire.md)
- モダンなWebアプリを作るための、Rails7からデフォルトになったアプローチ、以下3つから構成される。
  - Turbo
    - Turbo Drive
    - Turbo Frame
    - Turbo Stream
    - TUrbo Native
  - Stimulus
  - Strada

## 特徴
- サーバーがHTMLをレスポンスする点
- Hotwireを使うと、フォームやリンクからのリクエストはFetchAPIを使った非同期リクエストになり、これに対してサーバーがHTMLレスポンスを返す。
- 対して、フロントにJSを使うと、サーバーはJSONを返すため、実装が複雑になる。
  - クライアントでDOMを構築する必要があるし、バリデーションを設定したりする必要が出てくる。
- Hotwireを使うと、状態管理やモデル、バリデーションの実装はサーバーサイドだけで良いし、既存の資産も活用できる。

## Turbo
- Turbo
  - Turbo Drive
  - Turbo Frame
  - Turbo Stream
  - TUrbo Native
### Drive
- Turbo Driveは画面遷移を高速にするもの。基本的にはTurbolinksと同じ機能。
- リクエストをTurboDriveがインターセプトし、fetchによる非同期リクエストに差し替える。そうすることで、レスポンスのbodyタグだけを抜き出し、現在のページのbodyタグを置換する。正確には、bodyタグの置換とheadタグの一部をマージする。
  - この挙動の利点は、置換前のページのCSSとJSをそのまま適用できること。初期化してページに適用する処理をスキップできる。

### Frame
- Driveの部分置換版。`<turbo-frame>...</turbo-frame>`の部分を置換する。
- 一箇所しか更新できないという制約があるので、複数箇所更新する場合はStreamを使う。

### Stream
- 複数箇所のHTML要素を更新できる。
- frameがHTML要素の置換しかできないのに対し、streamは追加更新削除ができる。