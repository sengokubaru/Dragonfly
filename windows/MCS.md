# 要件の整理

- 親テーブルA：商品マスタ（商品番号・商品名・価格など）
- 子テーブルB：商品番号＋産地名（東京/神奈川/埼玉…）＋数量
- 例：
  - 入力（変数）
  - 産地名（例：東京）
  - 最小数（例：3）
  - 最大数（例：10）

- やりたい処理
  1. 子テーブルBの「東京」列を参照
  2. 最大数 ≥ 数量 ≥ 最小数 を満たす商品番号を抽出
  3. その商品番号に対応する親テーブルAの情報をすべて取得

# Power Automate（PA）での実装フロー

### 子テーブルBを取得
- SharePoint / Excel / Dataverse など、データソースに応じて
- 「一覧の取得」 アクションでテーブルBを取得。
### 産地名・最小数・最大数でフィルタ
- Filter array の条件例
```
@and(
  greaterOrEquals(item()?['東京'], variables('最小数')),
  lessOrEquals(item()?['東京'], variables('最大数'))
)
```
### 条件に合致した商品番号の配列を作成
- Select アクション
- From：Filter array の結果
- Map：
- Key：商品番号
- Value：item()?['商品番号']
### 親テーブルAを取得
- 同じく「一覧の取得」でテーブルAを取得。
### 親テーブルAを商品番号リストでフィルタ
- Filter array の条件例
- contains(variables('商品番号配列'), item()?['商品番号'])
### 📌 実装のポイント（重要）
- 子テーブルBの「産地名」は列名なので、
- 動的に列名を変える場合は item()?[variables('産地名')] のように書く
- 商品番号の配列は Select → Join で作ると扱いやすい
- 親テーブルAのフィルタは contains() を使うと JOIN 的に動く


# 汎用条件分岐テンプレート

## Operator（比較演算子）エンティティ テンプレート

値：greater_or_equal（以上）
シノニム：
- 以上
- ≧
- 以上の
- より上
- 4以上
- 高い方
- 上回る

値：greater_than（より大きい）
シノニム：
- より大きい
- より上
- 超える
- より高い
- より大きな
- 上のレベル

値：less_or_equal（以下）
シノニム：
- 以下
- ≦
- 以下の
- より下
- 低い方
- 下回る

値：less_than（未満）
シノニム：
- 未満
- より小さい
- より下
- 未満の
- 下のレベル

## 質問ノードでの設定テンプレート

質問文例：
レベルの条件を教えてください。（例：4以上、3より大きい、2未満など）
抽出するエンティティ：
- Operator
- Number

## 汎用化のための“抽象化テンプレート”
```
Switch(Operator,
    "greater_or_equal", int(TargetValue) >= int(Number),
    "greater_than",     int(TargetValue) >  int(Number),
    "less_or_equal",    int(TargetValue) <= int(Number),
    "less_than",        int(TargetValue) <  int(Number)
)
```

## 汎用条件式（レベル用）
```
Switch(Operator,
    "greater_or_equal", int(levelValue) >= int(Number),
    "greater_than",     int(levelValue) >  int(Number),
    "less_or_equal",    int(levelValue) <= int(Number),
    "less_than",        int(levelValue) <  int(Number)
)
```

## 汎用化の最終形

下限（Lower）と上限（Upper）を Optional にする
使うエンティティ
- OperatorLower（任意）
- NumberLower（任意）
- OperatorUpper（任意）
- NumberUpper（任意）

ユーザーが指定しなかった条件は 空（Blank） になる
→ その条件は無視される
→ 単一値・単一範囲・複数範囲すべてに対応できる

1. 特定のレベル
ユーザー入力：「レベルは4」
抽出：
- OperatorLower = greater_or_equal
- NumberLower = 4
- OperatorUpper = less_or_equal
- NumberUpper = 4
```
TargetValue >= 4 AND TargetValue <= 4
```
2. 単一範囲（例：4以上）
ユーザー入力：「4以上」
抽出：
- OperatorLower = greater_or_equal
- NumberLower = 4
- OperatorUpper = Blank
- NumberUpper = Blank
```
TargetValue >= 4 AND true
```
3. 複数範囲（例：2以上4以下）
ユーザー入力：「2以上4以下」
抽出：
- OperatorLower = greater_or_equal
- NumberLower = 2
- OperatorUpper = less_or_equal
- NumberUpper = 4
```
TargetValue >= 2 AND TargetValue <= 4
```
4. 汎用条件分岐テンプレート（完成版）
```
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
```
