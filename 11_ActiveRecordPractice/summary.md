# ActiveRecordPractice
https://tech-essentials.work/courses/50/tasks/28

## 課題
### 従業員番号が499999の従業員の給料全てを取得してください
- そのモデルより、カラムの値を指定して取得する場合、`where`メソッドでカラムとその値を指定する。
```
irb(main):014:0> Salary.where(emp_no: 499999)
  Salary Load (6.3ms)  SELECT `salaries`.* FROM `salaries` WHERE `salaries`.`emp_no` = 499999
+--------+--------+------------+------------+
| emp_no | salary | from_date  | to_date    |
+--------+--------+------------+------------+
| 499999 | 63707  | 1997-11-30 | 1998-11-30 |
| 499999 | 67043  | 1998-11-30 | 1999-11-30 |
| 499999 | 70745  | 1999-11-30 | 2000-11-29 |
| 499999 | 74327  | 2000-11-29 | 2001-11-29 |
| 499999 | 77303  | 2001-11-29 | 9999-01-01 |
+--------+--------+------------+------------+
5 rows in set
```
### 従業員番号が499999の従業員の2001-01-01時点の給料を取得してください
-　whereメソッドでAND検索をする場合、whereメソッドの引数をカンマで区切って複数渡す。
```
irb(main):171:0> Salary.where(emp_no: '499999', to_date: Time.local(2001, 1,1)...Float::INFINITY).where.not(from_date: Time.local(2001, 1,1)...Float::INFINITY)
  Salary Load (5.8ms)  SELECT `salaries`.* FROM `salaries` WHERE `salaries`.`emp_no` = 499999 AND `salaries`.`to_date` >= '2001-01-01' AND NOT (`salaries`.`from_date` >= '2001-01-01')
+--------+--------+------------+------------+
| emp_no | salary | from_date  | to_date    |
+--------+--------+------------+------------+
| 499999 | 74327  | 2000-11-29 | 2001-11-29 |
+--------+--------+------------+------------+
1 row in set
```
### 150000以上の給料をもらったことがある従業員の一覧を取得してください
- 関連するテーブルの条件で検索する場合、joinsメソッドで関連テーブルをつなげ、whereメソッドの引数にハッシュを渡す。結合したテーブルの条件で絞る場合、where句の中で`where(salaries: { salary: 150000... } )`といった具合でネストする
- 「〜以上」という条件を指定する場合、範囲オブジェクトを渡すことで指定できる。
- joinsは内部結合で、従業員が重複することになるので、distinctで重複を削除する。
```
irb(main):048:0> Employee.joins(:salaries).where(salaries: { salary: 150000... }  ).distinct
  Employee Load (1246.0ms)  SELECT DISTINCT `employees`.* FROM `employees` INNER JOIN `salaries` ON `salaries`.`emp_no` = `employees`.`emp_no` WHERE `salaries`.`salary` >= 150000
+--------+------------+------------+-----------+--------+------------+
| emp_no | birth_date | first_name | last_name | gender | hire_date  |
+--------+------------+------------+-----------+--------+------------+
| 43624  | 1953-11-14 | Tokuyasu   | Pesch     | M      | 1985-03-26 |
| 46439  | 1953-01-31 | Ibibia     | Junet     | M      | 1985-05-20 |
| 47978  | 1956-03-24 | Xiahua     | Whitcomb  | M      | 1985-07-18 |
| 66793  | 1964-05-15 | Lansing    | Kambil    | M      | 1985-06-20 |
| 80823  | 1963-01-21 | Willard    | Baca      | M      | 1985-02-26 |
| 109334 | 1955-08-02 | Tsutomu    | Alameldin | M      | 1985-02-15 |
| 205000 | 1956-01-14 | Charmane   | Griswold  | M      | 1990-06-23 |
| 237542 | 1954-10-05 | Weicheng   | Hatcliff  | F      | 1985-04-12 |
| 238117 | 1959-06-21 | Mitsuyuki  | Stanfel   | M      | 1988-01-03 |
| 253939 | 1957-12-03 | Sanjai     | Luders    | M      | 1987-04-15 |
| 254466 | 1963-05-27 | Honesty    | Mukaidono | M      | 1986-08-08 |
| 266526 | 1957-02-14 | Weijing    | Chenoweth | F      | 1986-10-08 |
| 276633 | 1954-01-27 | Shin       | Birdsall  | M      | 1987-10-08 |
| 279776 | 1955-05-06 | Mohammed   | Moehrke   | M      | 1986-06-10 |
| 493158 | 1961-05-20 | Lidong     | Meriste   | M      | 1987-05-09 |
+--------+------------+------------+-----------+--------+------------+
15 rows in set
```
### 150000以上の給料をもらったことがある女性従業員の一覧を取得してください
```
irb(main):050:0> Employee.joins(:salaries).where(gender: 'F', salaries: { salary: 150000... }  ).distinct
  Employee Load (587.0ms)  SELECT DISTINCT `employees`.* FROM `employees` INNER JOIN `salaries` ON `salaries`.`emp_no` = `employees`.`emp_no` WHERE `employees`.`gender` = 'F' AND `salaries`.`salary` >= 150000
+--------+------------+------------+-----------+--------+------------+
| emp_no | birth_date | first_name | last_name | gender | hire_date  |
+--------+------------+------------+-----------+--------+------------+
| 237542 | 1954-10-05 | Weicheng   | Hatcliff  | F      | 1985-04-12 |
| 266526 | 1957-02-14 | Weijing    | Chenoweth | F      | 1986-10-08 |
+--------+------------+------------+-----------+--------+------------+
2 rows in set
irb(main):051:0>  
```
### どんな肩書きがあるか一覧で取得してきてください
- selectを使うことで取得するカラムを指定できる。
```
irb(main):052:0> Title.select(:title).distinct
  Title Load (211.2ms)  SELECT DISTINCT `titles`.`title` FROM `titles`
+--------------------+
| title              |
+--------------------+
| Senior Engineer    |
| Staff              |
| Engineer           |
| Senior Staff       |
| Assistant Engineer |
| Technique Leader   |
| Manager            |
+--------------------+
7 rows in set
```
### 2000-1-29以降に肩書きが「Technique Leader」になった従業員を取得してください
- 日付を条件にする場合、日付の範囲を指定する。
- '2000-1-29'はそのままでは日付として使えないので、`to_date`メソッドでDateに変更して使う。
- 範囲演算子は、`..`は通常の範囲で、`...`は終端を含まない範囲（ =「〜未満」という意味となる）を作成する。
```
irb(main):099:0> Employee.joins(:titles).where( titles: {title: 'Technique Leader', from_date: '2000-1-29'.to_date... }).distinct
  Employee Load (134.5ms)  SELECT DISTINCT `employees`.* FROM `employees` INNER JOIN `titles` ON `titles`.`emp_no` = `employees`.`emp_no` WHERE `titles`.`title` = 'Technique Leader' AND `titles`.`from_date` >= '2000-01-29'
+--------+------------+------------+-----------+--------+------------+
| emp_no | birth_date | first_name | last_name | gender | hire_date  |
+--------+------------+------------+-----------+--------+------------+
| 79962  | 1953-05-29 | Maik       | Heping    | M      | 1995-08-09 |
| 83076  | 1960-09-07 | Jacopo     | Thiria    | M      | 1999-07-20 |
| 88961  | 1954-03-07 | Uta        | Asser     | F      | 1998-12-02 |
| 203599 | 1965-01-21 | Ewing      | DiGiano   | M      | 1985-04-27 |
| 253428 | 1954-04-06 | Ortrun     | Benner    | F      | 1995-01-31 |
| 262244 | 1963-05-08 | Gal        | Ramaiah   | F      | 1989-08-01 |
| 414550 | 1960-02-01 | Sandeepan  | Krogh     | F      | 1992-01-04 |
+--------+------------+------------+-----------+--------+------------+
7 rows in set
```
> [ActiveRecordで日付・時刻の範囲検索をシンプルに書く方法](https://techracho.bpsinc.jp/hachi8833/2021_08_26/24876)

### 部署番号がd001である部署のマネージャー歴代一覧を取得してきてください
- Employeeにdept_managerを内部結合する。dept_managerは単数形のモデルだが、joinsで結合する際は関連を指定するので引数に複数形を渡す。
```
irb(main):040:0> Employee.joins(:dept_managers).where(dept_manager: {dept_no: 'd001'})
  Employee Load (6.6ms)  SELECT `employees`.* FROM `employees` INNER JOIN `dept_manager` ON `dept_manager`.`emp_no` = `employees`.`emp_no` WHERE `dept_manager`.`dept_no` = 'd001'
+--------+------------+------------+------------+--------+------------+
| emp_no | birth_date | first_name | last_name  | gender | hire_date  |
+--------+------------+------------+------------+--------+------------+
| 110022 | 1956-09-12 | Margareta  | Markovitch | M      | 1985-01-01 |
| 110039 | 1963-06-21 | Vishwani   | Minakawa   | M      | 1986-04-12 |
+--------+------------+------------+------------+--------+------------+
2 rows in set
```
### 歴代マネージャーにおける男女比を出してください
- groupメソッドをつかうことで、指定したカラムごとにレコードをまとめることができる。単体で使用するのではなく、countやsumと併用する事が多い。併用すると、返り値はカラムと集計の結果のハッシュが返される。
```
irb(main):132:0> Employee.joins(:dept_managers).group('employees.gender').count
   (9.0ms)  SELECT COUNT(*) AS count_all, employees.gender AS employees_gender FROM `employees` INNER JOIN `dept_manager` ON `dept_manager`.`emp_no` = `employees`.`emp_no` GROUP BY employees.gender
=> {"M"=>11, "F"=>13}
```
> [【Rails】 groupメソッドの使い方とは？仕組みを図解で丁寧に解説！](https://pikawaka.com/rails/group)
> [ActiveRecordにおけるGROUP BYの使い方 - Qiita](https://qiita.com/yuyasat/items/e26bcf0eb2c89c63db9d)

### 部署番号がd004の部署における1999-1-1時点のマネージャーを取得してください
- 日付の範囲の終端に、浮動小数点数の無限大を表す`Float::INFINITY`を指定することができる。これは負の値を指定することはできないため、日付の範囲の開始時点を指定することはできない。この場合は、`where.not()`を使って終端に`Float::INFINITY`を指定する。
```
irb(main):121:0> Employee.joins(:dept_managers, :titles).where(dept_manager: { dept_no: 'd004' }, titles: { title: 'Manager', to_date: Time.local(1999,1,1) ... Float::INFINITY}).where.not(titles: { from_date: Time.local(1999,1,1)..Float::INFINITY })
  Employee Load (0.8ms)  SELECT `employees`.* FROM `employees` INNER JOIN `dept_manager` ON `dept_manager`.`emp_no` = `employees`.`emp_no` INNER JOIN `titles` ON `titles`.`emp_no` = `employees`.`emp_no` WHERE `dept_manager`.`dept_no` = 'd004' AND `titles`.`title` = 'Manager' AND `titles`.`to_date` >= '1999-01-01' AND NOT (`titles`.`from_date` >= '1999-01-01')
+--------+------------+------------+-----------+--------+------------+
| emp_no | birth_date | first_name | last_name | gender | hire_date  |
+--------+------------+------------+-----------+--------+------------+
| 110420 | 1963-07-27 | Oscar      | Ghazalie  | M      | 1992-02-05 |
+--------+------------+------------+-----------+--------+------------+
1 row in set
```
> [Rails における日付範囲で無限大 Float::INFINITY の扱う方法について調べたこと - Qiita](https://qiita.com/gotchane/items/1d6069c2bd4ecd5c386b)

### 従業員番号が10001, 10002, 10003の従業員が今までに稼いだ給料の合計を従業員ごとに集計してください
- where句で指定する条件の値に、配列を渡すことでIN式で検索できる。
- groupメソッドで集計する対象のカラムを指定（従業員ごとに集計）、sumメソッドで合計するカラムを指定する。
```
irb(main):144:0> Salary.where(emp_no: [10001, 10002, 10003]).group(:emp_no).sum(:salary)
   (1.6ms)  SELECT SUM(`salaries`.`salary`) AS sum_salary, `salaries`.`emp_no` AS salaries_emp_no FROM `salaries` WHERE `salaries`.`emp_no` IN (10001, 10002, 10003) GROUP BY `salaries`.`emp_no`
=> {10001=>1281612, 10002=>413127, 10003=>301212}
```
> [3.3 条件でハッシュを使う](https://railsguides.jp/active_record_querying.html#or%E6%9D%A1%E4%BB%B6:~:text=%2B%20%22%25%22)-,3.3%20%E6%9D%A1%E4%BB%B6%E3%81%A7%E3%83%8F%E3%83%83%E3%82%B7%E3%83%A5%E3%82%92%E4%BD%BF%E3%81%86,-Active%20Record%E3%81%AF)

### 上記に加えtotal_salaryという仮のフィールドを作ってemployeeの情報とがっちゃんこしてください。
- 取得するテーブルに任意のカラムを追加する場合、selectメソッドの中でsumメソッドを使い、asの後に使用するカラム名を指定する。
```
irb(main):151:0> Employee.joins(:salaries).where(emp_no: [10001, 10002, 10003]).select('employees.*, sum(salary) as total_salary').group(:emp_no)
  Employee Load (2.2ms)  SELECT employees.*, sum(salary) as total_salary FROM `employees` INNER JOIN `salaries` ON `salaries`.`emp_no` = `employees`.`emp_no` WHERE `employees`.`emp_no` IN (10001, 10002, 10003) GROUP BY `employees`.`emp_no`
+--------+------------+------------+-----------+--------+------------+--------------+
| emp_no | birth_date | first_name | last_name | gender | hire_date  | total_salary |
+--------+------------+------------+-----------+--------+------------+--------------+
| 10001  | 1953-09-02 | Georgi     | Facello   | M      | 1986-06-26 | 1281612      |
| 10002  | 1964-06-02 | Bezalel    | Simmel    | F      | 1985-11-21 | 413127       |
| 10003  | 1959-12-03 | Parto      | Bamford   | M      | 1986-08-28 | 301212       |
+--------+------------+------------+-----------+--------+------------+--------------+
3 rows in set
```
> [8 Having](https://railsguides.jp/active_record_querying.html#or%E6%9D%A1%E4%BB%B6:~:text=GROUP%20BY%20status-,8%20Having,-SQL%E3%81%A7%E3%81%AF%E3%80%81)
### 上記の結果を利用してコンソール上に以下のようなフォーマットでputsしてください。
```
irb(main):158:0> Employee.joins(:salaries).where(emp_no: [10001, 10002, 10003]).select('employees.*, sum(salary) as total_salary').group(:emp_no).each do |employee|
irb(main):159:1*   puts "emp_no: #{employee.emp_no}"
irb(main):160:1>   puts "full_name: #{employee.first_name} #{employee.last_name}"
irb(main):161:1>   puts "total_salary: #{employee.total_salary}"
irb(main):162:1> end
  Employee Load (2.8ms)  SELECT employees.*, sum(salary) as total_salary FROM `employees` INNER JOIN `salaries` ON `salaries`.`emp_no` = `employees`.`emp_no` WHERE `employees`.`emp_no` IN (10001, 10002, 10003) GROUP BY `employees`.`emp_no`
emp_no: 10001
full_name: Georgi Facello
total_salary: 1281612
emp_no: 10002
full_name: Bezalel Simmel
total_salary: 413127
emp_no: 10003
full_name: Parto Bamford
total_salary: 301212
+--------+------------+------------+-----------+--------+------------+--------------+
| emp_no | birth_date | first_name | last_name | gender | hire_date  | total_salary |
+--------+------------+------------+-----------+--------+------------+--------------+
| 10001  | 1953-09-02 | Georgi     | Facello   | M      | 1986-06-26 | 1281612      |
| 10002  | 1964-06-02 | Bezalel    | Simmel    | F      | 1985-11-21 | 413127       |
| 10003  | 1959-12-03 | Parto      | Bamford   | M      | 1986-08-28 | 301212       |
+--------+------------+------------+-----------+--------+------------+--------------+
3 rows in set
```

## その他メモ
### 内部結合と外部結合について
- 内部結合とは、複数のテーブルを結合する際に指定したカラムが一致したものだけを結合するもの。双方のテーブルで、カラムが合致しないものは取得結果に含まれない。
```rb
# テーブルAにテーブルBを内部結合する場合のSQL
SELECT 取得するカラム（なければアスタリスク） FROM テーブルA
  INNER JOIN テーブルB
    ON 結合条件(テーブルAの主キー = テーブルBの外部キー など)
```
- 外部結合とは、内部結合の結果に加え、どちらかのテーブルにしか存在しないデータも取得結果に含める。
- 外部結合には2つの種類がある。
  - LEFT OUTER JOINとは、左側のテーブル(FROM側のテーブル)を軸にして結合を行う。
  - RIGHT OUTER JOINとは、右側のテーブル(JOINするテーブル)を軸にして結合を行う
```rb
# テーブルAにテーブルBを外部結合する場合のSQL
SELECT 取得するカラム FROM テーブルA
  LEFT(RIGHT) OUTER JOIN　テーブルB
    ON 結合の条件
```
> [SQL素人でも分かるテーブル結合(inner joinとouter join) - Zenn](https://zenn.dev/naoki_mochizuki/articles/60603b2cdc273cd51c59)

##　結合のメソッドについて調べたこと
### joinsを使った結合
- デフォルトでINNER JOINを行う。OUTER JOINを行う場合、`left_joins`メソッドを使う。
- ActiveRecordのオブジェクトをキャッシュしないので、メモリの消費量を抑えることができる。結合先のテーブルの条件で絞りたいときに最適（但し、結合先テーブルのデータは取得しない場合）
- associationをキャッシュしないため、eager loadには使えない（？？）

### eager_load
- 指定したテーブルをLEFT OUTER JOINで結合し、取得してキャシュする。
- クエリが一つで済むので、preloadより早いことがある。
- joinsで結合しているので、取得した関連テーブルで絞り込みできる。

### preload
- 指定した関連ごとにクエリを作成し、関連テーブルのデータを取得してキャッシュする。
- 引数に多対多の関連先を渡した場合、中間テーブルを介して（中間テーブルも含めて）関連先のテーブルを取得する。
- テーブルを結合しているわけではないので、関連テーブルで絞り込みはできない。（例外を返される）

### includes
- デフォルトではpleloadと同じ挙動をする。
- 関連テーブルの条件で絞り込みなどをする場合、eager_loadと同じ挙動をする。

> [preload、eager_load、includesの挙動を理解して使い分ける](https://tech.stmn.co.jp/entry/2020/11/30/145159)
> [ActiveRecordのjoinsとpreloadとincludesとeager_loadの違い - Qiita](https://qiita.com/k0kubun/items/80c5a5494f53bb88dc58)
