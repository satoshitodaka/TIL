# レイアウト
## Header
### Headerを上部に固定する
- 固定する要素に`fixed`を使うと、指定した要素の表示を固定、`top-0`で固定する場所を指定する。
- [参考記事](https://qiita.com/gugen_sakai/items/f65494ef2b2d29cb6285)も読んだが最終的には実装を変えた。
> [Position](https://tailwindcss.com/docs/position)
> [Top / Right / Bottom / Left](https://tailwindcss.com/docs/top-right-bottom-left)
#### レイアウト調整
- HeaderをFixedすると、フッターがレイアウト的に浮いてしまい、その分コンテンツを下げて（Marginを設ける）表示する必要がある
- 後述するBottomNavigationは画面下部に固定するため、同様にコンテンツの下部にMarginを設ける。
```html
<div class="grow mt-7 mb-12">
  <%= yield %>
</div>
```
> [ヘッダーをposition:fixedで固定時、直下の要素が重なって隠れる原因と解決法](https://tanomasaki.com/headernav-fixed/)

### Headerとコンテンツの内容が重複する問題
- 要素の重なりのルールについて
1. 原則は、コンテンツのコードの記述順に重なって表示される。
2. z-indexを指定する場合、positionにrelativeかabsoluteを指定する。


> [CSSでz-indexが効かない時の4つの原因とその対応方法](https://coliss.com/articles/build-websites/operation/css/4-reasons-z-index-isnt-working.html)

### headerのpositionをfixedする
- header自体のpositionをfixedすることで、画面上部に表示を固定することができる。

### headerのぶんだけコンテンツを下げる
- 手法はいくつかあるらしい。
  - コンテンツのpadding-topを設定する方法。暫定的にこれで対応したが、最終的には修正。
  - Javascriptでコンテンツの高さを変動する方法。最終的にこれにしたい。

- scriptタグに処理を記述するには、jQueryが必要らしい。
- 現在の環境では、yarnの導入は以下が参考になりそう。
> [Migrate from webpacker to esbuild](https://www.fastruby.io/blog/esbuild/webpacker/javascript/migrate-from-webpacker-to-esbuild.html)
- ただ、yarnで導入しても読み込みがうまくいかなかったので、暫定的にscriptタグで読み込む方法にした。
```html
<script src="https://code.jquery.com/jquery-latest.js"></script>
<script type="text/javascript">
  $(function() {
      var height=$("#header").height();
      $("body").css("margin-top", height + 10);//10pxだけ余裕をもたせる
  });
</script>
```
> [ヘッダーの高さ分だけコンテンツを下げる（paddingは使わない）方法](https://uguisu.skr.jp/html/fixed.html)
### ページ内リンクのanchorを調整する
- html要素全体に適用するため、以下の通り記述する。
- 本来であれば`app/assets/stylesheets/application.tailwind.css`に記述するが、読み込みが上手くいかなかったので暫定的に`app/views/layouts/application.html.erb`のstyleタグに記述した。
```html
<style>
  html {
    scroll-padding-top: 5rem; 
  }
</style>
```

## Footer
- 画面サイズに応じて表示を変えるようにした
  - 通常のFooterはhiddenとし、lgサイズでgridで表示させる。
  - BottomNavigationはlgサイズでのみ非表示とする。
```html
<!-- footer for Desktop-->
<footer class="footer items-center p-4 bg-neutral text-neutral-content hidden lg:grid">
  <!-- 省略 -->
</footer>
<!-- Bottom navigation for smart phone -->
<div class="lg:hidden">
  <div class="btm-nav">
    <!-- 省略 -->
  </div>
</div>
```

### footerを下部に固定する
- 要素全体を囲む部分にクラス`flex flex-col min-h-screen`を適用する
  - `flex-col`は垂直方向の伸び縮みを指定する。
  - `min-h-screen`は画面の最小の高さを100vhに指定する。（min-h-fullではうまくいかなかった）
- 伸び縮みする部分にクラス`flex-grow`を適用する(参考にしたブログ記事は少し古いかも)
```rb
<div class="flex flex-col min-h-screen">
  <div class="grow">
    <%= yield %>
  </div>
</div>
```
> [Tailwind CSSでフッターを固定する方法](https://webty.jp/staffblog/production/post-2133/)
> [Flex Grow](https://tailwindcss.com/docs/flex-grow)