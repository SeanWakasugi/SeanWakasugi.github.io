---
title: "iOSアプリのローカライズリリースを先送りにする時のチェックリスト"
date: 2024-12-12
categories: ["Swift"]
tags: ["Xcode", "swift"]
---

iOSアプリのローカライズ対応を進めたものの、翻訳確認などに時間がかかり、リリースを先送りにすることがあります。このとき、変更を長期間ブランチで塩漬けにすると、いざリリース直前になってからconflictが発生し、面倒なことになりがちです。

そこで、コードのマージだけでも先行して行う方法について、いくつか情報をまとめました。

ローカライズ対応のためにStringを別ファイルに切り出したり、言語・地域を考慮した実装を行うことは、今後のローカライズ対応においても有益な修正です。実際、現段階で多言語対応版をリリースしない場合でも、早めにmergeしておくことをお勧めします。

## String Catalog

ローカライズ対応を始める際、以下の手順で対応予定の言語を設定したと思います。

1. プロジェクトファイルのInfoタブを選択
2. Localizationsセクションの「+」ボタンで言語を追加
3. 言語をクリックし、下部の「Set Default」ボタンでデフォルト言語を設定

もし、今回はまだリリースしない言語があるなら、Localizationsセクションで「-」ボタンをクリックして削除しても問題ありません。String Catalogにはローカライズ情報が残るので、後で「+」ボタンから言語を追加すれば戻せます。ただし、デフォルト言語を変更するときは注意が必要です。

### デフォルト言語変更時の注意点

デフォルト言語を変えると、String Catalogファイル内の `"sourceLanguage"` が書き換わります。これはXcodeに任せて問題ありませんが、`"sourceLanguage"` の変更によってローカライズ値が消えることがあります。

`"state" : "new"` と `"extractionState" : "extracted_with_value"` 2つのフラグが付いているローカライズ値は、`sourceLanguage` 変更時に値が消えてしまうことがあります。

例えば、Storyboard上に「決定する」というUIButtonがあり、そのまま日本語が抽出されていた場合、デフォルト言語を英語にすると日本語の「決定する」は消失します。

これを避けるには、`"state" : "new"` をすべて `"state" : "translated"` に文字列置換し、1度ビルドすると `"extractionState" : "extracted_with_value"` も自動的に消えます。
その後はデフォルト言語を変更しても値が保持されます。

## 現在のローカライズ言語によって処理を変えるコード

String Catalogでまかなえない部分をコードで処理している場合、それらもローカライズに合わせた対応が必要です。結論としては、`Locale.current` に依存して処理を分けるようにすることをお勧めします。

`Locale.current` は、現在アプリで使用されているローカライズ言語の `Locale` を返します。ユーザーのiPhone言語が英語でもフランス語でも、アプリ表示が日本語なら `Locale` (の言語部分)は `ja` を返します。これにより、アプリ表示言語と処理が自然と同期するため、プロジェクトのLocalizations設定に依存してローカライズを管理できます。

例えば、以下のような `Language` enumを `Locale.current` で初期化すれば、enumごとに言語別の処理を書けます。
たとえプロジェクトが英語のみ対応でも、`Locale.current` が英語を返してくれるため、何も特別な対応なしで処理を振り分けられます。

```swift
enum Language: CaseIterable {
    case japanese
    case english
    case simplifiedChinese
    case traditionalChinese
    case korean

    init(locale: Locale) {
        self = Self.allCases.first { locale.identifier.hasPrefix($0.localePrefix) } ?? .english
    }

    var localePrefix: String {
        switch self {
        case .japanese:
            "ja"
        case .english:
            "en"
        case .simplifiedChinese:
            "zh-Hans"
        case .traditionalChinese:
            "zh-Hant"
        case .korean:
            "ko"
        }
    }
}
```


以上
