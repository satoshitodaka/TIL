# Rspec関連の設定
## Rspecを導入
- Gemfileに追記
```Gemfile
group :development, :test do
  # See https://guides.rubyonrails.org/debugging_rails_applications.html#debugging-with-the-debug-gem
  gem "debug", platforms: %i[ mri mingw x64_mingw ]
  gem 'factory_bot_rails'
  gem 'faker'
  gem 'rspec-rails'
end

group :test do
  gem 'capybara'
  gem 'selenium-webdriver'
  gem 'webdrivers'
end
```
- gemをインストール後、`rails g rspec:install`する。
## Docker環境でシステムスペックを実行する
- Headless Chromeとして、docker-selenimのDockerイメージを使う。
- システムスペックではブラウザ上の挙動を見たいので、[selenium/standalone-chrome-debug](https://hub.docker.com/r/selenium/standalone-chrome-debug)を使用する。
- docker-compose.ymlを編集する。
  - depends_onでchromeを指定することで、Docker起動時にChromeコンテナも自動で立ち上がる。
  - アプリ側からアクセスできるように、環境変数`web.environment.SELENIUM_DRIVER_URL`を設定する。
```yml
version: "3.9"
services:
  web:
    depends_on:
      - db
      - chrome # 追記
    stdin_open: true
    tty: true
    environment: # 追記
      SELENIUM_DRIVER_URL: http://chrome:4444/wd/hub
  chrome: # 追記
    image: selenium/standalone-chrome-debug:latest
    ports:
      - 4444:4444
```
- Rspec実行時に、docker-seleniumのコンテナを使用するよう設定する。
- 今回は`spec/support/capybara.rb`に設定を記述するので、`spec/rails_helper.rb`にファイルを読み込む設定も記述する。
```rb
# spec/rails_helper.rb
Dir[Rails.root.join('spec', 'support', '**', '*.rb')].sort.each { |f| require f }
```
```rb
# spec/support/capybara.rb

# 非同期処理の際、テストでその処理が完了した後に、次のリンクが表示される場合、リンクの表示に時間を要する可能性があるので、指定した時間内で再試行することができる。デフォルトは2秒
Capybara.default_max_wait_time = 5
# Seleniumドライバーを使用するために追記
Capybara.server = :puma, { Silent: true }

# Selenium::WebDriverでHeadless Chromeを起動した時のオプションを設定する
options = ::Selenium::WebDriver::Chrome::Options.new
# ヘッドレスモードで起動する
options.add_argument('--headless')
# サンドボックスモードを解除する
options.add_argument('--no-sandbox')
# ウィンドウサイズを指定する
options.add_argument('--window-size=1680,1050')
# GPUハードウェアアクセラレーションを無効にする
options.add_argument('--disable-gpu')
# 特定のVM環境では、/dev/shmパーティションが小さすぎてChromeが失敗することがあり、これを回避するためのフラグ
options.add_argument('--disable-dev-shm-usage')

# コンテナ環境かどうかで処理を分岐する
# docker-composeで指定した環境変数SELENIUM_DRIVER_URLを判定する。
if ENV['SELENIUM_DRIVER_URL']
  # Capybaraにremote_chromeというドライバーを設定する
  Capybara.register_driver :remote_chrome do |app|
    url = ENV['SELENIUM_DRIVER_URL']
    # ドライバーを作成する
    Capybara::Selenium::Driver.new(
      app,
      browser: :remote,
      url: url,
      options: options
    )
  end
  # システムのホスト名を取得する
  Capybara.server_host = IPSocket.getaddress(Socket.gethostname)
  Capybara.server_port = 3001
  # Capybaraのアクセス先のホスト名を（動的に）指定する
  Capybara.app_host = "http://#{Capybara.server_host}:#{Capybara.server_port}"
else
  # Dockerコンテナではない場合、selenium_chrome_headlessというドライバーを設定する
  Capybara.register_driver :selenium_chrome_headless do |app|
    Capybara::Selenium::Driver.new(
      app,
      browser: :chrome,
      # timeoutにCAPYBARA_TIMEOUTもしくは10を設定する
      timeout: ENV.fetch('CAPYBARA_TIMEOUT') { 10 },
      options: options
    )
  end
end

# Capybara.javascript_driverを環境に応じて指定する
Capybara.javascript_driver = ENV['SELENIUM_DRIVER_URL'] ? :remote_chrome : :selenium_chrome_headless

# Rspecで使用するドライバーを指定する
RSpec.configure do |config|
  config.before(:each, type: :system) do |_example|
    driven_by Capybara.javascript_driver
  end
end
```
### 参考
- [Capybara](https://github.com/teamcapybara/capybara/blob/master/README.md)
- [Capybara(和訳)](https://github.com/willnet/capybara-readme-ja)
- [Seleniumでよく使うChromeOptionsまとめ](https://boardtechlog.com/2020/08/programming/seleniumchrome%E3%81%A7%E3%82%88%E3%81%8F%E4%BD%BF%E3%81%86chromeoptions%E3%81%BE%E3%81%A8%E3%82%81/)

## Factory_botの設定
- `spec/rails_helper.rb`に追記する
```rb
# FactoryBotを使用するために読み込み
require 'factory_bot'
```
- 今回は`spec/rails_helper.rb`を綺麗に保つため、`spec/support/factory_bot.rb`に追記する。
```rb
RSpec.configure do |config|
  # 下記を追記
  config.include FactoryBot::Syntax::Methods
end
```