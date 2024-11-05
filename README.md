# db-test

# 使える関数たち

## generate_series()

```sql
SELECT generate_series(1,10); -- 1,2,3,4,5,6,7,8,9,10
```

で 1~10 の連続値を生成する。
第三引数を指定することで、刻み幅を指定することもできる。

```sql
SELECT generate_series(1,10,2) -- 1,3,5,7,9
```

## lpad() rpad()

文字列を指定された長さに拡張し、指定した文字で埋める関数。
lpad は前(左)埋め rpad は後ろ(右)埋め

```sql
select lpad('12345', 8, 0); -- 00012345
```

第一引数に対象の文字列
第二引数に長さ指定。（何文字にしたいか）
第三引数になにの文字で埋めたいか。

### generate_series と lpad の合わせ技

```sql
SELECT 'ユーザー' || lpad(gs :: text, 5, '0') FROM generate_series(1, 10000) gs;
```

gs :: text は generate_series()は Integer 型を返すのに対して lpad は文字列型を受取る関数のため
text 型に cast するという記法。

## 一時間ごとのタイムスタンプを生成する

```sql
SELECT
  timestamp '2024-10-31 22:00:00' + '1 hour'::INTERVAL * i -- '2024-10-31'::timestampでも同義
FROM
  generate_series(1,3) AS i;
```
