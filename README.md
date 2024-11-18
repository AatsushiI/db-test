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
