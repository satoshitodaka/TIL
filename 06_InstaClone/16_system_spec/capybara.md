# capybara
### capybaraとは
- WebアプリケーションのE2Eテスト用のフレームワークで、MinitestやRspecと組み合わせて使う。
- アプリのブラウザ操作(UI)をシュミレーションすることができる。
- ドライバーの選択肢が複数あり、任意のドライバーを使用することができる。
  - WebDriverを使用する。

### 導入
Gemfileに記述してインストールする。
```
group :test do
  gem 'capybara'
  # Capybaraが利用するドライバを任意に追加する
  gem 'webdrivers'
end
```

spec/rails_helper.rbに記述する。
```rb
# 下記を追記（公式にはspec/spec_helper.rbとある）
require 'capybara/rspec'

# 省略

config.include FactoryBot::Syntax::Methods
# 下記を追記
config.include SystemHelper, type: :system
```

spec/spec_helper.rbに記述する。
```rb
RSpec.configure do |config|
  config.before(:each, type: :system) do
    driven_by :selenium, using: :headless_chrome, screen_size: [1920, 1080]
  end
end
```

### 参考
[Using Capybara with RSpec - Capybara](https://github.com/teamcapybara/capybara#:~:text=selenium%2C%20%40rack_test%2C%20etc).-,Using%20Capybara%20with%20RSpec,-Load%20RSpec%203.5)

[Capybaraチートシート - Qiita](https://qiita.com/morrr/items/0e24251c049180218db4)

[Webdrivers](https://github.com/titusfortner/webdrivers)※あまり読めてない


