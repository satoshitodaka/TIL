# EXcheckerのコントローラとエンドポイント
## 一覧
|やりたいこと|HTTPメソッド|エンドポイント|コントローラ#アクション|
|:-------|:-------:|:---------|:---------------|
|ログイン画面を表示する|GET|/login|user_sessions#new|
|ログインする|POST|/login|user_sessions#create|
|ログアウトする|DELETE|/logout|user_sessions#destroy|
|ユーザーアカウントを表示|GET|admin/users/:id|admin/users#show|
|ユーザーの新規作成画面を表示|GET|admin/users/new|admin/users#new|
|ユーザーを新規作成する|POST|admin/users|admin/users#create|
|ユーザーアカウントを編集|GET|admin/users/:id/edit|admin/users#edit|
|ユーザーアカウントを更新|PATCH|admin/users/:id|admin/users#update|
|ユーザーアカウントを削除|DELETE|admin/users/:id|admin/users#destory|
|ユーザー情報を取り込む|GET|admin/users/import|admin/users#import|
|勤休情報の一覧を表示する|GET|attendances|attendances#index|
|勤休情報を表示する|GET|attendances/:id|attendances#show|
|勤休情報を作成画面を表示する|GET|attendances/new|attendances#new|
|勤休情報を作成する|POST|attendances|attendances#create|
|勤休情報の編集画面を表示する|GET|attendances/:id/edit|attendances#edit|
|勤休情報を更新する|PATCH|attendances/:id/|attendances#update|
|勤休情報を削除する|DELETE|attendances/:id|attendances#destroy|
|勤休情報のチェック|POST|attendances/:id/check|attendances#check|
|勤休情報を対応済にする|POST|attendances/:id|attendances#check|
|一日単位勤休を取り込む|GET|day_attendance/import|day_attendances#import|
|一日単位勤休の作成画面を表示する|GET|day_attendances/new|day_attendances#new|
|一日単位勤休を作成する|POST|day_attendances|day_attendances#create|
|一日未満勤休を取り込む|GET|hour_attendances|hour_attendances#import|
|一日未満勤休の作成画面を表示する|GET|hour_attendances/new|hour_attendances#new|
|一日未満勤休を作成する|POST|hour_attendances|hour_attendances#create|
|一日未満勤休の編集画面を表示する|GET|hour_attendances/:id/edit|hour_attendances#edit|
|一日未満勤休を更新する|POST|hour_attendances|hour_attendances#update|
|通知の一覧を取得する|GET|notifications|notifications#index|
|通知の詳細を表示する|GET|notifications/:id|notifications#show|
|通知の編集画面を表示する|GET|notifications/:id/edit|notifications#edit|
|通知を更新する|PATCH|notifications/:id|notifications#update|
|通知を削除する|DELETE|notifications/:id|notifications#destory|
|カレンダの一覧を表示する|GET|department_calenders|department_calenders#index|
|カレンダの詳細を表示する|GET|department_calenders/:id|department_calenders#show|
|カレンダの作成画面を表示する|GET|department_calenders/new|department_calenders#new|
|カレンダを作成する|POST|department_calenders|department_calenders#create|
|カレンダの編集画面を表示する|GET|department_calenderss/:id/edit|department_calenders#edit|
|カレンダを更新する|PATCH|department_calenders/new|department_calenders#update|
|カレンダを削除する|DELETE|department_calenders|department_calenders#destroy|

## 補足
### Users
### Attendances
### Day_attendances
### Hour_attendances
### Notifications
### Department_calenders
### Department_schedules
