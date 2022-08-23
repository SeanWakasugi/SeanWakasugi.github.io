---
title: "リーダブルコード10章感想"
date: 2022-08-23
categories: ["本まとめ"]
tags: ["リーダブルコード", "swift"]
---

## 10章: 無関係な下位問題を抽出する

プロジェクト固有のコードから汎用コードを抽出しようという話です。

例えば、このような計算(プロジェクト固有)をする関数の中に、  
JSONファイル読み込み(汎用コード)の処理が入っていたら、  
それを抽出して別の関数にしてしまおうということです。

```swift
func calculateJSON() -> Int? {
    // JSONファイル読み込み
    guard let url = Bundle.main.url(forResource: "FileName", withExtension: "json") else {
        showError()
        return nil
    }
    guard let data = try? Data(contentsOf: url) else {
        showError()
        return nil
    }
    let decoder = JSONDecoder()
    guard let myData = try? decoder.decode(MyData.self, from: data) else {
        showError()
        return nil
    }
    // 計算
    let sum = myData.firstNumber + myData.secondNumber + myData.thirdNumber
    if let multiplier = myData.multiplier {
        return sum * myData.multiplier
    }
    return sum
}
```
抽出した場合
```swift
func calculateJSON() -> Int? {
    // JSONファイル読み込み
    let myData = loadJSON()
    // 計算
    let sum = myData.firstNumber + myData.secondNumber + myData.thirdNumber
    if let multiplier = myData.multiplier {
        return sum * myData.multiplier
    }
    return sum
}

func loadJSON() -> MyData? {
    // JSONファイル読み込み
    guard let url = Bundle.main.url(forResource: "FileName", withExtension: "json") else {
        showError()
        return nil
    }
    guard let data = try? Data(contentsOf: url) else {
        showError()
        return nil
    }
    let decoder = JSONDecoder()
    guard let myData = try? decoder.decode(MyData.self, from: data) else {
        showError()
        return nil
    }
    return myData
}
```

抽出するといいことが3つあります。

1. 読みやすい - 関数のメインの目的に集中できる
2. 再利用可能 - 汎用的な内容をプロジェクトを超えて使いまわせる
3. 改善が楽 - 他の部分を気にせず一括で改善できる

私もここまでは理解できていました。読みにくくなってきたら関数を抽出するべきだし、抽出したものは再利用できるように汎用的にするべきだと考えてコードを書いています。

私が1つ課題としていたことは、「どこを抽出するのか」です。ここの部分を詳しく読み解いてまとめていきます。

### どこを抽出するのか

「無関係な下位問題」を抽出します。

「無関係」とは、自己完結している、他の部分に依存しない処理のことです。

無関係な処理は、再利用可能で汎用的なものです（実際複数箇所で使うかはともかく）。

「下位問題」を見極める方法について、作者は以下のように書いています。

>1. 関数やコードブロックを見て「このコードの高レベルの目標は何か?」と自問する。
>
>2. コードの各行に対して「高レベルの目標に直接的に効果があるのか? あるいは、無関係の下位問題を解決しているのか?」と自問する。
>
>3. 無関係の下位問題を解決しているコードが相当量あれば、それらを抽出して別の関数にする。 

つまり、高レベルの目標に直接的に効果がない部分は、高レベルではなく「下位」の問題であるというわけです。

「高レベルの目標に直接的に効果がなく、自己完結している処理」が抽出する基準になりそうです。

### 抽出してどこに置くのか

ここからは自分用のメモです。

どこに置くかの基準は、「汎用性」だと考えています。

汎用性を高めたければスコープを広く、どこからでもアクセスできるようにしたくなりますが、スコープが広すぎてどこからでも使えるようになると、読む時に把握しにくく、意図せず呼び出してしまったりと問題があります。

汎用性に応じて、適切なアクセスコントロールすることが必要です。

プロジェクト全体から利用しうるならユーティリティメソッドとしてまとめておいたり、

特定のクラスのみでしか使わないなら、クラス内メソッドでいいはずですし、

逆にいろんなアプリで使い回すなら、自分でライブラリを作成して分離するのも挑戦したいです。

Swiftなら、特定の型に依存した関数は`extension`などで（既存でも自作でも）型内のメソッドとして利用するのが有用だと感じました。

以上