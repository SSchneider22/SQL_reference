## 本ページについて

### 概要：
筆者がよく使うSQLの句・式・関数についての簡単な説明と、実行可能なサンプルクエリを記載したものです。<br>
本ページ作成の一番の目的は「筆者のためのTIPSを作ること」なので、説明はかなり省いています。<br>
また、文章が稚拙な場合もあります。<br>
各構文のより詳細な説明は他のウェブサイトに載っていますので、<br>
本ページを利用される場合は「SQLの主要機能の一覧表・練習帳」としてご利用ください。
<br>
<br>
「サンプルクエリを自分で変更して実行して、どう挙動が変わるか」をひたすら繰り返すのが、筆者は一番身につくのが速いと感じていますので、とにかくやってみることをオススメします。

## サンプルクエリ実行環境＆データ
### 環境
私はWSL2上のUbuntuにDockerを用いてPostgreSQLの環境を構築しています。<br>
「psqlが実行できるCLI環境」があれば、なんでも良いはずです。
- OS:WSL2 Ubuntu 18.04 LTS
- psql (PostgreSQL): 10.12 (Ubuntu 10.12-0ubuntu0.18.04.1)

私がWSL2~PostgreSQL環境構築の際に参考にしたサイト：<br>
https://necojackarc.hatenablog.com/entry/2019/10/09/080908


### 使用データ
Kaggleのデータを使用しています。<br>
https://www.kaggle.com/c/recruit-restaurant-visitor-forecasting/data

### スキーマ・テーブル
- スキーマ名：kaggle_recruit_data
- 各テーブル名：元のCSVの名称をそのまま使用

実際にPostgreSQL上にこのサンプルと同じスキーマとテーブルを構築する方は、本ページ末尾に「参考：スキーマ・テーブル構築手順」という見出しで記載していますので、こちらをご参照ください。

### メタコマンド
PostgreSQLをCLIから操作する際は、メタ文字でのコマンドも覚えておいたほうが良いです。例えば、クエリの出力結果が多すぎて止めたい場合は、「\q」または「Shift + q」で止めることが出来ます。詳しくは以下のURLなどを参照。<br>
https://www.dbonline.jp/postgresql/connect/index5.html



## SQLの種類
大きく以下の3種類に分けられる。
- DML(Data Manipulation Language)<br>
テーブルに対するデータの取得・追加・更新・削除を行う際に用いる。以下は例。
  - SELECT文：指定したテーブルから、レコードを抽出する。
  - INSERT文：指定したテーブルに対し、新しいレコードを登録する。
  - UPDATE文：指定したテーブルに対し、既存レコードの内容を変更する。
  - DELETE文：指定したテーブルに対し、既存レコードを削除する。

- DDL(Data Definition Language)<br>
データベースオブジェクトの生成や削除変更を行う際に用いる。以下は例。
  - CREATE TABLE文：クエリの内容に従い、新しいテーブルを作成する。
  - CREATE VIEW文：クエリの内容に従い、新しいビューを作成する。
  - DROP TABLE文：指定したテーブルを削除する。

- DCL(Data Control Language)<br>
トランザクションの制御を行う際に用いる。<br>
筆者が学習ベースにしているBigqueryではトランザクション処理をサポートしていないため、本ページでは扱う予定なし。
（Bigqueryは分析用途に特化しており、決済などのトランザクション処理が不可）

<br>
ここで、「文 statement」について簡単に説明する。<br>
1つの実行単位となる。 CLIから実行する際は、末尾に「;」が必要となることが多い印象。<br>


## 主要な句 clause
主要な「句 clause」について説明。<br>
節ともいう。文、またはクエリの構成要素。
最低限、句で改行すると読みやすいクエリになる。

### SELECT句・FROM句
#### 説明
- SELECT句：抽出したい列名を指定する
- FROM句：抽出したいテーブル名を指定する
#### 例
1. 指定したテーブルの全ての列を取得

```
SELECT
  *
FROM
  kaggle_recruit_data.air_reserve
```

2. 指定したテーブルから、指定した列を取得
```
SELECT
  reserve_visitors,
  visit_datetime
FROM
  kaggle_recruit_data.air_reserve
```

3. 指定したテーブルから、指定した列を取得。ただし、レコードに重複がある場合は削除
```
SELECT DISTINCT
  reserve_visitors
FROM
  kaggle_recruit_data.air_reserve
```

4. 指定した列の名称を変更した上で取得
```
SELECT DISTINCT
  reserve_visitors AS vistor
FROM
  kaggle_recruit_data.air_reserve
```

### SQLで使用できる比較演算子(=,<,>など)
WHERE句やCASE式など、様々な場面で使用できる比較演算子の一覧です
| 演算子 | 内容       |
| ------ | ---------- |
| =      | 左項と右項が等しい     |
| <      | 左項が右項未満     |
| >      | 左項が右項より大きい     |
| <=     | 左項が右項以下       |
| >=     | 左項が右項以上       |
| <>, != | 左項と右項が等しくない |

### SQLで使用できる論理演算子(AND,ORなど)
| 演算子 | 内容                 |
| ------ | -------------------- |
| AND    | ●●かつ▲▲   |
| OR     | ●●または▲▲ |
| NOT    | ●●でない     |


### WHERE句
#### 説明
指定した列に対して、各種演算子を用いて条件を指定して、その条件に一致するレコードのみを抽出するための句

#### 例
1. 2016年以前のデータを取得したいとき

```
SELECT
  air_store_id,
  visit_datetime
FROM
  kaggle_recruit_data.air_reserve
WHERE
  visit_datetime < '2017-01-01';
```

2. 2017年1月1日～12月31日のデータを取得したいとき。

```
SELECT
  air_store_id,
  visit_datetime
FROM
  kaggle_recruit_data.air_reserve
WHERE
  visit_datetime BETWEEN '2017-01-01' AND '2017-12-31';
```

3. reserve_visitorsが2,4,9のデータのみ取得したいとき

