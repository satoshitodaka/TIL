# Static Pages
## プライバシーポリシーや利用規約など
- 基本的にはテンプレを転用させてもらった
> [Webサイトの利用規約](https://kiyaku.jp/)

## About
- 上部に使い方の順番を記述し、リンクにアンカーを貼るようにした。
- アンカーはパスの引数として渡す。
```rb
<h2>使い方</h2>
<div>
  <ul class="steps steps-vertical">
    <li class="step step-primary"><%= link_to '行きたい場所のタイプを選ぶ', about_path(anchor: "step1") %></li>
    <li class="step step-primary"><%= link_to '出発地点を選んでくじを引く', about_path(anchor: "step2") %></li>
    <li class="step step-primary"><%= link_to 'くじに書かれた場所まで散歩する', about_path(anchor: "step3") %></li>
    <li class="step step-primary"><%= link_to 'アクティビティにチャレンジする', about_path(anchor: "step4") %></li>
    <li class="step step-primary"><%= link_to 'ご家族や友人にシェアしよう', about_path(anchor: "step5") %></li>
  </ul>
</div>
```
> [Steps](https://daisyui.com/components/steps/)
> [リンクを生成](https://railsdoc.com/page/link_to)
> [Rails link_toでアンカーを設定する - Qiita](https://qiita.com/tatsuya1156/items/595fe0df912c6c89f991)
