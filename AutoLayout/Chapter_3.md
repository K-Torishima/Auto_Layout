# UIViewControllerとレイアウトをサポートするクラス

## レイアウトの構造とプロセス

UIViewControllerはMVCモデルにおけるコントローラの役割を補っている
iOSアプリでは基本的に一画面に最低一つのVCが存在する
このVCはモデルとviewの媒介であり、ユーザーインタラクションを受けたり、
ある画面において必要となるモデル操作の指令を出したり、Viewに対してレイアウトを指示したりする
そのためVC自身はモデルのロジックを知りませんし、Viewにおいてどのようにレイアウトが実施されているかは知りません
しかし、コントローラーは媒介的な役割を持つので、そのライフサイクルはレイアウトされるオブジェクトたちに大きな影響を与える
Viewオブジェクト生成、制約計算、レイアウト、さらに画面回転などiOSアプリのレイアウトに関係する多くの要素はこのVCをを介して行われるため、レイアウトを深く知るには、VCの理解が不可欠である

### 表示に関わる階層構造 -スクリーン、ウィンドウ、VC

VCは、viewプロパティとしてついとなるViewを持っている
これは、VCのみによって画面に表示されているわけではない
複数の表示に関わるオブジェクトが、階層構造となることでディスプレイに表示されているのです
iOSアプリのViewはMacアプリと同様に、次のような階層構造になっている

![階層構造](image/3-2.png)

### UIViewにおけるレイアウトのライフサイクル

各VCはついとなるUIViewを持っている
UIViewControllerのライフサイクルを理解するには、UIViewのレイアウトライフサイクルを知る必要がある

- 制約の更新
- フレームの更新
- レンダリング
  
#### 制約の更新

制約が変更されると、レイアウトにおけるコンポーネントの位置関係が変更されるので、その制約を満たすために、再計算がレイアウトエンジンのよって行われる、具体的にはUIViewのupdateConstrains()がよばれ、ボトムアップ（子Viewから親View）に制約の計算が実行される

制約の更新は、以下のような条件で引き起こされる

- 制約のactiveフラグによる有効化および無効化
- 制約の優先度変更
- 制約の追加や削除
- 制約を与えられたViewの階層変更

updateConstraints()をオーバーライドすることで、制約が更新するタイミングで独自の処理をつかすることができる
しかしこのメソッドがオーバーライドする必要がない
制約の変更が必要になるのは端末の回転やウインドウサイズの変更、オブジェクトの追加削除といったイベントが発生するタイミングであることが多いからである。
従って、そのイベントの記述に近い場所に制約の変更を記述した方が良いのは、制約更新において、パフォーマンスが十分でない場合がある
このメソッド内で制約を更新すると、レイアウトエンジンが複数の制約変更を特定のレイアウトパス中でバッチ処理できるため、制約を効率よく更新できる

開発者が制約の更新を明示的に実行することも可能、
updateConstrains()を直接呼ぶことはない。
代わりに、updateConstrainsIfNeeded()を呼ぶ

``` swift
self.updateConstraintsIfNeeded()
```

また、SetNeedsUpdateConstraints()
メソッドを呼ぶことでも、制約の更新ができる

このメソッドを呼ぶと、「制約を更新する必要がある」というフラグが立ち、
次のレイアウトパスで制約の再設計が実行される

updateConstraintsIfNeeded()とは違い、即座にupdateConstraints()が呼ばれるだけでなく、複数の箇所で制約を更新しても同じレイアウトパスで制約が更新されるため、少しパフォーマンスが良いのが特徴である

この辺もいろいろやりながら、覚えていくと良さそう

### フレームの更新

制約情報が更新されると、レイアウトを更新するために、レイアウトエンジンが計算した、フレーム情報をViewが受け取りLayoutSubViews()が呼ばれ、トップダウン
にフレームの更新が実施され、更新が必要であれば、制約も更新される
このフレームの更新を引き起こす条件の例として、次の物があげられる

- Viewのフレームが変更された時（端末が回転した時）
- サブビューが追加あるいは、削除された時
- UIScrollViewのサブクラスにおいて、contentOffSetが変更された時

レイアウトが必要なタイミングで、開発者が明示的にフレームの更新を実施することもできる

UIKitによるレイアウトは、スレットセーフではないため、メインスレッドでレイアウトメソッドを呼ぶ必要がある
このフレームの更新を実行するメソッド達は、制約の更新を実行するメソッドと比べるとよく使われる
LayoutSubViewsのオーバーライド