```
SELECT
  air_store_id,
  visit_datetime,
  reserve_visitors
FROM
  kaggle_recruit_data.air_reserve
WHERE
  reserve_visitors IN (2, 4, 9);
```

4. reserve_visitorsが2,4,9「以外」かつ2017年1月1日～12月31日のデータのみ取得したいとき

```
SELECT
  air_store_id,
  visit_datetime,
  reserve_visitors
FROM
  kaggle_recruit_data.air_reserve
WHERE
  reserve_visitors NOT IN (2, 4, 9)
  AND (visit_datetime BETWEEN '2017-01-01' AND '2017-12-31');
```

5. NULL値を持つデータを取得したいとき。（NULLのデータがないため、出力は0件です）

```
SELECT
  air_store_id,
  visit_datetime,
  reserve_visitors
FROM
  kaggle_recruit_data.air_reserve
WHERE
  reserve_visitors IS NULL;
```

6. NULL値でないデータを取得したいとき

```
SELECT
  air_store_id,
  visit_datetime,
  reserve_visitors
FROM
  kaggle_recruit_data.air_reserve
WHERE
  reserve_visitors IS NOT NULL;
```

7. ワイルドカードを用いて、前半部分一致で検索したいとき<br>
   （以下の例では、air_area_nameが「Tōkyō-to」から始まるレコードを抽出）

```
SELECT
  air_store_id,
  air_genre_name,
  air_area_name
FROM
  kaggle_recruit_data.air_store_info
WHERE
  air_area_name LIKE 'Tōkyō-to%';
/* LIKEは、「_」で何かしらの1文字が入る、という検索も可能*/
```

### ORDER BY句
#### 説明
レコードを指定した列で並び替えるための句
#### 例
1. レコードを昇順に並び替えて表示（並び替えキーは単一列）
```
SELECT DISTINCT
  reserve_visitors AS vistor
FROM
  kaggle_recruit_data.air_reserve
ORDER BY
  reserve_visitors ASC;
```

2. レコードを降順に並び替えて表示（並び替えキーは単一列）
```
SELECT DISTINCT
  reserve_visitors AS vistor
FROM
  kaggle_recruit_data.air_reserve
ORDER BY
  reserve_visitors DESC;
```

3. レコードをreserve_visitorsは昇順、visit_datetimeは降順に並び替えて表示
```
SELECT
  reserve_visitors,
  visit_datetime
FROM
  kaggle_recruit_data.air_reserve
ORDER BY
  reserve_visitors ASC, visit_datetime DESC;
```


### JOIN句（INNER,OUTER,CROSS,など含む）
#### 説明
複数のテーブルを結合することが出来る句
#### 結合の種類一覧
よく使うのは、「左外部結合」と「内部結合」<br>
各結合の内容については、以下のURLが参考になる<br>
https://www.sejuku.net/blog/73650

| 名称             | 内容         |
| ---------------- | ------------ |
| LEFT OUTER JOIN  | 左外部結合   |
| RIGHT OUTER JOIN | 右外部結合   |
| FULL OUTER JOIN  | 完全外部結合 |
| INNER JOIN       | 内部結合     |
| CROSS JOIN       | 交差結合     |
#### 例
1. air_reserveに対して、air_store_infoを左外部結合<br>
   (予約データに対してお店の情報を付け足す、というイメージ)
```
SELECT
  R.air_store_id,
  R.reserve_visitors,
  S.air_genre_name,
  S.air_area_name
FROM
  kaggle_recruit_data.air_reserve R
LEFT OUTER JOIN
  kaggle_recruit_data.air_store_info S
ON
  R.air_store_id = S.air_store_id;
```


### GROUP BY句・HAVING句
#### 説明
指定した条件と集計キーに応じて、結果を集計（複数レコードをまとめる）ことが出来る。<br>
GROUP BYで集計した結果に対して、WHERE句のような絞り込みを行いたい場合は、HAVING句を用いる。

#### 集計関数一覧
| 名称  | 内容                                         |
| ----- | -------------------------------------------- |
| SUM   | 集計された行の中で、指定した列の合計を出力   |
| MAX   | 集計された行の中で、指定した列の最大値を出力 |
| MIN   | 集計された行の中で、指定した列の最小値を出力 |
| AVG   | 集計された行の中で、指定した列の最小値を出力 |
| COUNT | 集計された行の数を出力                       |
#### 例
1. テーブル内のレコード総数を出力する
```
SELECT
  COUNT(*) AS record_count
FROM
  kaggle_recruit_data.air_reserve;
```

2. air_store_id別に、reserve_visitorsの合計を出力
```
SELECT
  air_store_id,
  SUM(reserve_visitors) AS sum_reserve_visitors
FROM
  kaggle_recruit_data.air_reserve
GROUP BY
  air_store_id;
```

3. air_store_id別の、reserve_visitorsの合計値が100以上のデータを出力
```
SELECT
  air_store_id,
  SUM(reserve_visitors) AS sum_reserve_visitors
FROM
  kaggle_recruit_data.air_reserve
GROUP BY
  air_store_id
HAVING
  SUM(reserve_visitors) >= 100;
```

## 式 experssion

### CASE式
#### 説明
出力結果を式内で指定した条件で抽出結果を変えることができる式
#### ポイント
**必ず「ELSE」を入れること！！** 入れないと、条件に一致しない場合NULLになってしまう
#### 例
1. reserve_datetimeとvisit_datetimeの日付の差が1日以上、<br>
   つまり前日までに予約がされているならばTRUE、<br>
   そうでないならばFALSEを「reserved_by_previousday」として出力
```
SELECT
  hpg_store_id,
  visit_datetime,
  reserve_datetime,
  reserve_visitors,
  CASE WHEN (CAST(visit_datetime AS DATE) - CAST(reserve_datetime AS DATE)) > 0 THEN TRUE
    ELSE FALSE
    END AS reserved_by_previousday
FROM
  kaggle_recruit_data.hpg_reserve;
```

