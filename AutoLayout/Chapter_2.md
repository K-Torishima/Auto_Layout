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