レイアウトの変更はlayoutSubViews()で実行されるため、このメソッドをオーバーライドすることで、制約付与によるレイアウトは難しいレイアウトでも実現できる
オーバーライドした際に、super.layoutSubViews()を実行する時点ですでに制約は更新される、
そのレイアウト情報を用いて処理を実行したい場合に有効
layoutSubViews()は、通常のレイアウトパスで、updateConstrains()と共に、バッチ処理的にレイアウトの変更を行うので、メインスレッドをブロックせずに、レイアウトの変更が実施できる

### レンダリング

レイアウトライフサイクルの３つ目のステップは、更新されたフレーム情報のレンダリング
フレーム情報が更新されたあと、ディスプレイに変更を表示するために、drawRect()メソッドが呼ばれる

iOSにおいては、drawRect()がそれぞれのViewを一度だけオフスクリーンバッファに描画する、描画されたレイヤーは、それぞれ独立した矩形となる
明示的にフラグを立てない限り、単純なオブジェクトの移動や、拡大縮小では再描画されない

再描画が必要な矩形は、drawRect()がグラフィックコンテクストを引数として、受け取り、描写処理を実行する
タイミングはAppleのiOS描画及び、印刷ガイドによると次の条件となる

- Viewの一部を隠している別のViewの移動または除去
- Hidden(非表示)になっていたViewの再表示（プロパティをNOに設定）
- Viewを画面外までスクロールし、際画面内に戻す
- ViewのSeetNeedDisplayメソッドまたはsetNeedsDisplayInRect()メソッドの明示的な呼び出し

drawRect()を直接呼び出すことはなく、setNeedDisplay()またはsetNeedsDisplayInRect()を呼び出すことで
再描画のトリガを与え実行する

タイマーを用いて一秒間に複数回レンダリングすることで、アニメーションにすることもできる

## ViewControllerのライフサイクル

![ライフサイクル](image/3-5.png)

詳細
VCが生成され画面に表示される段階になると、Viewの読み込みが始まる
Viewの呼び込みはloadView()で行われ、完了するとviewdidLoad()が呼ばれる、
次にVCは表示を開始し、viewWillAppear()を飛び出す
その後、レイアウトを実施しるviewWillLayoutSubViews()
が呼ばれることで、フレームの更新が、実施された後にviewDidLayoutSubViews()が呼ばれ、レイアウトが完了したことを通知する
最後にviewDidLayoutSubView()が呼ばれ、レイアウトが完了したことを通知する
最後にレイアウトされたviewが表示されるとviewDidAppear()が呼ばれて画面への表示が完了する

画面遷移をしたり、アプリを閉じたり、した時に呼ばれるのが、VCの非表示プロセス
VCが画面から消える直前にviewWillDisAppear()が呼ばれ、画面遷移が完了してVCがスクリーン常に表示されなくなった時、
viewDidDisapperが呼ばれる

### loadView()

Viewをメモリ上へ読み込み、対象のポインタと関連付けを行うメソッド
UIViewContrillerのメソッドを読んだタイミングで、つまりsuper.loadView()を読んだタイミングで、
UIViewControllerのViewプロパティにViewがセットされる

Storyboardが存在するときは、Storyboard上で定義された、Viewが生成され、viewプロパティに代入される
viewの読み込みに特別な操作を必要としない場合、特にオーバーライドする必要はない

一方、Storyboardを用いずにカスタムviewを定義している場合は、loadview()をオーバーライドし、
レイアウトを定義することができる

storyboardを用いたときは、親クラスが自動的にviewを生成しているが、この場合は開発者がviewを生成し、
viewプロパティに代入する必要がある、storyboardを用いない場合はsuper.loadview()を呼ばない方がメモリー効率が高くなる

### vieDidLoad()

loadView()が完了したときに呼ばれるメソッドで、オーバーライドを用いる
通常のメソッドでは、loadview()で読み込みが完了したUIオブジェクトにプロパティをセットしたり追加したり、
追加の処理を実行したりする

このメソッドはviewの読み込みが完了したタイミングで一度だけ呼ばれるため、クラス内で用いる値やオブジェクトを初期化する処理の
記述に適している

### viewWillApper()

viewが表示される直前に呼ばれるメソッドで、オーバーライドして用います
このメソッドは表示される直前であれば、いつでも呼ばれるため、メモリ上に存在する既に表示したviewが再度表示される際
にも同様に挙動をする

例えば、バックグラウンドからアプリを開き直した場合や、UIViewControllerにおけるタブの切り替えが当てはまる
表示の度に呼ばれるので、動的なセットに向いている
ただし、この段階ではUserにUI提供されていないため、計算コストが高い処理は避けた方が良い

### viewWillLayoutSubViews()

viewのレイアウトが開始するときに呼ばれるメソッドで、このときに実行したい処理がある場合は、オーバーライドして用いる
このメソッドは、viewControllerが読み込まれたあとだけではなく、端末の回転やviewの再表示などによって、viewが新しい大きさに変わった時に必ず呼ばれる

