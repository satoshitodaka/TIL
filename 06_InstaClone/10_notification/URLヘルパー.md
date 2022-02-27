# URLヘルパーをビューとコントローラ以外で使うためには
- RailsにはURLヘルパーが用意されているが、標準ではビューとコントローラ、Helperの中でしか使うことができない。
- モデルや自作のモジュールで使用する場合は一手間必要。

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
