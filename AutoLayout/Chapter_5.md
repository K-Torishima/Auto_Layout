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

今日はちょっと休み
