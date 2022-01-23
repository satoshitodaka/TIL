# 概要
- ImageMagick（イメージマジック）は画像を操作したり表示したりするためのソフトウェア
- 様々な画像フォーマットに対応しており、プログラム上での画像編集などの操作を行うことができる。

# 個人的にわかりづらかったところ
- imagemagick を使うためには、何らかのgemを導入する必要がある。
- 別途導入したCarrieWaveに`image_processing`が同梱されており、これに mini_magickが同梱されている。（別途rmagick などは不要。）
- 別のgemを入れても良いのだが、なぜgemを改めて導入しなくてもimagemagickが動くのかがわかりづらかった。

# 要件
- imagemagick が導入されていることを確認する。
```
$ convert --version
Version: ImageMagick 7.1.0-19 Q16-HDRI x86_64 2021-12-22 https://imagemagick.org
Copyright: (C) 1999-2021 ImageMagick Studio LLC
License: https://imagemagick.org/script/license.php
Features: Cipher DPC HDRI Modules OpenMP(4.5) 
Delegates (built-in): bzlib fontconfig freetype gslib heic jng jp2 jpeg lcms lqr ltdl lzma openexr png ps tiff webp xml zlib
Compiler: gcc (4.2)
```
- 未インストールならbrewを使ってインストールする。
```
brew install imagemagick
```
# 基本的な使い方
```
require "mini_magick"

image = MiniMagick::Image.open("input.jpg")
image.path #=> "/var/folders/k7/6zx6dx6x7ys3rv3srh0nyfj00000gn/T/magick20140921-75881-1yho3zc.jpg"
image.resize "100x100"
image.format "png"
image.write "output.png"
```
# インスタクローンでの使い方
- わかりづらかったので別途質問する。

# 参考
- https://github.com/carrierwaveuploader/carrierwave#:~:text=Using-,MiniMagick,-MiniMagick%20is%20similar
- https://github.com/minimagick/minimagick
