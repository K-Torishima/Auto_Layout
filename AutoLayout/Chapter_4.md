# StoryboardとAuto Layout

このChapterは主にメモ的なことを書く

StoryboardをCodeで変更したい場合

- AppDelegateで記載数

``` swift

class AppDelegate: UIResponder, UIApplicationDelegate {

    func application(application: UIApplication, didFinishLaunchingWithOptions launchOptions: [NSObject: AnyObject]?) -> Bool {
        let window = UIWindow()

        // 対象のストーリーボードを参照する
        let storyboard = UIStoryboard(name: "Main", bundle: NSBundle.mainBundle())
        // Initial View Controller に設定されたVCを参照
        let firstViewController = storyboard.instantiateInitialViewController()
        // rootViewControllerに上の行で取得したVCを設定
        window.rootViewController = firstViewController
        // キーウィンドウに設定
        window.makeKeyAndVisible()
        
        return true
    }
}

```

## Interfase BuilderでAuto Layoutを有効にできる

- use Auto Layoutというボタンがある

[FileinspecterからAutoLayoutを有効にできる](image/4-4.png)

## 制約編集ボタンを用いて制約を追加する

[制約ボタン](image/4-5.png)

[追加可能な制約表](image/4-6.png)

### spacing to nearest neighbor

- 選択したViewのLeading, trailing, top, Bottomとキャンパス上で近いView、もしくはスーパービューとの間制約を設定することができる
- 定数を入力する入力欄の左側の[▼]をクリックすると、選択項目が表示される
- 他のコンポーネントとある一定の距離に近づくと、吸い付く距離
- 最も近いViewだけでなく、スーパービューも選択可能

定数にはMultipleを表示されるが、値を指定しなければ、現在キャンパス表示されている値が最も近いビューとの間に設定される
この方法を用いると、より効率的に制約を指定することができる

[▼をクリックすると、制約の設定を変更できる](image/4-7.png)

### width Height

width Height はViewの幅と高さを設定する、UILabelやUIButtonの幅の高さを設定すると、コンテンツに影響されず大きさが固定される
そのためIntrinsic Content Size を使う必要がない、もしくは使えない場合に用いる

### Equal Widths, Equal Heights

- 選択したViewどうしの幅または高さを等しくする制約を与えてくれる
- 等しい幅や高さを与えたビューコンポーネントに対して、別途大きさの制約を与えていない場合は、その一つのうち、Intrinsic Content Size が大きい方の値が優先される
- 通常Content　Hugging Priority（大きくなりにくさ）よりもCompression Resistance Priority（小さくなりにくさ）の方が、高いのでCompression Resistance Priorityが優先されるため

### Aspect Ratio

選択したビューが現在キャンパス上で表示されている縦横比を保つように制約を与える
これは、幅と、高さに対して、Multiplierを設定した制約を与えることで実現する
描画領域の大きさに合わせて、表示画像の大きさを変えたい時などに有効である

### Align

Aligin は　選択した2つのViewのLeading, Trailing,　Top, Bottom, x方向, Y方向, 中央, ベースライン　のどれかを同じ値にする
制約ボタン上のAlign と同様の機能ですが、定数を設定することはできない

[Aliginの表](image/4-8.png)

それぞれの説明は実際に使って確認する

### Resolve Auto Layout Issues パネル

- レイアウトに関する問題を解決するときに使う
- 各ボタンの説明
  - Update Fremes 　- 制定した制約を元にキャンバス上のフレームを更新する
  - Update Constraints - キャンパス上のフレームを元に、既に設定された制約を更新する
  - Add Missing Constraints - 十分な制約が与えられないViewに対して、足りない制約を自動的に付与する、制約不足エラーが出ているビューに対して用いることができる
  - Reset to Suggested Constraints　- 既に与えられている制約情報を削除し、キャンパス上の位置情報に基づいた制約を与える、便利な機能だが、複雑なビューになると、意図した通りの制約が与えられないことが多いので注意が必要
  - Clear Contraints　- 全ての制約を削除する

### stackボタン

選択されたビューをUIStackView上にまとめる

## Control　＋　ドラッグで制約を追加する

[詳細図](image/4-9.png)

### 一つのViewを選択した時

[詳細図](image/4-10.png)

### 同じ階層のViewを選択した時

[詳細図](image/4-11.png)

[制約一覧](image/4-12.png)

### 親子関係にあるビューを選択した時

[詳細図](image/4-13.png)

## インターフェースビルダー上での制約表示

### 通常の制約
