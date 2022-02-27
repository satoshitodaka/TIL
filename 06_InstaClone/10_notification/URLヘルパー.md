# URLヘルパーをビューとコントローラ以外で使うためには
- RailsにはURLヘルパーが用意されているが、標準ではビューとコントローラ、Helperの中で使うことができる。
  - これらの中で`_url`といったヘルパーを使うと、hostドメインをアプリケーションが補完してくれる。
  - それら以外（モデルやモジュール）で使う場合、hostを明示的に伝える必要がある。

### 実装
```rb
class Activity < ApplicationRecord
  # モデルでURLヘルパーを使うために導入
  include Rails.application.routes.url_helpers
  
  def hogehoge
    # ここでURLを扱う
  end
end
```

### 参考
[Rails: URLヘルパーをビューやコントローラ以外の場所で使う（翻訳）](https://techracho.bpsinc.jp/hachi8833/2021_03_05/104476)

[Railsの不特定ModuleやClass(Modelなど)で`_path`を使う](https://qiita.com/jerrywdlee/items/f91c9ea01055cb74083c)
