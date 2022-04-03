# ngrok
### ngrokとは
「エングロック」といい、ローカルサーバーで動作しているアプリをトンネリングを用いて全世界に公開できるようにする。
利用にあたり、公式サイトで事前にサインアップを済ませておく

[ngrok](https://ngrok.com/)

作成したアプリをデプロイせずに公開できるので、メタタグの確認をしたいときにとても便利

### インストール
zipファイルのダウンロードがうまくいかなかったのでbrewでインストールする。
```
brew install ngrok
```

### 使ってみる
ターミナルで　`ngrok http 3000`を実行する。
```
satoshitodaka@SatoshinoMacBook-Pro: ~ 
$ ngrok http 3000

ngrok by @inconshreveable                                                                                                                                       (Ctrl+C to quit)
                                                                                                                                                                                
Session Status                online                                                                                                                                            
Account                       todaka (Plan: Free)                                                                                                                               
Version                       2.3.40                                                                                                                                            
Region                        United States (us)                                                                                                                                
Web Interface                 http://127.0.0.1:4040                                                                                                                             
Forwarding                    http://59ad-133-106-56-134.ngrok.io -> http://localhost:3000                                                                                      
Forwarding                    https://59ad-133-106-56-134.ngrok.io -> http://localhost:3000                                                                                     
                                                                                                                                                                                
Connections                   ttl     opn     rt1     rt5     p50     p90                                                                                                       
                              55      0       0.00    0.01    1.80    31.04              
```
`https://59ad-133-106-56-134.ngrok.io`にアクセスすると、インスタクローンが動作していることを確認できる。

URLをslackに貼ってみる。

- トップページ
[![Image from Gyazo](https://i.gyazo.com/52e8164d44d6df4bc5185ad8cbf6ee02.png)](https://gyazo.com/52e8164d44d6df4bc5185ad8cbf6ee02)

- 投稿詳細ページ
[![Image from Gyazo](https://i.gyazo.com/e34c9d8782c1e0a0cf8f132586f6dc49.png)](https://gyazo.com/e34c9d8782c1e0a0cf8f132586f6dc49)

### 参考
[ngrokを使用してローカル環境を外部に公開する - Qiita](https://qiita.com/kitaro0729/items/44214f9f81d3ebda58bd)

[ngrokインストール方法と簡単な使い方 - Qiita](https://www.mgo-tec.com/blog-entry-ngrok-install.html)

[ngrokが便利すぎる - Qiita](https://qiita.com/mininobu/items/b45dbc70faedf30f484e)

[nginx vs ngrok: What are the differences?](https://stackshare.io/stackups/nginx-vs-ngrok)
