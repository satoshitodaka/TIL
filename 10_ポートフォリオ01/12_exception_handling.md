# exception_handling
## 実装の方針
- 以下のケースを対応する
  - 無効なページにアクセスした場合
  - APIのレスポンス取得に失敗した場合

## 404/500エラーの処理
- `app/controllers/application_controller.rb`に以下の通り記述する。
 - `rescure_from`でExceptionを指定するのは良くないらしいが、resucue_fromは下から評価されるので、範囲が広いrescue_fromは下から書くと良い。
```rb
rescue_from Exception, with: :render_500
rescue_from ActiveRecord::RecordNotFound, with: :render_404
rescue_from ActionController::RoutingError, with: :render_404

def render_404
  render template: 'errors/error_404', status: 404, layout: 'application', content_type: 'text/html'
end

def render_500
  render template: 'errors/error_500', status: 500, layout: 'application', content_type: 'text/html'
end
```
- ルーティングを以下の通り指定する。
```rb
get '*path', to: 'application#render_404'
```
- 表示するビューファイルは`app/views/errors`直下に作成する。
> [Railsで404/500エラーの処理 - Qiita](https://qiita.com/itoytr/items/fb1ea251197003deec12)
> [Rails の rescue_from で Exception や StandardError をキャッチすることは推奨されていない](https://please-sleep.cou929.nu/handle-exception-by-rescue_from-is-not-recommended.html)

### 開発環境でエラーページを確認する
- 404ページを開くためには、`config/environments/development.rb`を以下の通りfalseに修正する
```rb
# Show full error reports.
config.consider_all_requests_local = false
```
- 500エラーを表示させるには、対象のコントローラのアクションで`raise`する。
```rb
def index
  raise
end
```
> [404/500エラーページを開発環境で表示したい - Zenn](https://zenn.dev/matsu18/articles/4bd608c2eadf32)

## NearbySearchの結果により挙動を変える
- 処理の順番は以下の通り
1. 入力内容のバリデーション。修正前は`@lot.save`で実行していた。
2. NearbySearchで検索し、取得件数で処理を変える。
3. 保存と処理をおこなう。

lot = Lot.build(start_point_latitude: 130, start_point_longitude: 135, location_type_id: 2)