2. reserve_datetimeとvisit_datetimeの日付の差が1日以上、<br>
   つまり前日までに予約がされているならばTRUE、<br>
   そうでないならばFALSEを「reserved_by_previousday」として出力。この上で、TRUEのデータのみに絞り込む
```
SELECT
  hpg_store_id,
  visit_datetime,
  reserve_datetime,
  reserve_visitors,
  CASE WHEN (CAST(visit_datetime AS DATE) - CAST(reserve_datetime AS DATE)) > 0 THEN TRUE
    ELSE FALSE
    END AS reserved_by_previousday
FROM
  kaggle_recruit_data.hpg_reserve
WHERE
  CASE WHEN (CAST(visit_datetime AS DATE) - CAST(reserve_datetime AS DATE)) > 0 THEN TRUE
    ELSE FALSE
    END = TRUE;
```

## 文字列関数

### LENGTH関数
#### 説明
指定した列の各データの文字数を返す関数
#### 例
1. hpg_genre_nameの文字数を出力
```
SELECT
  hpg_store_id,
  hpg_genre_name,
  LENGTH(hpg_genre_name) AS charcount_genre_name
FROM
  kaggle_recruit_data.hpg_store_info;
```

### TRIM関数
#### 説明
指定した列のデータに対して、
- 引数無しの場合：左右のスペースを削除
- 引数ありの場合：削除したい文字を引数に入れ、その文字を削除する
#### 例
1. hpg_genre_nameのデータに含まれている「 style」を削除
```
SELECT
  hpg_store_id,
  hpg_genre_name,
  TRIM(hpg_genre_name, ' style') AS trim_genre_name
FROM
  kaggle_recruit_data.hpg_store_info;
```

### REPLACE関数
#### 説明
指定した列のデータに対して、置換前と置換後の文字を指定して置き換える
#### 例
1. hpg_genre_nameのデータに含まれている「style」を「restaurant」に置換
```
SELECT
  hpg_store_id,
  hpg_genre_name,
  REPLACE(hpg_genre_name, 'style', 'restaurant') AS trim_genre_name
FROM
  kaggle_recruit_data.hpg_store_info;
```

### SUBSTR関数
#### 説明
指定した列のデータに対して、文字列の一部を抽出する関数<br>
DB製品によって、関数名が異なるため注意。使い方は同じ
- SUBSTR : Bigquery,Oracle,PostgreSQL
- SUBSTERING : SQL Server,MySQL,Redshift
#### 例
1. hpg_store_idの1文字目～3文字目を抽出
```
SELECT
  hpg_store_id,
  hpg_genre_name,
  SUBSTR(hpg_store_id, 1,3) AS trim_genre_name
FROM
  kaggle_recruit_data.hpg_store_info;
```

### CONCAT関数・||演算子
#### 説明
複数の文字列を連結することが出来る関数<br>
||を用いても連結が可能。DB製品によっては、どちらかしか用意されていない場合もあるため、要確認。
#### 例
1. air_store_idとair_genre_nameを、「_」を間に挟んで連結する。CONCAT関数を使用したとき。
```
SELECT
  air_store_id,
  air_genre_name,
  CONCAT(air_store_id,'_',air_genre_name)
FROM kaggle_recruit_data.air_store_info;
```
2. air_store_idとair_genre_nameを、「_」を間に挟んで連結する。||演算子を使用したとき。
```
SELECT
  air_store_id,
  air_genre_name,
  air_store_id || '_' || air_genre_name
FROM kaggle_recruit_data.air_store_info;
```


## 数学関数
公式ドキュメントは以下<br>
https://www.postgresql.jp/document/10/html/functions-math.html

### 基本的な数学関数
少し説明がわかりづらいROUNDとTRUNC以外は、以下の表にまとめる。

|項目|SQLでの演算子・関数名|使い方|
|-|-|-|
|足し算|+|(数値型)+(数値型)|
|引き算|-|(数値型)-(数値型)|
|掛け算|* |(数値型)*(数値型)|
|割り算|/|(数値型)/(数値型)|
|余剰の計算|%|(数値型)%(数値型)|
|絶対値|ABS()|ABS(X) --> Xの絶対値を返す|
|三角関数|SIN(),COS(),TAN()|SIN(X) --> Xのサインを返す(-1~1)|
|指数関数|EXP()|EXP(X) --> eのX乗を返す|
|自然対数|LN()|LN(X) --> Xの自然対数（eを底とする対数）を返す|
|常用対数|LOG()|LOG(X) --> 10を底とする対数を返す。引数追加で底の変更も可能|
|べき乗|POWER()|POWER(X,Y) --> XをY乗した値を返す|
|平方根|SQRT()|SQRT(X) --> Xの平方根を返す|
|符号の出力(-1,0,+1を出力)|SIGN()|SIGN(X) --> Xの符号を返す|

### ROUND関数
#### 説明
指定した列のデータに対して、四捨五入(DOUBLE PRECISION型の場合は五捨五超入)する関数。
このややこしい事象については、以下を参考にした。<br>
https://qiita.com/fujii_masao/items/c79575fb57827f658063

<br>
また、サンプルクエリの中で「::NUMERIC」のような記述をしているが、
これは通常CAST関数などを用いる型変換の、簡易記述ver。

#### 例
1. latitudeを、少数第一位で四捨五入
```
SELECT
  latitude,
  ROUND(latitude::NUMERIC) AS round_numeric_latitude
FROM
  kaggle_recruit_data.hpg_store_info;
```

2. latitudeを、少数第一位で五捨五超入
```
SELECT
  latitude,
  ROUND(latitude::DOUBLE PRECISION) AS round_double_latitude
FROM
  kaggle_recruit_data.hpg_store_info;
```


