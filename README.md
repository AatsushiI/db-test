- PostgreSQL CTE
- 結論
- CTEとは
    - 解説
    - 書き方
- 再帰の書き方
    - 親と子関係をもつ１つのテーブルを再帰CTEで調べる方法
- マテリアライズオプション
    - PostgreSQL 12でパフォーマンスが改善した話
        - 基本的にはマテリアライズするものだった。
- 一時ビューとの使い分け
    - フローチャート用意できればよりよいか。
- まとめ

# PostgreSQLの共通テーブル式（CTE）について

本記事ではPostgreSQLの共通テーブル式（Common Table Expressions）について、調べてみたので記事にしたいと思います。

なお、本記事では共通テーブル式のことをCTEを呼称することにします。

## 結論

複数回同じ処理を書くならCTEにするとパフォーマンスが向上するケースがある。

ネストはできるだけ避けろ。

部品に分解して、レビュアーを助けろ

## CTEとは

CTEとは最初でひとことで説明すると、「WITH句によって、1つのクエリ内のために存在する一時テーブルを定義できる。」です。

WITH句内にはSELECT, INSERT, UPDATE, DELETEを取ることができ、メインクエリにはSELECT, INSERT, UPDATE, DELETEに加えてMARGEを使用することできますが、本記事では主にSELECTに絞って解説したいと思います。

## CTEの書き方

下記のような構文で、WITH句による一時テーブルの定義とメインクエリでそれを使用することができます。

月ごとの給料を記録したemployees_salaryテーブルから2024年の合計を取得しイッセンマン以上の社員一覧を取得するクエリです。

```sql
WITH 2024_salary_total AS (
	SELECT 
		employeeid, 
		sum(salary) as total_salary,
	FROM 
		employees_salary
	WHERE 
		term = '2024'
	GROUP BY 
		employeeid
)
SELECT 
	employeeid 
FROM 
	2024_salary_total 
WHERE total_salary >= 10,000,000;
```

CTEを使用しない場合

```sql
SELECT 
    employeeid
FROM 
  (
    SELECT 
      employeeid, 
      SUM(salary) AS total_salary
    FROM 
      employees_salary
    WHERE 
      term = '2024'
    GROUP BY 
      employeeid
  ) AS salary_summary
WHERE 
  total_salary >= 10000000;
```

このくらいのクエリなら可読性はあまり変わらないのですが、CTEを使った場合ネストが1つ浅いことはわかると思います。

CTEはこのように問い合わせを部品に分解することで、可読性の向上、責任所在の明確化、再利用性を持たせることができます。




## 再帰問い合わせもできるよ

WITH句はRECURSIVE修飾子を使用することで、再帰的な問い合わせを実現することが可能です。

構文は下記のような形です。

```sql
WITH RECURSIVE t(n) AS (
	VALUES(1) --非再帰的表現(non-recursive term)
	UNION ALL -- UNION or UNION ALL
	SELECT n+1 FROM t WHERE n < 100 -- 再帰的表現(recursive term)
)
SELECT sum(n) FROM t;
```

常に非再帰的表現と再帰的表現、それをつなぐUNION or UNION ALLによって構成されます。

まず最初に非再帰的表現が実行され、その後再帰的表現のクエリでn < 100がfalseになるまで繰り返し実行されます。

WHERE n < 100 のような終了条件が明示的にない場合は無限ループに陥る可能性があるため注意が必要です。

実際使用する際に n < 100のような条件をつけることは難しいかもしれないので、思想としては

基本的には再帰的表現部分がレコードを返さなくなる作りになっていることが重要です。

もう少し具体の例で見ていきましょう。

例えば、ファイルシステムの例を考えてみましょう。

```sql
CREATE TABLE file_system (
    id SERIAL PRIMARY KEY,       -- 自身のID
    name VARCHAR(100),           -- フォルダ名またはファイル名
    parentid INT,               -- 親フォルダのID
    isfile BOOLEAN              -- ファイルかどうか
);
```

このようなテーブルがあり、それぞれ親ディレクトリの参照をparentidでもっています。

これらをたどり階層構造を視覚的に出力してみましょう。

TODO: //hogeのようになってしまっているので修正

```sql
WITH RECURSIVE directory_path AS (
	SELECT
		id,
		name,
		parentid,
		name AS path,
		isfile
	FROM file_control
	WHERE parentid IS NULL 
	UNION ALL
  SELECT 
      f.id,
      f.name,
      f.parentid,
      dp.path || '/' || f.name AS path,
      f.isfile
  FROM file_control f
  INNER JOIN directory_path dp ON f.parentid = dp.id
)
SELECT id, name, path FROM directory_path;
```

これであればネストの深さがバラバラなディレクトリ構成のパスを一気に出力することができますね！

## まだまだこんなもんじゃないよCTEちゃん

CTEちゃんにはまだまだ有用な機能があります。

それはマテリアライズ化です。

冒頭で、CTEとは「1つのクエリ内のために存在する一時テーブルを定義できる。」と紹介しました。

マテリアライズ化とはメモリやディスク上に実体をつくるということです。

```sql
WITH cte1 AS (
	SELECT id FROM file_control WHERE 
)
```
