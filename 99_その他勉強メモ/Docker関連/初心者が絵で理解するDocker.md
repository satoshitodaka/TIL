# 初心者が絵で理解するDocker
## イメージとコンテナ
### docker run docker ps docker exec
- ターミナルでdocker runを実行すると、イメージを指定してDockerに接続できる。
  - 引数としてイメージを渡す。
  - 起動後はdocker psを実行すると、Dockerのプロセスを確認できる。
- docker exec は、コンテナを指定してコンテナ内でコマンドを実行する。

### docker build
- docker buildは、Dockerfileを指定してイメージを作成する。
  - デフォルトでカレントディレクトリのDockerfileを使うので、指定しなくてもOK（やろうと思えばできる）

### DockerfileのRUNとCMD
- RUNはイメージを作成する段階で実行するコマンド
  - docker buildに影響する
- CMDはコンテナ起動時に実行するデフォルトのコマンドを指す
  - docker runに影響する。

### まとめ
|コマンド|引数|結果|
|:-------|:-------:|:---------|
|docker run|イメージ|コンテナを起動|
|docker ps|実行中のコンテナを確認|
|docker exec|コンテナ|コンテナでコマンドを実行|
|docker pull|イメージ|DockerHubから取得|
|docker push|イメージ|DockerHubに登録|
|docker build|Dockerfile（デフォルトで使用される）|イメージを作成|

|Dockerfileの命令|実行タイミング|影響コマンド|
|:-------|:-------:|:---------|
|RUN|イメージ作成|docker build|
|CMD|コンテナ起動|docker run|

## プロジェクトで使う
### Git管理する
- DockerfileはGit管理し、メンバーと共有する。
- 記事には書いていないけど、docker-compose-ymlなども共有するはず。（記事はPHP向け）

## WEBサーバーとDBサーバーを構築する

## Docker Composeをつかう
- docker-compose.ymlに従って、dockerのイメージのビルドや起動を一元管理してくれるツール。
