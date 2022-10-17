# login_logout
## 不要なファイルが生成されないように設定
```rb
module WalkingLot
  class Application < Rails::Application
    config.generators do |g|
      g.helper false
      g.test_framework :respec,
        controller_specs: false,
        view_specs: false,
        helper_specs: false,
        routing_specs: false
    end

  end
end

```
## annotate
## タイムゾーン
```rb
module WalkingLot
  class Application < Rails::Application

    config.time_zone = 'Tokyo'
    config.active_record.default_timezone = :local
  end
end

```
## sorceryの導入とUserモデル
## 国際化