### TRUNC関数
#### 説明
指定した列のデータに対して、指定桁未満で切り捨てる関数。NUMERIC型に対して使用可能。
DOUBLE PRECISION型だとpostgreSQLのVer8.X以上ではエラーになる模様。
#### 例
1. latitudeを、少数第2位以上を残して、切り捨て
```
SELECT
  latitude,
  TRUNC(latitude::NUMERIC, 2) AS round_latitude
FROM
  kaggle_recruit_data.hpg_store_info;
```


## 日付・時間(DATE/TIME/TIMESTAMP)関数＆演算

### 日時に関する3つの型
日時に関しては、以下3つが主な型。<br>
TIMESTAMP型の末尾の「+00」は、UTCを軸にどれだけ時差があるかを示している<br>
|型名|サンプル|
|-|-|
|DATE|2016-12-25|
|TIME|05:30:00|
|TIMESTAMP|2016-12-25 05:30:00+00|

### 日時関数を使用する際の”part”について
日時関数では、年・月・日・時・分など、どの部分(part)を使うのか指定する場合がある<br>
このpartについて、代表的なものを下記にまとめる<br>
(以下はBigqueryにおけるpartを記載。他のDB製品では異なるかもしれないため、要確認)
|part|内容|
|-|-|
|YEAR|年|
|MONTH|月|
|WEEK|週番号0~53 (月曜日が開始日)|
|DAY|日|
|HOUR|時|
|MINUTE|分|
|SECOND|秒|

### 参考URL
日時関数関係は公式ドキュメントを見ると、より詳細なオプションが記載されている<br>
https://www.postgresql.jp/document/10/html/datatype-datetime.html

### CURRENT_(DATE/TIME/TIMESTAMP)関数
#### 説明
現在の日時をTIMESTAMP型で取得する関数
同様の関数として、CURRENT_DATE、CURRENT_TIME、CURRENT_DATETIMEなどがある
#### 例
1. 今日の日付をTIMESTAMP型で取得
```
SELECT
  CURRENT_TIMESTAMP;
```

### EXTRACT関数
#### 説明
日時データ(DATE型,TIME型,TIMESTAMP型)から、part部に指定した内容を取得する関数
#### 例
1. DATE型のデータから「日(DAY)」を取得
```
SELECT EXTRACT('DAY' FROM  DATE('2018-07-15'));
```

2. TIMESTAMP型であるvisit_datetimeから、「時(HOUR)」を取得
```
SELECT
  air_store_id,
  visit_datetime,
  EXTRACT('HOUR' FROM visit_datetime) AS hour_of_visit_datetime
FROM
  kaggle_recruit_data.air_reserve;
```

### (DATE/TIME/TIMESTAMP)型同士の差
#### 説明
PostgreSQLで、2つの同じ型の日時データ(DATE型,TIME型,TIMESTAMP型)の差を得るには、
引き算を用いるしかない。<br>
使用するDBによっては、差分を求める関数がある場合もある。要確認。<br>
以下、他製品での日時データの差分確認方法の参考
- DATE_DIFF：Bigquery,
- DATEDIFF：MySQL,SQL Server,Redshift
- 同じ日時型同士の引き算：PostgreSQL,Oracle
#### 例
1. どちらもTIMESTAMP型の、visit_datetimeとreserve_datetimeの日差(DAY)を取得
```
SELECT
  air_store_id,
  visit_datetime,
  reserve_datetime,
  CAST(visit_datetime AS DATE) - CAST(reserve_datetime AS DATE) AS day_diff
FROM
  kaggle_recruit_data.air_reserve;
```
2. DATE型のvisit_dateと、CURRENT_DATE()で今日の日付をDATE型で取得したときの、月差(MONTH)を取得。（満〇か月、で計算している）
```
SELECT
  air_store_id,
  visit_date,
  EXTRACT(YEAR FROM AGE(CURRENT_DATE ,visit_date))*12 + EXTRACT(MONTH FROM AGE(CURRENT_DATE ,visit_date)) AS month_diff
FROM
  kaggle_recruit_data.air_visit_data;
```

### (DATE/TIME/TIMESTAMP)_型に対する加算・減算
#### 説明
日時データ(DATE型,TIME型,TIMESTAMP型)に対して、指定した数値を加算・減算した日時を取得するためには、関数が用意されていないためそれぞれ演算処理を記述する。<br>
使用するDBによっては、関数が用意されているため要確認。<br>
以下は、「DATE_ADD」の例
- DATE_ADD：Bigquery,MySQL
- DATEADD：SQL Server,Redshift
- DATEADD関数無し、個別に演算作る必要あり：PostgreSQL,Oracle
#### 例
1. TIMESTAMP型のreserve_datetimeに、1日加算した値を出力
```
SELECT
  air_store_id,
  visit_datetime,
  reserve_datetime,
  reserve_datetime + ('1 DAYS') AS reserve_datetime_add1day
FROM
  kaggle_recruit_data.air_reserve;
```

2. TIMESTAMP型のreserve_datetimeに、1か月加算した値を出力
```
SELECT
  air_store_id,
  visit_datetime,
  reserve_datetime,
  reserve_datetime + ('1 MONTHS') AS reserve_datetime_add1month
FROM
  kaggle_recruit_data.air_reserve;
```

3. TIMESTAMP型のreserve_datetimeに、1週間減算した値を出力
```
SELECT
  air_store_id,
  visit_datetime,
  reserve_datetime,
  reserve_datetime + ('-1 WEEKS') AS reserve_datetime_sub1week
FROM
  kaggle_recruit_data.air_reserve;
```

### (DATE/TIME/TIMESTAMP)_TRUNC関数
#### 説明
日時データ(DATE型,TIME型,TIMESTAMP型)に対して、型は変換せずに、指定したpartの粒度の中で最も小さい値に変換する関数。<br>
月初、月末の日付を取得する時に便利。<br>
使用するDBによって、表記と使い方が異なるため要確認。<br>
以下は、「DATE_TRUNC」の例
- DATE_TRUNC：Bigquery,Redshift,PostgreSQL
- TRUNC：Oracle
- DATE_TRUNC関数無し、個別に演算作る必要あり：MySQL,SQL Server
#### 例
1. reserve_date_timeをその年月の中で最も小さい日付に変換する。
   (例：2016-10-31 --> 2016-10-01)
