# 概要
- Font-awesome とは、無料で使えるアイコン
- font-awesome-sass とは、font-awesome をrailsで使えるようにしたgemのこと。
# 導入
- Gemfile に記載し、bundle installを実行
```
gem 'font-awesome-sass'
```
- app/assets/stylesheets/application.scss 　に以下を記載する。
```
@import "font-awesome-sprockets";
@import "font-awesome";
```
# 使い方
### 書き方
```
<i class="fas fa-flag"></i>
icon('fas', 'flag')
icon 'fas', 'flag'
```
- 接頭辞がアイコンの分類を表す。

| 分類     | 接頭辞 |
|:---:|:---:|
| solid   | fas |
| Regural | far |
| Light   | fal |
| Duotone | fad |
|Brands|fab|

- 続けてアイコン名を入力する。
- 大きさはclassで指定できる。
# 使ったアイコン
### user
- https://fontawesome.com/v5.15/icons/user
### trash-alt
- https://fontawesome.com/v5.15/icons/trash-alt

# Sizing
- https://fontawesome.com/v5.15/how-to-use/on-the-web/styling/sizing-icons

# 参考
- https://fontawesome.com/v5.15/how-to-use/on-the-web/using-with/sass
- https://pikawaka.com/rails/font_awesome_sass