この時点では、サブviewのレイアウトは決定されていませんが、画面の向きは決定している
あまり推奨しないが、AutoLayout使用時に、画面の状態によって制約に変更を与えたい場合は、回転時に呼び出されるデリゲートメソッドではなく、このメソッド内で、変更を実施することもある

viewWillLayoutSubViewsが呼ばれたあと、VCのレイアウトが開始される、
このタイミングでは、UIviewのレイアウトサイクルでいうところの制約の更新とフレームの更新を実施される

### ViewDidLayoutSubviews()

viewのレイアウトが完了した時に呼ばれるメソッドで、この時に実行したい処理がある場合は、オーバーライドして用いる
UIViewのレイアウトサイクルにおける、layoutSubviews()によるフレームの更新が既に実施されているため、この時点では
VCの持つview内のレイアウトが確定する

### ViewDidAppear()

viewが表示された直後に呼ばれるメソッドで、オーバーライドして用いる
UIViewレイアウトサイクルの最後のステップであるレンダリングが終わったタイミングに当たる
表示開始のメソッドviewWillAppear()と対になっているため、バックグラウンドから復帰したときや、
UITabViewControllerにおけるタブの切り替え時などにもこのメソッドは呼ばれる、
既にUIがUserに提供されているため、User体験に直接影響を与えない処理を実行することが多い

## UIWindow

### UIWindowとは

UIWindowはviewを管理し、ディスプレイに表示するウインドウの役割を担っている
UIWindowは、タッチイベン、トフォーカス、座標変換に関して特別な機能が追加されたUIViewのサブクラス
ウィンドウは以下のようにキーウィンドウとなることで、これらの入力を受け付けることができる

このUIWindowは一つのアプリ上で複数表示できる
開発者が明示的にウィンドウを生成していなくても、実は新しいウィンドウが生成されていることがある
キーボードとかAlertViewは新しく作っている

### UIWindowの重なり順

複数のウィンドウが表示されているとき、どのウィンドウが上にくるか、つまりz軸方向の重なり順を決定するのが、
UIWindowが保持するwindowlevelプロパティ
windowlevelが大きいウィンドウほど手前に表示される
UIKitでは、Normal,Alert、StatusBarの3つのレベルが定義されている、

通常のウィンドウ、アラート、ステータスバーの３つの中では、アラートがもっとも手前に表示される

### ウィンドウからの画面サイズの取得

ウィンドウサイズのとりかた、

``` swift
let windowSize = UIApplication.sharedApplication().keyWindow?.bounds
```

マルチタスク対応などのものもあるため

## UStackView

 UIStackViewとは、縦もしくは、複数のサブviewを並べることができるインターフェース
 Auto Layoutを活用しており、UIStackViewにViewを並べると自動的に制約を設定するため、
 変更に対してつよいレイアウトを手軽に作ることができる

UIStackViewを用いると、縦横のレイアウトを組み合わせ、viewのコンポーネントを並べていくだけで、
製品で使われるようなレイアウトが実現できる

storyboardを用いていれば、キャンパスに配置したスタックviewに表示したいオブジェクトをドラッグしてくるだけで、
並べるviewを追加することができる
プログラムで追加削除を実行するには、追加の場合はinsertArrangedSubView(_view: UIView, atIntdex stackIndex: Int)
削除の場合はremoveArrangedSubView(_ view: UIView)を使う

UIStackViewはオブジェクトの追加削除に合わせたアニメーションを定義することも可能
このアニメーションはUIViewの保持するhoddenプロパティを用いて行う
以下のように実行すると、オブジェクト追加時のアニメーションを定義することができる

``` swift

additionalButton.hidden = true
self.stackView.insertArragedSubView(additonalButton, atIntdex: 1)

UIView.animateWithDuration(1.0) { () -> Void in
    self.additionalButton.hidden = false
}

```

### UIStackViewとレイアウト

