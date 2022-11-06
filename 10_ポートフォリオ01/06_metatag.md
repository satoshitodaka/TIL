# Metatag
## メタタグ

## OGP画像
- OGP画像とは、SNSにシェアする際に表示する画像のこと。
- 散歩くじにおいては、サービスで写真を登録しないこと、Googleから取得する画像は著作権の問題がありそうなので、予め作成した画像を使用する。
- サイズについてはひとまず1200x630が汎用的らしいので、今回はこのサイズで作成する。
> [【2022年】全SNSにオールマイティーなOGP画像サイズ](https://bluetraff.com/ogp_thumbnail/)
- 作成はcanvasでおこなった。
- 開発環境ではOGP画像の確認ができないので、Localhost OGP チェッカーを使用した（本当にできない？）
> [localhostの状態でOGPのテストを開発環境で行う - Qiita](https://qiita.com/TeruhisaFukumoto/items/6032efde115a17b45637)

## ファビコン
- ブラウザのタイトル横に表示されるアイコンのこと。
- 画像は前出のCanvasで作成した。
- 背景色の切り抜きは以下サイトで行った。
> [Peko Step](https://www.peko-step.com/tool/alphachannel.html)
- ファビコンとして使用するためには、png形式からico形式に変更する必要があるので、以下サイトで変換した。
> [Favicon Generator. For real.](https://realfavicongenerator.net/)

## リンク類
- いずれのリンクも、新しいタブorウィンドウを開くようにしたいので、`target: '_blank'`を設定した。
- ただこの場合は遷移先サイトからもとのサイトを操作する脆弱性があるらしいので、`rel: 'noopener noreferrer'`を設定している。
> []()

### GoogleMapへのリンク
- 基本的にはドキュメント通りの実装でOKで、本家のMapでは出発地点から目的地までの徒歩のルートを表示するようにした。
```rb
  <div class="flex-shrink-0">
    <%= link_to "https://www.google.com/maps/dir/?api=1&origin=#{@lot.start_point_latitude},#{@lot.start_point_longitude}&destination=#{@lot.destination_latitude},#{@lot.destination_longitude}&travelmode=walking", class: 'btn btn-primary btn-sm', target: '_blank', rel: 'noopener noreferrer' do %>
      <svg class="ml-1 mr-2 h-5 w-5" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 20 20" fill="currentColor" aria-hidden="true">
        <path fill-rule="evenodd" d="M10 5a1 1 0 011 1v3h3a1 1 0 110 2h-3v3a1 1 0 11-2 0v-3H6a1 1 0 110-2h3V6a1 1 0 011-1z" clip-rule="evenodd" />
      </svg>
      <span>GoogleMapでひらく</span>
    <% end %>
  </div>d %>
</div>
```
> [Maps URLs](https://developers.google.com/maps/documentation/urls/get-started?hl=ja)
> [GoogleMapへのリンクURL作成方法を調べた - Qiita](https://qiita.com/hiron2225/items/8d5cd1b6728b4d63434b)

### Twitterシェアボタン
- ツイートに`目的地で「アクティビティ」します！`というテキストを入れようと思ったが、全角の「」はURLとして含めることができなかったので、エンコードした文字列を入れるようにした。
- ツイート内で改行する部分は、改行をエンコードした文字列を差し込むことにした。
```rb
<div class="flex-shrink-0">
  <%= link_to "https://twitter.com/intent/tweet?text=#{@lot.destination_name}で%E3%80%8C#{@lot.activity.content}%E3%80%8Dにチャレンジします!!%0D%0A&hashtags=散歩くじ,散歩,ウォーキング,散歩好きな人と繋がりたい&url=https://guides.rubyonrails.org/%0D%0A", class: 'btn btn-primary btn-sm', target: '_blank', rel: 'noopener noreferrer' do %>
    <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" class="fill-current">
      <path d="M24 4.557c-.883.392-1.832.656-2.828.775 1.017-.609 1.798-1.574 2.165-2.724-.951.564-2.005.974-3.127 1.195-.897-.957-2.178-1.555-3.594-1.555-3.179 0-5.515 2.966-4.797 6.045-4.091-.205-7.719-2.165-10.148-5.144-1.29 2.213-.669 5.108 1.523 6.574-.806-.026-1.566-.247-2.229-.616-.054 2.281 1.581 4.415 3.949 4.89-.693.188-1.452.232-2.224.084.626 1.956 2.444 3.379 4.6 3.419-2.07 1.623-4.678 2.348-7.29 2.04 2.179 1.397 4.768 2.212 7.548 2.212 9.142 0 14.307-7.721 13.995-14.646.962-.695 1.797-1.562 2.457-2.549z"></path>
    </svg>
    <span>Twitterでシェアする</span>
  <% end %>
</div>
```
> [Tweet button](https://developer.twitter.com/en/docs/twitter-for-websites/tweet-button/overview)
> [リプライ、ハッシュタグ、URLなどを指定したツイートリンクの貼り方](https://hyakuyattsu.com/create/tweet-link)
> []()

### LINE送信ボタン
- 公式アイコンにURLを埋め込む場合、URLを動的に変更できなかったので、カスタムボタンに動的なURLをを埋め込む形にした。
```rb
<div class="flex-shrink-0">
  <%= link_to "https://social-plugins.line.me/lineit/share?url=https://walking-lot.com/lots/#{@lot.id}", class: 'btn btn-primary btn-sm', target: '_blank', rel: 'noopener noreferrer' do %>
    <svg class="ml-1 mr-2 h-5 w-5" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 20 20" fill="currentColor" aria-hidden="true">
      <path fill-rule="evenodd" d="M10 5a1 1 0 011 1v3h3a1 1 0 110 2h-3v3a1 1 0 11-2 0v-3H6a1 1 0 110-2h3V6a1 1 0 011-1z" clip-rule="evenodd" />
    </svg>
    <span>LINEで送る</span>
  <% end %>
</div>
```
>[「LINEで送る」ボタンを設置する](https://developers.line.biz/ja/docs/line-social-plugins/install-guide/using-line-share-buttons/)