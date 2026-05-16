# 要件の整理
- 親テーブルA：商品マスタ（商品番号・商品名・価格など）
- 子テーブルB：商品番号＋産地名（東京/神奈川/埼玉…）＋数量
例：
　- 入力（変数）
　- 産地名（例：東京）
　- 最小数（例：3）
　- 最大数（例：10）
- やりたい処理
  1. 子テーブルBの「東京」列を参照
  2. 最大数 ≥ 数量 ≥ 最小数 を満たす商品番号を抽出
  3. その商品番号に対応する親テーブルAの情報をすべて取得

# Power Automate（PA）での実装フロー
1. 子テーブルBを取得
SharePoint / Excel / Dataverse など、データソースに応じて
「一覧の取得」 アクションでテーブルBを取得。
2. 産地名・最小数・最大数でフィルタ
Filter array の条件例
@and(
  greaterOrEquals(item()?['東京'], variables('最小数')),
  lessOrEquals(item()?['東京'], variables('最大数'))
)
3. 条件に合致した商品番号の配列を作成
Select アクション
From：Filter array の結果
Map：
Key：商品番号
Value：item()?['商品番号']
4. 親テーブルAを取得
同じく「一覧の取得」でテーブルAを取得。
5. 親テーブルAを商品番号リストでフィルタ
Filter array の条件例
contains(variables('商品番号配列'), item()?['商品番号'])


# 汎用条件分岐テンプレート

(
    IsBlank(OperatorLower) 
    || 
    Switch(
        OperatorLower,
        "greater_or_equal", int(TargetValue) >= int(NumberLower),
        "greater_than",     int(TargetValue) >  int(NumberLower),
        true
    )
)
&&
(
    IsBlank(OperatorUpper) 
    || 
    Switch(
        OperatorUpper,
        "less_or_equal", int(TargetValue) <= int(NumberUpper),
        "less_than",     int(TargetValue) <  int(NumberUpper),
        true
    )
)
