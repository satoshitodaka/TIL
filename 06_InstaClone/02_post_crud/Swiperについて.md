# 概要
- 画像のスライド表示を簡単に導入できるツール
# 導入
### 導入の種類
- ファイルのダウンロード、CDNでの適用、npm（またはyarn）での導入がある。
### yarnでの導入
- コンソールで実行
```
yarn add swiper
```
- package.json を確認すると、無事にインストールされたことが確認できる。
```
{
  "name": "insta_clone",
  "private": true,
  "dependencies": {
    "bootstrap-material-design": "4.1.1",
    "swiper": "^7.4.1"
  },
}
```
- app/assets/javascript.application.rb に追記
```
//= require swiper/swiper-bundle.js
//= require swiper.js
```
- app/assets/stylesheet/application.rb に追記
```
@import 'swiper/swiper-bundle';
```
- 画像のスライドを導入したい箇所のHTMLを追記する。
```
.swiper
      .swiper-wrapper 
        - post.images.each do |image|
          .swiper-slide 
            = image_tag image.url
```
# その他
- バージョンが５から６に変わった際に、導入するパッケージのディレクトリ構造が変わった。
- バージョンが６から７に変わった際に、 Swiperのクラス名が`swiper-container` から `swiper`変更となった。
```
/ バージョン６
.swiper-container
  .swiper-wrapper 
    - post.images.each do |image|
      .swiper-slide 
        = image_tag image.url

/ バージョン７
.swiper
  .swiper-wrapper 
    - post.images.each do |image|
      .swiper-slide 
        = image_tag image.url
```
# 参考
- https://qiita.com/ken_ta_/items/bdf04d8ecab6a855e50f
- https://qiita.com/miketa_webprgr/items/0a3845aeb5da2ed75f82
- https://swiperjs.com/get-started
- https://hackmd.io/QRkgKj51RyWVz5O2uBaYdQ
