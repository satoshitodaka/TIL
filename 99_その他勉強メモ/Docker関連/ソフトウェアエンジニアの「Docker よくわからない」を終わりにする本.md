# ソフトウェアエンジニアの「Docker よくわからない」を終わりにする本
## 1部
### Dockerとは
- コンテナ型仮想化を用いてアプリの開発などを行うプラットフォーム
- Dockerという単語はプラットフォームに含まれるサービス全体を指すことが多いよう。
  - Docker Engine
  - Docker CLI
  - Docker Desktop
  - Docker Compose
  - Docker Hub

### Dockerの用語について
#### Docker Engine
- コンテナ型仮想化のソフトウェアで、Dockerのコアとなる部分。

#### Docker CLI
- Dockerを操作するコマンドのこと

#### Docker Desktop
- DockerのGLIアプリケーション
- 一般的に、Dockerのインストールとはこのデスクトップアプリに対して実行することを指す。

### Docker Compose
- docker-composeから始まるコマンドで、Docker CLIをまとめて実行できるもの。

#### Docker Hub
- イメージのGithubのようなもの。

## 2部
### コンテナとは
- ホストマシン上の隔離された領域で、特定のコマンドを実行するために作られる。
- ホストマシン上で動く仮想OSのように見えるが、Linuxの1プロセスに過ぎない。
- コンテナには以下の特徴がある。
  - コンテナはイメージをもとに作られる。
  - DockerのCLIやAPIを使って制御する。
  - コンテナが複数あってもそれぞれ独立しており、互いに影響しない。
  - Docker Engineの上なら、ローカルマシンでも仮想マシンでもクラウド環境でも動かせる。

### イメージとは
- コンテナの実行に必要なパッケージで、ファイルやメタ情報を集めたもの。
- 特定のファイルを指すものではなく、複数のレイヤーからなる情報のこと
- イメージは以下を含む
  - ベースは何か
  - 何をインストールしているか
  - 環境変数はどうなっているか
  - どういう設定ファイルを配置しているか
  - デフォルト命令は何か

### Dockerfileとは
- 既存のイメージにレイヤーを積み重ねるためのテキストファイル。
- インターネットで公開されているイメージでは足りない部分を、自分で書き足すもの。
- Dockerfileは GitHubなどで共有するのが望ましい。

### 基本のコマンド
- コマンドの基本形を3つに分類して考えることができる
   - コンテナの起動
   - イメージのビルド
   - そのほかコンテナの操作

### コンテナの起動
- container runというコマンドは、イメージからコンテナを起動する。

### イメージを作成する
- image buildというコマンドは、Dockerfileからイメージを作成する。

### コンテナを操作する
- container execなど、コンテナに対して命令を送るコマンド
- コンテナに対して命令するので、イメージやDockerfileに対しては命令できない。

### 新旧のコマンドについて
- v.1.13よりコマンドが新しくなり、タイプ数は増えたもののわかりやすくなった。
|旧コマンド|新コマンド|
|:-------|:-------|
|docker build|docker image build|
|docker run|docker container run|
|docker pull|docker image pull|
|docker create|docker container create|
|docker start|docker container start|
|docker images|docker image ls|
|docker ps|docker container ls|

### Docker Compose
- yamlファイルを書くことにより、複数コンテナをまとめて起動してくれたりする。
- Docker CLIのみで環境構築すると、複雑なコマンドを何度も実行する必要がある。それらを一つのファイル(docker-compose.yml)にまとめ、Githubなどで共有できるようにしている。

### Dockerのコマンド
```
<!-- []で囲まれたオプションは任意、<>で囲まれた箇所は必須 -->
<!-- コンテナを起動する。 -->
docker container run [option] <image> [command]

<!-- コンテナ一覧を確認する -->
docker container ls [option]

<!-- コンテナを停止する -->
docker container stop [option] <container>

<!-- コンテナを削除する -->
docker container rm [option] <container>
```

### 停止するか削除するか
- コンテナは軽量で使い捨てのイメージなので、用が済んだら削除する、で良さそう。
- 使う必要が出た時に再度作成する。

