# コードとAuto Layout

IBを使わない場合のAutoLayoutの方法

## 制約の生成

NSLayoutConstraintとNSLayoutAnchorがある
後者が主に使われているため後者のみ書く
前者は古い

### NSLayoutAnchor

利点

- リーダブルである、自然言語のようにCodeをかける
- メソッドチェーンすることができ

Code例

``` swift

let constraint1 = textField.topAnchor.constraintEqualTopAnchor(view.topAnchor, constant: 10.0)
let constraint2 = textField.widthAnchor.constraintEqualToConstant(20.0)

```

[関係図](image/5-1.png)

### アンカー、　Viewを固定する対象

- 上の例で登録するtopAnchorやwidthAnchorに含まれる「Anchor」とは？
  - オブジェクト内で固定する対象を指定し、指定した箇所に対して制約を生成する役割がある

オブジェクト内で固定する対象はAttributeという形で表され、このAttributeの性質に合わせてNSLayoutConstraintを生成させるためのクラスであるNSLayoutAnchorを継承した以下の３つのクラスが定義されている

- NSLayout

明日ここやる
