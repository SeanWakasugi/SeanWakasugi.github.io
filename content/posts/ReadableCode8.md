---
title: "リーダブルコード8章感想"
date: 2022-08-09
categories: ["本まとめ"]
tags: ["リーダブルコード", "swift"]
---

## 8章: 巨大な式を分割する

式がでかいと読みにくいので分割して短くしようというお話です。

### 説明変数・要約関数
こうではなくて、

```swift
if UserDefaultManager.shared.token() != nil {
    // 処理
}
```
こういうふうに書いて式を分割しようという話です。

```swift
let IsTokenAssigned = UserDefaultManager.shared.token() != nil)

if IsTokenAssigned {
    // 処理
}
```

コードが横に長くなるのが嫌でよく使っている方法でしたが、認識していないメリットがたくさんありました。

一つ目は、変数に置き換えることで何をしている式なのか説明できることです。

`IsTokenAssigned`という名前をつけることで、変数のヌルチェックをしているという情報だけではなく、

このトークンは割り当てられる(Assigned)もので、それが既に割り当てられているかどうかを調べている。

ということまで推測できます。

```swift
let IsTokenAssigned = (UserDefaultManager.shared.token() != nil)
```

2つ目のメリットは、コードの主要な概念を事前に伝えることができることです。

例えば、先ほどのtokenの有無によって画面の設定を行うメソッドがいくつかあるとします。

これでは読んで何をしているのか理解するのに時間をかかります。

```swift
func configButton() {
    // tokenがnilでないならボタンを表示
    if UserDefaultManager.shared.token() != nil {
        button.isHidden = false
    }
}

func configTextLabel() {
    // tokenがnilならエラーと表示
    textField.Label = UserDefaultManager.shared.token() ? "成功" : "エラー"
}
```

式を変数にして名前につけ、コードの最初に定義しておきます。

こうすることで、最初の変数を読んだ時点で、「このコードではトークンをチェックしてこれによって処理が分岐するのだな」と予想できます。

文字数自体は減っていませんが、事前に重要な概念を伝えることで理解しやすくなるのです。

```swift
// トークンがあれば成功画面、なければエラー画面を表示
let IsTokenAssigned = UserDefaultManager.shared.token() != nil)

func configButton() {
    if IsTokenAssigned {
        button.isHidden = false
    }
}

func configTextLabel() {
    textField.Label = IsTokenAssigned ? "成功" : "エラー"
}
```


### 短絡評価

私が全く意識していなかった内容です。

短絡評価とは、

複数の条件文が`&&`や`||`で結ばれている時、内部処理としては左から評価している。

この時、左の評価だけで必要十分であれば左の評価だけで式全体の評価を終了してしまう。

というものです。例えば、

`a && b`で`a = false`ならb関係なく`false`とか

`a || b`で`a = true`ならb関係なく`true`とか

この処理を利用して、例えば`a && b`の式はaが真だった場合のみbを確認するような処理を書くことができますが、分かりにくいのでやめよう、という話です。

例えば、こんな式です（多分コードエラーで動きませんが）。

aがnilでなかった場合しか右の式は確認されないので、aがnilでないことを前提にaの中身を調べる条件を書いています。

```swift
if a != nil && a.component == item {

}
```

書かないようにします。

### より優雅な方法を見つける

これはなるほど！　と感心した話です。

期間を扱う構造体があるとして、その期間が重複しているかどうかのメソッドを実装したいと考えているとします。

```swift
struct Range {
    let begin: Int
    let end: Int

    // 例えば、[0,5) は [3,8) と重なっている。
    func OverlapsWith() -> Bool {
        // ここはどう書けばいい？
    } 
}
```

このメソッドを、普通に考えるとかなり複雑になります。

```swift
func OverlapsWith(other: Range) -> Bool {
    // 'begin' または 'end' が 'other' のなかにあるかを確認する。
    return (self.begin >= other.begin && self.begin <= other.end) || (self.end >= other.begin && self.end <= other.end)
    // まだカバーできていない条件が無数にある
}
```

これを、逆転の発想で、重複していない条件を探してその逆を取ればいいと考えるとかなり簡単になります。

```swift
func OverlapsWith(other: Range) -> Bool {
    // 一方の終点が、この始点よりも前にある
    if other.end <= self.begin { return false }
    // 一方の始点が、この終点よりも後にある 
    if other.begin >= self.end { return false }
    // 残ったものは重なっている
    return true
}
```

### まとめ

この章で学んだこと
* 長い式を変数で置き換えることで、「何をしている式なのか説明できる」し、「コードで使う概念を事前に伝えられる」
* 短絡評価はなるべく使わない
* 複雑な条件式は、反対の条件を考えると簡単になることがある

お疲れ様でした。