```
SELECT
  air_store_id,
  visit_datetime,
  reserve_datetime,
  DATE_TRUNC('MONTH', DATE(reserve_datetime)) AS firstday_reserve_datetime_month
FROM
  kaggle_recruit_data.air_reserve;
```

2. reserve_date_timeをその日付が該当する週の中での1日目(月曜日)に変換する。
   (例：2016-01-01 --> 2015-12-28)
```
SELECT
  air_store_id,
  visit_datetime,
  reserve_datetime,
  DATE_TRUNC('WEEK', DATE(reserve_datetime)) AS firstday_reserve_datetime_week
FROM
  kaggle_recruit_data.air_reserve;
```


### to_charを用いた任意の書式での日時情報取得
#### 説明
PostgreSQLでは、日時データ(DATE型,TIME型,TIMESTAMP型)を、任意の形(YYYYMMDDなど)に指定したフォーマットに変換する関数
このフォーマットについて、代表的なものを下記にまとめる<br>
|フォーマット|内容|
|-|-|
|"DAY"|その日時の曜日をすべて大文字で(FRIDAYなど)|
|"MON"|その日時の月をすべて大文字で(DECEMBERなど)|
|"YYYY"|年の4桁表記（2020など）|
|"MM"|月の2桁表記(01,12など)|
|"DD"|日の2桁表記(09,23など)|
<br>
上記以外にもたくさんのフォーマットがある、下記公式ドキュメントを参照。<br>
https://www.postgresql.jp/document/10/html/functions-formatting.html

<br>
<br>
使用するDBによって、表記と使い方が異なるため要確認。<br>
以下は、「FORMAT_DATE」の例

- FORMAT_DATE：Bigquery
- DATE_FORMAT：MySQL
- FORMAT関数などを駆使して、個別に作る必要あり：SQL Server,Redshift,PostgreSQL,Oracle

#### 例
1. reserve_datetimeを「YYYY-MM-DD weekday」の表記に変換
```
SELECT
  air_store_id,
  visit_datetime,
  reserve_datetime,
  to_char(reserve_datetime, 'YYYY-MM-DD DAY') AS date_weekday
FROM
  kaggle_recruit_data.air_reserve;
```


## 型変換関数

### CAST関数
#### 説明
型の変換を行う関数。PostgreSQLの型の一覧は以下の公式ドキュメントを見る。<br>
https://www.postgresql.jp/document/10/html/datatype.html

#### 例
1. reserve_datetimeをVARCHAR型に変換する
```
SELECT
  air_store_id,
  visit_datetime,
  reserve_datetime,
  CAST(reserve_datetime AS VARCHAR) AS str_reserve_datetime
FROM
  kaggle_recruit_data.air_reserve;
```

2. latitude(DOUBLE PRECISION型)をINT型に変換する。latitudeの少数第一位で四捨五入された結果が出力される。
```
SELECT
  latitude,
  CAST(latitude AS INT) AS INT_latitude
FROM kaggle_recruit_data.air_store_info;
```

## 集合演算子（複数のクエリの和・差・積）

### UNION句
#### 説明
和集合。2つのクエリの結果を縦に繋げて出力
Bigqueryの場合は、UNIONだけではエラーになるため、以下のどちらかを選択
- UNION DISTINCT ： レコードに重複があれば、削除する
- UNION ALL ： レコードに重複があっても、削除しない
#### 例
1. air_area_nameが「Tōkyō-to」から始まるレコードと、air_genre_nameが「Dining bar」であるレコードを繋げて出力。DISTINCTで重複レコードは削除
```
SELECT
  *
FROM
  kaggle_recruit_data.air_store_info
WHERE
  air_area_name LIKE 'Tōkyō-to%'
UNION DISTINCT
SELECT
  *
FROM
  kaggle_recruit_data.air_store_info
WHERE
  air_genre_name = 'Dining bar';
```

### EXCEPT句
#### 説明
差集合。1つ目の結果から、2つ目の結果と重複するレコードを取り除いて出力。<br>
2つのクエリがあって、それぞれ同じ結果を算出するはずなのに結果が異なる場合、その差分分析などに便利。<br>
Bigqueryの場合は、EXCEPTだけではエラーになるため、「EXCEPT DISTINCT」と記述する。
#### 例
1. air_area_nameが「Tōkyō-to」から始まり、air_genre_nameが「Dining bar」ではないレコードを出力。<br>
   （この例は1つのSELECT文だけでもWHERE ~ AND ~で書くことが出来ます、良い例ではないです）
```
SELECT
  *
FROM
  kaggle_recruit_data.air_store_info
WHERE
  air_area_name LIKE 'Tōkyō-to%'
EXCEPT DISTINCT
SELECT
  *
FROM
  kaggle_recruit_data.air_store_info
WHERE
  air_genre_name = 'Dining bar';
```

### INTERSECT句
#### 説明
積集合。1つ目の結果と2つ目の結果が重複するレコードを出力。<br>
2つ異なるクエリがあって、それぞれの結果から重複するレコードを確認したいときに便利。<br>
Bigqueryの場合は、INTERSECTだけではエラーになるため、「INTERSECT DISTINCT」と記述する。

#### 例
1. air_area_nameが「Tōkyō-to」から始まり、air_genre_nameが「Dining bar」であるレコードを出力。<br>
  （この例は1つのSELECT文だけでもWHERE ~ AND ~で書くことが出来ます、良い例ではないです）
```
SELECT
  *
FROM
  kaggle_recruit_data.air_store_info
WHERE
  air_area_name LIKE 'Tōkyō-to%'
INTERSECT DISTINCT
SELECT
  *
FROM
  kaggle_recruit_data.air_store_info
WHERE
  air_genre_name = 'Dining bar';
```


