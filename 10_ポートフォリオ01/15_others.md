# その他メモ
## デバッグについて
- Rails7において、esbuildを使っている場合はデバッグの方法がやや特殊
### なぜ
- Railsの起動の際に、Procfile.devを使用する
- ProcfileにはRails本体だけでなくcssやJSの起動コマンドが含まれており、これらがすべて実行されるようになっている。そのため、Railsの挙動だけ止めてもインタラクティブに操作することができない。
### どうするか
- ProcfileでRailsの起動コマンドをコメントアウトし、`docker-compose up -d`で立ち上げる。
- 別のターミナルを追加し、`docker compose exec web bash`でコンテナに入る。
- コンテナ内で`bin/rails s -b '0.0.0.0'`を実行してRailsサーバーを立ち上げる。

## Dockerコンテナが立ち上がらない不具合の解消
- 別のブランチで、MySQLイメージのバージョンを変えて、そのボリュームが残って影響を及ぼしていたっぽい。ボリュームを削除したところ解消（エラーの内容が変わった）
- Dockerコンテナのタイムゾーンを設定するコマンドがもしかしたら間違っていたかも。TZの変数についてエラーが出たので、一旦コマンド`command: mysqld --default-time-zone=Asia/Tokyo`を削除したところ立ち上がった。
> [DockerでMySQL5.7コンテナを使う際の注意点](https://www.code-mogu.com/2020/12/10/docker-mysql/)