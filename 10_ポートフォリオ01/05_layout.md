# レイアウト
## Header
```html
<!-- Navbar -->
<div class="navbar bg-base-100 fixed top-0">
  <!-- left side -->
  <div class="navbar-start">
    <!-- Logo -->
    <div class="flex-1">
      <%= link_to root_path, class:"btn btn-ghost normal-case text-xl" do %>
        <%= image_tag 'brand-logo.png' %>
        <span>散歩くじ</span>
      <% end %>
    </div>
  </div>
  <!-- right side -->
  <div class="navbar-end">
    <!-- New lot button -->
    <div class="flex-none">
      <div class="flex-shrink-0">
        <%= link_to new_lot_path, class: 'btn btn-primary btn-sm' do %>
          <svg class="ml-1 mr-2 h-5 w-5" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 20 20" fill="currentColor" aria-hidden="true">
            <path fill-rule="evenodd" d="M10 5a1 1 0 011 1v3h3a1 1 0 110 2h-3v3a1 1 0 11-2 0v-3H6a1 1 0 110-2h3V6a1 1 0 011-1z" clip-rule="evenodd" />
          </svg>
          <span>くじを引く</span>
        <% end %>
      </div>
    </div>
    <!-- menu for desktop -->
    <ul class="menu menu-horizontal p-0 hidden lg:flex">
      <li tabindex="0">
        <a>
          散歩くじについて
          <svg class="fill-current" xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24"><path d="M7.41,8.58L12,13.17L16.59,8.58L18,10L12,16L6,10L7.41,8.58Z"/></svg>
        </a>
        <ul class="p-2 bg-base-100">
          <li><%= link_to '使い方', about_path %></li>
          <li><%= link_to '散歩のススメ', tips_to_enjoy_path %></li>
        </ul>
      </li>
      <!-- for login User -->
      <% if logged_in? %>
        <li tabindex="0">
          <a>
            ユーザー
            <svg class="fill-current" xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24"><path d="M7.41,8.58L12,13.17L16.59,8.58L18,10L12,16L6,10L7.41,8.58Z"/></svg>
          </a>
          <ul class="p-2 bg-base-100">
            <li><%= link_to 'マイページ', about_path %></li>
            <li><%= link_to 'ログアウト', logout_path, data: { turbo_method: :delete, turbo_confirm: 'ログアウトしますか？' } %></li>
          </ul>
        </li>
      <!-- for unless login User -->
      <% else %>
        <li><%= link_to 'ユーザー登録', signup_path %></li>
        <li><%= link_to 'ログイン', login_path %></li>
      <% end %>
    </ul>
    <!-- Notification Icon -->
    <% if logged_in? %>
      <%= link_to '#', class:"btn btn-ghost btn-circle" do %>
        <div class="indicator">
          <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 17h5l-1.405-1.405A2.032 2.032 0 0118 14.158V11a6.002 6.002 0 00-4-5.659V5a2 2 0 10-4 0v.341C7.67 6.165 6 8.388 6 11v3.159c0 .538-.214 1.055-.595 1.436L4 17h5m6 0v1a3 3 0 11-6 0v-1m6 0H9" /></svg>
          <!-- 通知があればバッジを表示する -->
          <span class="badge badge-xs badge-primary indicator-item"></span>
        </div>
      <% end %>
    <% end %>
    <!-- Dropdown menu for smart phone -->
    <% if logged_in? %>
      <!-- for login User -->
      <div class="dropdown dropdown-end">
        <label tabindex="0" class="btn btn-ghost btn-circle avatar lg:hidden">
          <div class="w-10 rounded-full">
            <img src="https://placeimg.com/80/80/people" />
          </div>
        </label>
        <ul tabindex="0" class="menu menu-compact dropdown-content mt-3 p-2 shadow bg-base-100 rounded-box w-52">
          <li><%= link_to 'マイページ', '#' %></li>
          <li><%= link_to '使い方', about_path %></li>
          <li><%= link_to '散歩のススメ', tips_to_enjoy_path %></li>
          <li><%= link_to 'ログアウト', logout_path, data: { turbo_method: :delete, turbo_confirm: 'ログアウトしますか？' } %></li>
        </ul>
      </div>
    <% else %>
      <!-- for unless login user -->
      <div class="dropdown dropdown-end">
        <label tabindex="0" class="btn btn-ghost lg:hidden">
          <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 6h16M4 12h8m-8 6h16" /></svg>
        </label>
        <ul tabindex="0" class="menu menu-compact dropdown-content mt-3 p-2 shadow bg-base-100 rounded-box w-52">
          <li><%= link_to 'ユーザー登録', signup_path %></li>
          <li><%= link_to 'ログイン', login_path %></li>
          <li><%= link_to '使い方', about_path %></li>
          <li><%= link_to '散歩のススメ', tips_to_enjoy_path %></li>
        </ul>
      </div>
    <% end %>
    <!-- Dropdown -->
  </div>
</div>
<!-- Navbar -->
```
### Headerを上部に固定する
- 固定する要素に`fixed`を使うと、指定した要素の表示を固定、`top-0`で固定する場所を指定する。
- [参考記事](https://qiita.com/gugen_sakai/items/f65494ef2b2d29cb6285)も読んだが最終的には実装を変えた。
> [Position](https://tailwindcss.com/docs/position)
> [Top / Right / Bottom / Left](https://tailwindcss.com/docs/top-right-bottom-left)

### Headerとコンテンツの内容が重複する問題

## Footer
- 画面サイズに応じて表示を変えるようにした
  - 通常のFooterはhiddenとし、lgサイズでgridで表示させる。
  - BottomNavigationはlgサイズでのみ非表示とする。
```html
<!-- footer for Desktop-->
<footer class="footer items-center p-4 bg-neutral text-neutral-content hidden lg:grid">
  <!-- footer logo-->
  <div class="items-center grid-flow-col">
    <%= image_tag 'logo.png' %>
    <p class="text-xl font-bold">散歩くじ</p>
    <p>Copyright © 2022 - All right reserved</p>
  </div>
  <!-- footer content list-->
  <div class="grid-flow-col gap-4 md:place-self-center md:justify-self-end">
    <%= link_to '散歩のススメ', tips_to_enjoy_path, class: 'link link-hover' %>
    <%= link_to '使い方', about_path, class: 'link link-hover' %>
    <%= link_to '利用規約', rules_path, class: 'link link-hover' %>
    <%= link_to 'プライバシーポリシー', privacy_path, class: 'link link-hover' %>
  </div>
</footer>
<!-- Bottom navigation for smart phone -->
<div class="lg:hidden">
  <div class="btm-nav">
    <%= link_to root_path type = 'button' do %>
      <svg xmlns="http://www.w3.org/2000/svg" class="icon icon-tabler icon-tabler-home-2" width="24" height="24" viewBox="0 0 24 24" stroke-width="2" stroke="currentColor" fill="none" stroke-linecap="round" stroke-linejoin="round">
      <path stroke="none" d="M0 0h24v24H0z" fill="none"></path>
      <polyline points="5 12 3 12 12 3 21 12 19 12"></polyline>
      <path d="M5 12v7a2 2 0 0 0 2 2h10a2 2 0 0 0 2 -2v-7"></path>
      <rect x="10" y="12" width="4" height="4"></rect>
    </svg>
    <% end %>
    <%= link_to new_lot_path type = 'button' do %>
      <svg xmlns="http://www.w3.org/2000/svg" class="icon icon-tabler icon-tabler-shoe" width="24" height="24" viewBox="0 0 24 24" stroke-width="2" stroke="currentColor" fill="none" stroke-linecap="round" stroke-linejoin="round">
        <path stroke="none" d="M0 0h24v24H0z" fill="none"></path>
        <path d="M4 6h5.426a1 1 0 0 1 .863 .496l1.064 1.823a3 3 0 0 0 1.896 1.407l4.677 1.114a4 4 0 0 1 3.074 3.89v2.27a1 1 0 0 1 -1 1h-16a1 1 0 0 1 -1 -1v-10a1 1 0 0 1 1 -1z"></path>
        <path d="M14 13l1 -2"></path>
        <path d="M8 18v-1a4 4 0 0 0 -4 -4h-1"></path>
        <path d="M10 12l1.5 -3"></path>
      </svg>
    <% end %>
    <%= link_to about_path type = 'button' do %>
      <svg xmlns="http://www.w3.org/2000/svg" class="icon icon-tabler icon-tabler-info-circle" width="24" height="24" viewBox="0 0 24 24" stroke-width="2" stroke="currentColor" fill="none" stroke-linecap="round" stroke-linejoin="round">
        <path stroke="none" d="M0 0h24v24H0z" fill="none"></path>
        <circle cx="12" cy="12" r="9"></circle>
        <line x1="12" y1="8" x2="12.01" y2="8"></line>
        <polyline points="11 12 12 12 12 16 13 16"></polyline>
      </svg>
    <% end %>
    <!-- link for mypage -->
    <% if logged_in? %>
      <%= link_to about_path type = 'button' do %>
      <svg xmlns="http://www.w3.org/2000/svg" class="icon icon-tabler icon-tabler-user-circle" width="24" height="24" viewBox="0 0 24 24" stroke-width="2" stroke="currentColor" fill="none" stroke-linecap="round" stroke-linejoin="round">
        <path stroke="none" d="M0 0h24v24H0z" fill="none"></path>
        <circle cx="12" cy="12" r="9"></circle>
        <circle cx="12" cy="10" r="3"></circle>
        <path d="M6.168 18.849a4 4 0 0 1 3.832 -2.849h4a4 4 0 0 1 3.834 2.855"></path>
      </svg>
      <% end %>
    <% end %>
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