## 副問合せ(サブクエリ)
あるクエリの結果を用いて、別のクエリを実行したいときに使用するのが副問合せ、サブクエリとも言う。<br>
FROM句の中で書くことも出来るが、可読性が落ちる場合が多いため、WITH句で書くことを推奨。

### 単一行副問合せ
#### 説明
サブクエリが出力する結果を1列1レコードとして用いる手法のこと。
#### 例
1. reserve_visitorsが、reserve_visitors全レコード平均値以上であるレコードを出力する
```
SELECT
  air_store_id,
  visit_datetime,
  reserve_datetime,
  reserve_visitors
FROM
  kaggle_recruit_data.air_reserve
WHERE
  reserve_visitors >= (
    SELECT
      AVG(reserve_visitors)
    FROM
      kaggle_recruit_data.air_reserve
  );
```

### WITH句
#### 説明
サブクエリを主となるSELECT文の中に書くのではなく、外に出して書くことが出来るようになるのがWITH句。<br>
- WITH句を用いるメリット
  - クエリのネストが深くならないので、クエリの可読性が上がる
  - 同じサブクエリを何度も使いたい場合、1度WITH句で定義することで使いまわせる

<br>
また、一度WITH句では複数のサブクエリが定義可能であり、同じクエリ内の後続で定義するサブクエリでは、事前に定義されているサブクエリが使用可能。文章ではわかりづらいため、以下の例3でも実践する。

#### 例
1. reserve_visitorsが、reserve_visitors全レコード平均値以上であるレコードを出力する（WITH句ver）
```
WITH avg_reserve_visitors AS (
  SELECT
    AVG(reserve_visitors) AS avg_reserve_visitors
  FROM
    kaggle_recruit_data.air_reserve
)
SELECT
  A.air_store_id,
  A.visit_datetime,
  A.reserve_datetime,
  A.reserve_visitors
FROM
  kaggle_recruit_data.air_reserve A,
  avg_reserve_visitors SUB
WHERE
  reserve_visitors >= SUB.avg_reserve_visitors;
```

2. air_visit_dataのair_store_id別のレコード数を、air_reserveテーブルに左外部結合する
```
WITH record_count_of_visit_data AS (
  SELECT
    air_store_id,
    count(*) AS record_count
  FROM
    kaggle_recruit_data.air_visit_data
  GROUP BY
    air_store_id
)
SELECT
  A.air_store_id,
  A.visit_datetime,
  A.reserve_datetime,
  A.reserve_visitors,
  B.record_count
FROM
  kaggle_recruit_data.air_reserve A
LEFT OUTER JOIN
  record_count_of_visit_data B
ON
  A.air_store_id = B.air_store_id;
```

3. air_visit_dataのvisitorsが、visitorsの全レコード平均値以上であるデータの中で、air_store_id別にレコード数を集計する。その後、この集計したレコード数を、air_reserveテーブルに左外部結合する
```
WITH avg_visitors_tbl AS (
  SELECT
    AVG(visitors) AS avg_visitors
  FROM
    kaggle_recruit_data.air_visit_data
),record_count_of_visit_data AS (
  SELECT
    A.air_store_id,
    count(*) AS record_count
  FROM
    kaggle_recruit_data.air_visit_data A,
    avg_visitors_tbl SUB1
  WHERE
    A.visitors >= SUB1.avg_visitors
  GROUP BY
    air_store_id
)
SELECT
  B.air_store_id,
  B.visit_datetime,
  B.reserve_datetime,
  B.reserve_visitors,
  C.record_count
FROM
  kaggle_recruit_data.air_reserve B
LEFT OUTER JOIN
  record_count_of_visit_data C
ON
  B.air_store_id = C.air_store_id;
```

### EXISTS句
#### 説明
2つの同じキー項目を持つテーブルにおいて、片方のテーブルでの結果をサブクエリとしてフィルタに用いて、もう片方のテーブルから結果を得るときに使用する。<br>
(EXISTS句はフィルタとして使う場合が多いため、WHERE句やHAVING句の中で書くことが多いため、こういった説明にしている。逆にこれ以外の用途がパッと浮かびません…)

#### 例
1. air_visit_dataのair_store_id別のvisitorsの合計値が2000以上であるair_store_idについて、air_reserveのair_store_id別のreserve_visitorsの合計値を出力する
```
SELECT
  A.air_store_id,
  SUM(A.reserve_visitors) AS sum_reserve_visitors
FROM
  kaggle_recruit_data.air_reserve A
WHERE
  A.visit_datetime BETWEEN '2017-01-01' AND '2017-12-31'
  AND EXISTS (
    SELECT
      B.air_store_id,
      SUM(B.visitors)
    FROM
      kaggle_recruit_data.air_visit_data B
    WHERE
      B.visit_date BETWEEN '2017-01-01' AND '2017-12-31'
    GROUP BY
      B.air_store_id
    HAVING
      A.air_store_id = B.air_store_id
      AND SUM(B.visitors) >= 2000
  )
GROUP BY
  A.air_store_id;
```


## 分析関数 (主にWINDOW関数)
### WINDOWS関数
#### 説明
あるテーブル内でGROUP BYを用いたような集計値を、元のテーブルのレコード数を変更することなく、元テーブルに1列追加することができる関数。<br>
例えば、「該当レコードの売上値/該当レコードが含まれる年の売上合計値」のようなデータが欲しいときに便利。<br>
WINDOW関数がないと、この「該当レコードの売上値/該当レコードが含まれる年の売上合計値」は記述に手間がかかる上、可読性も落ちてしまう。<br>
以下は、WINDOW関数がない場合の手順例。
1. 年間の売上合計値を集計するクエリをGROUP BYを用いて作る
2. 前工程で作成したクエリをサブクエリとして、元テーブルに対して左外部結合させるクエリを書く。