UIStackViewでは、主にAxis(軸）　Alignmment(揃え方）Distribution（配置）、spacing（スペース）
の４つのプロパティを設定することで、細かいレイアウトを変更できる
spacingは配置したサブview同士の距離を定義するだけのものなので、省略し、ここでは他の3つのプロパティについて見ていく

### Axis　並べる方向

Axisはサブviewをhorizontal（水平方向に並べるか）Vertiocalに並べるかを決定している

### Alignment 揃え方

Alignmentは軸垂直方向のサブビュー達の位置を決定している
Codeから指定する場合はalignmentプロパティに次のenumで定義された値を代入する

``` swift

enum UIStackViewAlignment: Int {
    case Fill
    case Leading
    static var Top: UIStackViewAlignment { get }
    case FirstBaseline
    case Center
    case Trailing
    static var Bottom: UIStackViewAlignment { get }
    case LastBaseline
}
```

AxisがHorizontalの場合はサブビューたちが並ぶ際の縦方向の基準を選択する
プロパティはFill,　Top, Center, Bottom, Baselineから選択できる

Fillは高さが一番高いオブジェクトに他のオブジェクトの縦横比を変えて高さを揃え、
Topは上揃え、Centerは中央揃え、Bottomはした揃え、BaselineはビューオブジェクトのBalinese Attrbuteに揃える

Axisがverticalの場合、横方向におけるサブviewたちの位置を設定することができ、Fill Leading、Center, Trailingから選択できる
Fill幅が一番広いオブジェクトに他のオブジェクトの幅を揃え、Leadingは先頭揃え、Centerは中央揃え、Trailongは末尾揃え

### Distribution 並べ方

Distributionは、サブviewの軸方向の表示方法を決定するもので、Fill、Fill, Equally,　FillProportionally
Equal Spacing, Equal Centeringから選択できる　Codeから指定する場合はdistributionプロパティに
次のenumが定義された値を代入する

``` swift

enum UIViewDistribution: Int {
    case Fill
    case FillEqually
    case FillProportionally
    case Equalspacing
    case EqualCentering
}

```

Distribution Fillは　Intrinsic Content Size を満たしながらスタックビューの軸方向にサブビューを並べる
AxisがHorizontalの場合、スタックビュー内の各サブビューはそれぞれのintrinsic Content Size の幅だけの幅を持つことになる
そのため、DistributionにFillを選択している場合はスタックビューの幅はコンテンツの大きさによって決定され、軸方向の制約を指定することはできない

Distribution　Full Equallyは与えられたスタックビューの軸方向を満たすように、サブビューを均等な幅もしくは高さで並べます
AxisがHorizontalの場合各サブビューの幅は外部から与えられたUIStackViewの高さを均等に割った幅となる

Distribution　Fill　Proportionallyはスタックビューの軸方向を満たすように、サブビューのinstrinsic Content Sizeの比率に合わせた幅もしくは高さで並べる

AlignmentがCenterAxisがHorizontalのスタックビューの場合、画面のサイズに合わせて幅が決定される

Distribution Equal Spacing はサブビューのサイズをIntrinsic Content Size
から変更せずに、スタックビューの軸方向におけるサブビュー間のスペースを均一にして並べる

AlignmentはCenter、DistributionがEqual Spacing のスタックビュー
サブビューは　intrinsic Content Size に余ったスペースは均等に分割されていることがわかる
spacing　プロパティとDistributionのEqual Spacingによるスペースで、大きい方の値がレイアウトに採用されるので、注意が必要

Distribution Equal Centering はUIStackView内にあるIntrinsic Content Sizeの大きさのサブビュー達の中心の距離が均一になるように配置する

## UIStackViewとAutoLayout

UI StackViewは制約を活用しているため、スタックビューにオブジェクトを比べると、オブジェクトは制約が自動的に付与される
これらの制約は開発者自信が設定する制約にも影響を与えるため、UIStackViewがAuto Layoutの技術の上に成り立っていることを理解していると、デバックの際に役に立つ

スタックviewに関してはいろいろあるので、これもいろいろやってみた方が良さそう、
いろいろいじってみないとわからない。

## まとめ

- iOSアプリはUIScreen.mainScreen()で取得できるスクリーン、UIApplication.sharedApplication().keywindowで取得できるウインドウ、このウインドウのrootViewControllerのビューコントローラーの階層構造になっており、通常はAppDelegete .Swiftでこの設定が行われる

- UIViewのレイアウトサイクルは、制約の更新、フレームの更新、レンダリングの3ステップで実施される
- UIViewControllerはviewの読み込み、表示、レイアウトを経て表示されるviewの読み込みが完了した時に呼ばれるviewDidloadでは初期化を表示の直前に呼ばれるviewWillApper()では、動的な値のセットを表示完了後に呼ばれるviewDidApper()ではview表示後の重い処理を実行するなど、最適なタイミングで、最適な処理を実行する必要がある
- UIwindowはUIViewのサブクラスでウィンドウの役割を担っている、そのためあるウィンドウの前に別のウィンドウを表示することで、アラートや、キーボードなどのUIで現在のUIのをカバーして表示できる

- UIStackViewを用いると、上や横から順番に積み重ねるレイアウトの作成を簡単に実施できる
これは自動的に制約を付与するオブジェクトであり、動的で柔軟なレイアウトの助けとなる
