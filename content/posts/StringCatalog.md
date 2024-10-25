---
title: "ローカライズなんて全く考えずに作ったiOSアプリをローカライズする方法"
date: 2024-08-05
categories: ["Swift"]
tags: ["Xcode", "swift"]
---

ローカライズを全く考慮せずに開発したiOSアプリを、後からローカライズ対応する方法について説明します。すべてのテキストがハードコードされている状態から、テキストをローカライズファイルに分離し、複数言語に対応させるまでの手順をまとめます。

参考公式動画: [String Catalogsの紹介 - WWDC23](https://developer.apple.com/jp/videos/play/wwdc2023/10155/)

## 結論: 対応手順まとめ

### ステップ1: 使用したい言語を追加する

1. プロジェクトファイルのInfoタブを選択。
2. Localizationsセクションの+ボタンをクリックし、言語を追加。
3. 言語をクリックし、下部の「Set Default」ボタンを押してデフォルト言語を設定。

### ステップ2: String Catalogファイルを追加する

1. Xcodeで「File > New > File」を選択。
2. 「String Catalog」を選択。
3. `Localizable.xcstrings` をターゲットのルートに作成。
4. Build Settingsで `Use Compiler to Extract Swift Strings` を有効にする。

### ステップ3: String Catalogに抽出されるようコードを編集

* String Catalogはビルド時にローカライズ可能と認識される文字列を自動で抽出してくれる。
  * SwiftUIは、UIテキストをローカライズのKeyとして使う文字列に置き換える。(ハードコードされたテキストをそのままKeyとして使う場合は対応不要)
  * SwiftUI以外のUIテキストを `String(localized: )` に置き換える。iOS15未満をサポートする場合は `NSLocalizedString( , comment: )`。
  * StoryboardやNIBファイルは右のファイル設定から `Localize...` をクリックしてファイル専用のString Catalogを生成する。
  * Info.plistは `InfoPlist.xcstrings` をInfo.plistと同じディレクトリに置く。
* 数値や日付などを自動でローカライズされるよう変更する 参考: [iOSアプリの国際化対応の勘所とTips集](https://qiita.com/mono0926/items/c41c1ce18b90b765a8f2)

### ステップ4: String Catalogに翻訳を追加

* String CatalogファイルはGUIで簡単に編集できる。
* 翻訳を外部の翻訳者やツールに任せる場合は、Product > Export Localizationから `.xcloc` のXcode独自形式で書き出す。
  * `.xcloc` ファイル内には翻訳でよく使われる `XLIFF` 形式が存在するので、翻訳ツールによってはこれを使う。
  * 翻訳した `.xcloc` ファイルはProduct > Import Localizationから取り込める。

## String Catalogについて

String Catalogは、従来分割されていた各言語のstringsファイル、strings dictionaryに変わる包括的な仕組みです。

iOSバージョンに依存せず使用でき、既存のローカライズファイルからの変換も容易なので、Xcode 15以降であればString Catalogを使うことが推奨されます。

### String Catalogの特徴

* 複数言語の翻訳を1つのファイルで管理できる
* ビルド時に自動的にローカライズ可能な文字列を抽出してくれる
  * コード側をすべてローカライズ可能な文字列に対応させることで、String Catalogに管理を任せることができる
* 翻訳の進捗状況を視覚的に確認可能
* 複数のString Catalogファイルを使い分けることができる

### String Catalogの作成方法

1. Xcodeで「File > New > File」を選択。
2. String Catalogを選択。
3. コードから自動抽出するString Catalogは、 `Localizable.xcstrings` という名前でターゲットのルートに作成。
4. Build Settingsで `Use Compiler to Extract Swift Strings` を有効にする。

### String Catalogの文字列管理

Build Settingsで `Use Compiler to Extract Swift Strings` を有効にすると、ビルド時にコード等からローカライズ可能な文字列の抽出が行われ、自動で新規作成、更新、削除されます。
ローカライズが入力されているのにコード等から削除されると、STALE状態となり、削除を承認できます。

抽出と別に任意のKeyで作成することもできますが、自動管理から外れるので推奨されません。

すべてのUIテキストをコード側、Interface BuilderとInfo.plist側でローカライズ可能と設定し、String Catalogに文字列管理を任せるのが便利です。

## SwiftUIのローカライズ

```swift
// Text.init(_ key: LocalizedStringKey, tableName: String? = nil, bundle: Bundle? = nil, comment: StaticString? = nil)
Text("Hello, World!")
```

`Text` など、SwiftUIの標準的なViewは自動的にString Catalogに抽出されます。

これは、SwiftUIでは一般に引数に `String` ではなく `LocalizeStringKey` を受け取るようすることで、ローカライズ可能な文字列として認識されるためです。

つまり、これらの引数はローカライズにおけるKeyおよびデフォルトの値として認識されます。
Keyとデフォルトの値が同一の場合は問題ないですが、同じ文字列で別の翻訳をつけるケースなども多いと思われるので、Keyとデフォルトの値は別にすることが推奨されます。

そのため、元々SwiftUIに文字列をハードコードしているケースでは、ハードコードした文字列をKeyとして使う文字列に置き換えていく作業が必要になります。

なお、自分でカスタムViewを作成する際も `String` の代わりに `LocalizedStringKey` を使用することで、自動的にローカライズ対象となります。

```swift
struct CustomView: View {
    let title: LocalizedStringKey

    var body: some View {
        Text(title)
    }
}
```

## UIKitのローカライズ

SwiftUIと違い、UIに使うテキストは明示的にローカライズ可能であるように書く必要があります。
ローカライズ可能であるように書くことで、自動的にString Catalogに抽出されます。

iOS15以降のみのサポートで良ければ、`String(localized: )` を使うのが良いです。

```swift
// String(
//     localized keyAndValue: String.LocalizationValue,
//     table: String? = nil,
//     bundle: Bundle? = nil,
//     locale: Locale = .current,
//     comment: StaticString? = nil
// )
let message = String(localized: "こんにちは")
```

iOS15未満をサポートする場合は `NSLocalizedString( , comment: )` を使う必要があります。

```swift
// NSLocalizedString(
//     _ key: String,
//     tableName: String? = nil,
//     bundle: Bundle = Bundle.main,
//     value: String = "",
//     comment: String
// )
let message = NSLocalizedString("こんにちは", comment: "ユーザーへの昼の挨拶")
```

`String(localized: )` が優れているのは、文字列中に変数が入る場合です。

```swift
// 正しく抽出される: `エラー%@` (%がつくのは変数)
let errorMessage = String(localized: "エラー: \(error.localizedDescription)")
// 変数部分が文字列として抽出されてしまう: `エラー(error.localizedDescription)`
let nsErrorMessage = NSLocalizedString("エラー\(error.localizedDescription)",comment: "エラーとその内容")
// 正しく抽出可能: Keyが `エラー` 翻訳先に `エラー%@` と入力できる。
let formattedErrorMessage = String(format: NSLocalizedString("エラー", comment: "エラーとその内容"), error.localizedDescription)
```

`String(localized: )` を使えば、大きく書き換えることなしに文字列中の変数を利用できます。

参考: [String Catalogで、%を使うやつまとめ](https://zenn.dev/samekard_dev/articles/e0149efe77e403)

なお、Objective-Cでは `NSLocalizableString` 、Cでも `CFCopyLocalizedString` を使用すればString Catalogが自動で抽出してくれます。
これらはビルド設定の `Localized String Macro Names` で設定できます。

## Interface BuilderとInfo.plistのローカライズ

Interface BuilderやInfo.plistはコードから抽出、管理される `Localizable.xcstrings` とは別管理になります。

Interface Builder(StoryboardやNIBファイル)は、右ペインのファイル設定から `Localize...` でそのファイル専用のString Catalogを生成できます。

つまり、Interface Builderのファイルごとに別々のString Catalogファイルが生成されます。大量のカスタムViewや、ViewControllerごとにInterface Builderを作成するようなコード方針だと大量にファイルができてしまい管理が少し不便です……。

Info.plistは `InfoPlist.xcstrings` をInfo.plistと同じディレクトリに置くことで、ビルド時にInfo.plistの内容が反映されたString Catalogになります。

Info.plistの中でも、ユーザーに表示されるテキストのみがローカライズが必要なテキストとして認識され、String Catalogの対象になります。

例えば、ホーム画面等に表示されるアプリ名である `CFBundleDisplayName` や、ユーザーに権限の許可を求めるダイアログに表示される `NSPhotoLibraryUsageDescription` などです。

## ローカライズのエクスポートとインポート

String CatalogファイルはGUIで簡単に編集できます。文章の少ないケースや、翻訳者がXcodeを使用できる場合は直接String Catalogに書き込んでいくのが簡単です。

翻訳を外部の翻訳者やツールに任せる場合は、Product > Export Localizationから `.xcloc` のXcode独自形式で書き出すことができます。

プロジェクト内もしくはターゲット内のすべてのString Catalogを認識して、まとめてくれます。

`.xcloc` は、Xcodeや一部対応ツールで開ける翻訳用のファイル形式です。言語ごとにファイルを分けたりすることもできます。

`.xcloc` ファイル内にはより汎用的で翻訳でよく使われる `XLIFF` 形式が存在するので、翻訳ツールによってはこれを使うこともできます。

翻訳した `XLIFF` を `.xcloc` のファイル内に戻すことで翻訳した `.xcloc` ファイルとして扱えます。

翻訳した `.xcloc` はProduct > Import Localizationから取り込めます。