#### 例
1. air_store_id別に、reserve_visitorsの合計値を集計し、新しくvisitors_summaryとして1列付与する
```
SELECT
  air_store_id,
  visit_datetime,
  reserve_datetime,
  reserve_visitors,
  SUM(reserve_visitors) OVER (PARTITION BY air_store_id) AS visitors_summary
FROM
  kaggle_recruit_data.air_reserve;
```

2. air_store_id、visit_datetimeの年、visit_datetimeの月、この3項目別にreserve_visitorsの合計値を集計し、新しくvisitors_summary_YYMMとして1列付与する
```
SELECT
  air_store_id,
  visit_datetime,
  reserve_datetime,
  reserve_visitors,
  SUM(reserve_visitors) OVER (PARTITION BY air_store_id, TO_CHAR(visit_datetime, 'YY'), TO_CHAR(visit_datetime, 'MM') AS visitors_summary_YYMM
FROM
  kaggle_recruit_data.air_reserve;
```


## その他DML(Data Manipulation Language)

### INSERT文
#### 説明
指定したテーブルに対し、新しいデータを登録する文
#### 例
1. 1レコード、データの内容を指定して追加する
```
INSERT INTO kaggle_recruit_data.air_visit_data_copy (air_store_id, visit_date, visitors)
  VALUES ('air_dummy99999999999', CAST('2080-01-01' AS DATE), 901);

/*すべての列に対して値を設定してレコードを追加する場合は、以下のように列名の省略も可能*/
INSERT INTO kaggle_recruit_data.air_visit_data_copy
  VALUES ('air_dummy99999999999', CAST('2080-01-02' AS DATE), 902);
```

2. 複数レコード、データの内容を指定して追加する
```
INSERT INTO kaggle_recruit_data.air_visit_data_copy (air_store_id, visit_date, visitors)
  VALUES
    ('air_dummy99999999999', CAST('2080-01-03' AS DATE), 903),
    ('air_dummy99999999999', CAST('2080-01-04' AS DATE), 904);
```

3. SELECT文のクエリ結果を新しく追加する
```
INSERT INTO kaggle_recruit_data.air_visit_data_copy (air_store_id, visit_date, visitors)
  SELECT
    'air_dummy99999999999',
    CAST('2080-01-05' AS DATE),
    905;
```

### UPDATE文
#### 説明
指定したテーブルに対し、条件に一致するレコード・列の値を変更する
#### 例
以下のサンプルは、上記INSERT文のサンプル全て実施後に実行可能です。

1. visit_dateが2080-01-01であるレコードの、visitorsを800に変更する
```
UPDATE kaggle_recruit_data.air_visit_data_copy
SET visitors = 800
WHERE visit_date = DATE('2080-01-01');
```

2. コピー元のair_visit_dataに存在するレコードの、visitorsを700に変更する
```
UPDATE kaggle_recruit_data.air_visit_data_copy
SET visitors = 700
FROM kaggle_recruit_data.air_visit_data B
WHERE kaggle_recruit_data.air_visit_data_copy.air_store_id = B.air_store_id
  AND kaggle_recruit_data.air_visit_data_copy.visit_date = B.visit_date;
```


### DELETE文
#### 説明
指定したテーブルに対して、条件に一致するレコードを全て削除する
#### 例
1. visitorsが700であるレコードをすべて削除する
```
DELETE
FROM kaggle_recruit_data.air_visit_data_copy
WHERE visitors = 700;
```

2. （要注意！！簡単にできてしまうため、気を付ける事）テーブルのすべての行を削除する
```
DELETE
FROM kaggle_recruit_data.air_visit_data_copy;
```

## DDL(Data Definition Language)

### CREATE TABLE文
#### 説明
新しくテーブルを作成するための文。<br>
主キーやNULLの許可など、様々な制約をかけることが可能。<br>
詳細は以下のURLが参考になる。<br>
http://db-study.com/archives/233 <br>
https://www.postgresql.jp/document/10/html/sql-createtable.html

#### 例
本ページ末尾の「参考：スキーマ・テーブル構築手順」に事例あり。

### CREATE TABLE ~ AS (SELECT ~)文
#### 説明
SELECT文によって出力された結果をそのまま新しくテーブルとして作成する。

#### 例
1. air_reserveにair_store_id別のreserve_visitorsの合計値を集計した列を追加したテーブルを新しく作成
```
CREATE TABLE kaggle_recruit_data.air_reserve_addsummary AS
  SELECT
    air_store_id,
    visit_datetime,
    reserve_datetime,
    reserve_visitors,
    SUM(reserve_visitors) OVER (PARTITION BY air_store_id) AS visitors_summary
  FROM
    kaggle_recruit_data.air_reserve;
```

### CREATE VIEW文
#### 説明
viewは、SELECT文のクエリを登録するものであり、作成したviewを参照することで、
都度登録したSELECT文のクエリ結果を見ることができるようになるもの。
複数のテーブルの結合を行うSELECT文など、テーブルとして定義してしまうと都度そのテーブルの更新作業が必要となってしまうのに対して、viewは参照するたびにクエリを実行するため、更新作業を必要とせず常に各テーブルの最新情報を見ることが出来る。
#### 例
1. air_visit_dataのair_store_id別のレコード数を、air_reserveテーブルに左外部結合したものを、「air_reserve_add_visit_data」という名称のviewとして登録する
```
CREATE VIEW air_reserve_add_visit_data AS
WITH record_count_of_visit_data AS (
  SELECT
    air_store_id,
    count(*) AS record_count
  FROM
    kaggle_recruit_data.air_visit_data
  GROUP BY
    air_store_id
)
SELECT
  A.air_store_id,
  A.visit_datetime,
  A.reserve_datetime,
  A.reserve_visitors,
  B.record_count
FROM
  kaggle_recruit_data.air_reserve A
LEFT OUTER JOIN
  record_count_of_visit_data B
ON
  A.air_store_id = B.air_store_id;
```


