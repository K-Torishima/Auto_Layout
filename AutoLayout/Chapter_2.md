# Chapter2 Auto Layoutの基本概念

Auto Layoutとは、制約を用いたレイアウト方法
制約はあるインターフェースオブジェクトの関係性を定義する
Auto Layoutでは制約を与えることでレイアウトを決定する
設定した制約はAuto Layoutエンジンにとって解釈され、レイアウトのフレームが決定される
制約を理解することによって、よりアダプティブなレイアウトに対応することが可能

## 制約

制約（Constraint)とは、Auto　Layoutで用いるインターフェースオブジェクトにおける
位置やサイズの関係性を定義する概念である
Viewの相違関係を定義するので、画面サイズの影響を最小限に押さえながらレイアウトを定義できる
制約を用いると絶対座標を用いてレイアウトした場合に比べ、より少ない情報で柔軟にレイアウトできる

## 制約式

AutoLayoutエンジンは制約を表した数式である、制約式の連立方程式を解くことでレイアウトを決定している
例
Button２の上端（Button2.top)がButton1の下端(Button1.bottom)から8ptの距離にあるというものの式

``` swift
Button2.top = Button1.bottom + 8
```

iOSでは左上が原点である

Auto Layoutでは、複数の制約が与えられた場合、上記の式を複数持つ線型方程式を解いている
Cassowaryというツールキットで用いられたアルゴリズムを使用している

## 外接矩形（がいせつくけい） AlignmentRectangle

auto layoutは各コンポーネントのフレームではなく外接矩形を用いている
外接矩形は装飾を除いたViewのこと
コンテンツをベースとした相対的なレイアウト手法であるため、各コンポーネントに対して装飾が追加されてもコンテンツやレイアウトに影響を与えない必要がある
そこでレイアウトの基準として外接矩形が用いられる

Xcodeで外接矩形を確かめる方法がある
[product] -> [Scheme] -> [Edit Scheme] -> [Run] -> [Alignment] 起動オプションを設定できる
[Alignments passed on Launch] に

- UIViewShowAlignmentRectsをチェックすると、デバック時に外接矩形を確認できるようになる

<image src="https://user-images.githubusercontent.com/52149750/101587835-5c811b80-3a28-11eb-9a83-ac586ee73f4f.png" width="500">

## 制約の定義する

制約を定義するには

- 開発者が、明示的に制約オブジェクトを生成する方法
  - NSLayoutConstraint を生成することで制約を指定する

- 予めViewオブジェクトが持つコンテンツによって決定させる制約を用いる方法
  - IntrinsicContent Size を有効活用することで制約を与える

この二種類

### NSLayoutConstraint 制約を設定するためのクラス

NSLayoutConstraintを精製してViewに追加することで、制約を設定したレイアウトを生成することができる
NSLayoutConstraintはInterface BuilderもしくはCodeから生成できる

``` swift

convenience init(item view1: AnyObject,
            attribute attr1: NSLayoutAttribute,
         relatedBy relation: NSLayoutRelation,
               toItem view2: AnyObject?,
            attribute attr2: NSLayoutAttribute,
      multiplier multiplier: CGFloat,
                 constant c: CGFloat)


let leftConstraint = NSLayoutConstraint(item: contentView, // 制約を追加する対象のViewA
                                   attribute: .Left,       // ViewAの制約を追加する位置　
                                   relatedBy: .Equal,      // View達の関係　＝　≧、≦　から選択
                                     toItem : view,        // 制約を追加する対象のViewB
                                   attribute: .Left,       // ViewBの制約を追加する処理　
                                   mutiplier: 1.0,         // 制約式の定数a
                                    constant: 10.0)        // 制約式の定数b


view.addConstraint(leftConstraint)
```

上記のパラメータは指定して書くことは少ないが、知識として必ずやくにたつ

以下各パラメータの種類に関して

### Item 制約の対象となるオブジェクト

itemとは制約を追加する対象となるインターフェースオブジェクトをさす

- firstItem, secondItemがある
- 両者が入れ替わると制約に関わる数値も変わる

### Attribute 対象となるオブジェクトの要素

Attributeは、対象となるオブジェクトのどこに対して制約を設定するのか示す
FirstItem, SecondItemに対して、Attributeを設定でき、それぞれFirst Attribute Second AttriそれぞれFirst Attribute Second Attribute
と呼ばれる

NSLayoutAttributeは次のenumとして定義されている

``` swift

enum NSLayoutAttrbute: Int {
    case Left
    case Right
    case Top
    case Bottom
    case Leading
    case Trailing
    case Width
    case Height
    case CenterX
    case CenterY
    case Baseline
    case FirstBaseline
    case LeftMargin
    case RightMargin
    case TopMargin
    case BottomMargin
    case LeadingMargin
    case TrailingMargin
    case CenterXWithinMargins
    case CenterYWithinMargins
    case NotAnAttrbute
}

```

それぞれの要素は次の通り

#### Left, Right, Top, Bottom

それぞれ、あるViewの境界線となる外接矩形の左端、右端、下端、上端を示す
言い方を変えると
X座標上の最小と最大の値、Y座標の最小と最大の値

#### Leading, Trailing

leading(先頭)　trailing(末尾)
言語によて変わるので、基本的にこれを使う

#### CenterX, CenterY

インターフェースオブジェクトの外接矩形のX座標とY座標における中心を示す
オブジェクトを親Viewに揃えたいときに利用する

#### Baseline, FirstBaseline

どちらもコンポーネントのテキストのベースラインを示す
テキストのベースラインとはテキストを整列するための基準線であり、日本語では文字の下端に存在する
アルファベットを整列するための基準線であり、日本語では文字の下端に存在する
アルファベットではgや、jなど、この基準線より、下に描画する文字もある

文字列が一行の時はこれらのAttributeに違いはない
複数の場合Baselineは文字列の最終行を示し、FirstBaselineは文字列の行のベースラインを示す
BaselineはLastBaselineである
文字列を縦方向に揃える場合、特に理由がなければ、Attributeを使うのが良い

#### マージン

LeftMargin　RightMargin TopMargin BottomMargin, LeadingMagin, TailingMargin
は、対応するAttributeのマージンを表す
CenterYWithinMargin, CenterXWithinMargin　X、Yにおける中央をさす

ここのサスマージンとは、Viewを置いたときの隙間
標準的なアプリのレイアウトに近づける

#### NotAnAttribute

名前のとおりAttributeではない

### Constant 制約の距離

制約定数はコンポーネントの相対距離や、サイズを表す
制約定数の変化をアニメーションにするさせることも可能

### Relation 要素の関係性

Relation はFirstItemとSecondItemのAttribute同士の関係性を示し、その値は以下のように宣言されている

``` swift
var relation: NSLayoutRelation { get }
```

 定義

 ``` swift

 enum NSLayoutRelation: Int { 
     case LessThanOrEqual
     case Equal
     case GreaterThanOrEqual
 }

 ```
