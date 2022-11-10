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