### DROP TABLE文
#### 説明
テーブルを削除する文。簡単にできてしまうがゆえに、注意すること。
#### 例
1. air_reserve_addsummaryテーブルを削除する
```
DROP TABLE kaggle_recruit_data.air_reserve_addsummary;
```



## 参考：よく使うメタコマンド一覧
- psqlを終了する
```
\q
```

- 指定したスキーマのテーブル一覧を取得
```
/*書き方*/
\dt <スキーマ名>

/*例*/
\dt kaggle_recruit_data
```

- 指定したスキーマのテーブル一覧とそれぞれのテーブルの構造を取得
```
/*書き方*/
\d <スキーマ名>.*

/*例*/
\d kaggle_recruit_data.*
```


## 参考：スキーマ・テーブル構築手順

### 使用データと置き場
Kaggleのデータを使用しています。<br>
https://www.kaggle.com/c/recruit-restaurant-visitor-forecasting/data

<br>
このデータをUbuntu上の以下のディレクトリ上に全てのCSVを置いています。（全てのCSVファイルの文字コードを、UTF-8に変換しておくこと。）<br>
私はDockerでPostgreSQLを入れているため、\copyコマンドを使っています。
```
/home/[ユーザー名]/practice/postgresql/data
```

### 手順
1. psqlを起動。
```
psql
```

2. スキーマ「kaggle_recruit_data」を作る
```
CREATE SCHEMA kaggle_recruit_data;
```

3. テーブル「air_reserve」を作成
```
CREATE TABLE kaggle_recruit_data.air_reserve(
  "air_store_id" VARCHAR
  , "visit_datetime" TIMESTAMP
  , "reserve_datetime" TIMESTAMP
  , "reserve_visitors" INT
);
```

4. テーブル「air_reserve」に、CSVの内容をコピーする
```
\copy kaggle_recruit_data.air_reserve FROM '/home/[ユーザー名]/practice/postgresql/data/air_reserve.csv' ENCODING 'utf8' CSV HEADER DELIMITER ',';
```

5. テーブル「air_store_info」を作成
```
CREATE TABLE kaggle_recruit_data.air_store_info(
  "air_store_id" VARCHAR
  , "air_genre_name" VARCHAR
  , "air_area_name" VARCHAR
  , "latitude" DOUBLE PRECISION
  , "longitude" DOUBLE PRECISION
);
```

6. テーブル「air_store_info」に、CSVの内容をコピーする
```
\copy kaggle_recruit_data.air_store_info FROM '/home/[ユーザー名]/practice/postgresql/data/air_store_info.csv' ENCODING 'utf8' CSV HEADER DELIMITER ',';
```

7. テーブル「air_visit_data」を作成
```
CREATE TABLE kaggle_recruit_data.air_visit_data(
  "air_store_id" VARCHAR
  , "visit_date" DATE
  , "visitors" INT
);
```

8. テーブル「air_visit_data」に、CSVの内容をコピーする
```
\copy kaggle_recruit_data.air_visit_data FROM '/home/[ユーザー名]/practice/postgresql/data/air_visit_data.csv' ENCODING 'utf8' CSV HEADER DELIMITER ',';
```

9. テーブル「date_info」を作成
```
CREATE TABLE kaggle_recruit_data.date_info(
  "calendar_date" DATE
  , "day_of_week" VARCHAR
  , "holiday_flg" INT
);
```

10.  テーブル「date_info」に、CSVの内容をコピーする
```
\copy kaggle_recruit_data.date_info FROM '/home/[ユーザー名]/practice/postgresql/data/date_info.csv' ENCODING 'utf8' CSV HEADER DELIMITER ',';
```

11. テーブル「hpg_reserve」を作成
```
CREATE TABLE kaggle_recruit_data.hpg_reserve(
  "hpg_store_id" VARCHAR
  , "visit_datetime" TIMESTAMP
  , "reserve_datetime" TIMESTAMP
  , "reserve_visitors" INT
);
```

12. テーブル「hpg_reserve」に、CSVの内容をコピーする
```
\copy kaggle_recruit_data.hpg_reserve FROM '/home/[ユーザー名]/practice/postgresql/data/hpg_reserve.csv' ENCODING 'utf8' CSV HEADER DELIMITER ',';
```

13. テーブル「hpg_store_info」を作成
```
CREATE TABLE kaggle_recruit_data.hpg_store_info(
  "hpg_store_id" VARCHAR
  , "hpg_genre_name" VARCHAR
  , "hpg_area_name" VARCHAR
  , "latitude" DOUBLE PRECISION
  , "longitude" DOUBLE PRECISION
);
```

14. テーブル「hpg_store_info」に、CSVの内容をコピーする
```
\copy kaggle_recruit_data.hpg_store_info FROM '/home/[ユーザー名]/practice/postgresql/data/hpg_store_info.csv' ENCODING 'utf8' CSV HEADER DELIMITER ',';
```

15. テーブル「store_id_relation」を作成
```
CREATE TABLE kaggle_recruit_data.store_id_relation(
  "air_store_id" VARCHAR
  , "hpg_store_id" VARCHAR
);
```

16. テーブル「store_id_relation」に、CSVの内容をコピーする
```
\copy kaggle_recruit_data.store_id_relation FROM '/home/[ユーザー名]/practice/postgresql/data/store_id_relation.csv' ENCODING 'utf8' CSV HEADER DELIMITER ',';
```

17. DML文(INSERT,UPDATE,DELETEの練習用に、air_visit_dataと同じ構造のテーブルを作成する
```
CREATE TABLE kaggle_recruit_data.air_visit_data_copy
(LIKE kaggle_recruit_data.air_visit_data INCLUDING ALL);
```

18. air_visit_dataからair_visit_data_copyへデータをコピーする
```
INSERT INTO kaggle_recruit_data.air_visit_data_copy
SELECT * FROM kaggle_recruit_data.air_visit_data;
```