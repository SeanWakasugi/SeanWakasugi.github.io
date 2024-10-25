---
title: "iOSアプリをローカライズしてつまづいたところメモ"
date: 2024-10-25
categories: ["Swift"]
tags: ["Xcode", "swift"]
---

以前書いた「ローカライズなんて全く考えずに作ったiOSアプリをローカライズする方法」に従って、実際にローカライズしていないアプリをローカライズした時につまづいた点や検討が必要だった点についてメモ的に残していきます。

## ローカライズテキストの抽出と適用

### Storyboard

iOSでは、Storyboardを使用したUIは、ほとんどのテキスト要素が自動的にString Catalogに抽出されます。しかし、以下の項目に関しては、コードから設定する必要があります。

#### UITextViewのtext

ローカライズ対象として抽出はされますが、ローカライズしたテキストを適用してくれません(バグ?)。コードからローカライズしたテキストを書き込む必要があります。

[ios - Localization for UITextView in storyboard - Stack Overflow](https://stackoverflow.com/questions/20038048/localization-for-uitextview-in-storyboard)

#### AttributedStringを利用したUILabel等

AttributedStringは自動でローカライズに抽出されず、適用もされません。ある単語を赤くハイライトするなど、元の言語を前提にAttributedStringを作成しているケースも多いと思われるのでローカライズすることが難しいです。
基本的にはAttributedStringをやめて普通のStringの組み合わせで設定できないか考えましょう。
どうしてもAttributedStringを使う必要があるのであれば、ローカライズしたテキストを必要に応じてAttributedStringに変換して、コード側から適用する必要があります。

### テキストの長さ

日本語は他の言語より短い文字列で表現できる傾向があります。日本語だと数文字のところが英語にするとかなりの長さになることが多々あります。
固定長のラベルや一行指定のラベルを可変で2行以上を許容するように修正する必要があります。Storyboard上では確認し切れないので、実機で見切れがないか探す作業が必要です。


## String Catalogをexportしたファイルの取り扱い

iOSのString Catalogをexportした `ja.xliff` ファイルからCSVファイルを作って、翻訳後にそれぞれのファイルに書き戻すようにスクリプトを組みました。
xliffファイルは比較的新しいこともあり、取り扱うライブラリなども見当たらなかったので、xmlファイルとしてpythonで処理するスクリプトを組みました。

### iOSのxliffファイルの読み出し

iOSの `ja.xliff` ファイルからデータを抜き出すコードです。
正確には、keyはアプリ全体でユニークとはかぎらないので重複するkeyがある場合はxml tree構造を読み取って別で管理するなどが必要ですが、省略しています。

```python
import xml.etree.ElementTree as ET

localizations_folder_path = "path/to/localizations_folder"

ja_xliff_path = localizations_folder_path / "ja.xcloc/Localized Contents/ja.xliff"
tree = ET.parse(ja_xliff_path)
root = tree.getroot()

# 空のリストを準備
data = []

# 各trans-unitを処理してデータをリストに格納
for trans_unit in root.findall(".//ns:trans-unit", {"ns": "urn:oasis:names:tc:xliff:document:1.2"}):
    unit_id = trans_unit.get("id")
    source_text = trans_unit.find("ns:source", {"ns": "urn:oasis:names:tc:xliff:document:1.2"}).text
    if unit_id in ("CFBundleName", "CFBundleDisplayName"):
        # 今回は翻訳不要だったためスキップ
        continue
    # リストに辞書としてデータを追加
    data.append({"key": unit_id, "ja": source_text})
```

### iOSのxliffファイルへの書き込み

翻訳後に各言語のxliffファイルに書き込むコードです。

```python
# 名前空間のプレフィックスを制御(設定しないと書き込み後のタグに不要なprefixがつく)
ET.register_namespace("", "urn:oasis:names:tc:xliff:document:1.2")

# 今回は3言語。必要に応じて変更。
target_languages = ["ja", "en", "ko"]
# dict[言語名, dict[key, 翻訳後テキスト]]の形で読み込んでいる。
translations: dict[str, dict[str, str]] = self._load_translations_from_csv(csv_file_path)

for lang in target_languages:
    xliff_path = localizations_folder_path / f"{lang}.xcloc/Localized Contents/{lang}.xliff"
    tree = ET.parse(xliff_path)
    root = tree.getroot()

    for trans_unit in root.findall(".//ns:trans-unit", self.namespace):
        unit_id = trans_unit.get("id")
        if unit_id in translations:
            target_elem = trans_unit.find("ns:target", self.namespace)
            if target_elem is None:
                # 一度も翻訳したことない場合はtargetタグがないので作る
                target_elem = ET.SubElement(trans_unit, "target")
            if "state" in target_elem.attrib:
                # 元のxliffファイルにはstateにnewとか入っていて、翻訳状態を管理している。書き込むときは消す必要がある。。
                del target_elem.attrib["state"]

            # 翻訳を書き込む
            target_elem.text = translations[unit_id].get(lang, "")

    # 更新されたXLIFFファイルを保存
    tree.write(xliff_path, encoding="utf-8", xml_declaration=True)
```

## Android翻訳ファイルとの統合時の注意

同じアプリのiOSバージョンとAndroidバージョンがある場合に翻訳を統一したいことがあると思います。
Anroidの `strings.xml` ファイルとiOSのString Catalogをexportした `ja.xliff` ファイルからCSVファイルを作って、翻訳後にそれぞれのファイルに書き戻すようにスクリプトを組んでいました。
CSVファイルで統合する際に困ったことを書いておきます。

### 改行

Androidでは改行コード`"\n"`を用いて改行を表しますが、iOSでは改行コード`"\n"`は改行と認識されません。実際の改行に変換してからxliffファイルに書き戻す必要があります。

### ワイルドカードの指定方法

AndroidとiOSでは、文字列の変数置き換えに使われるワイルドカードの形式が異なります。

- Android: 文字列`%s`, 数値`%d`
- iOS: 文字列`%@`, 数値`%d`

他にも文字列が複数ある場合などありますが、必要に応じて変換か、iOSとAndroidで文字列を分けてしまう必要があります。

### HTMLタグ

Androidでは簡単なHTMLタグをテキストに挿入してスタイリングすることができますが、iOSではこの機能は利用できません。
HTMLタグを頻繁に使う運用ではなかったので、一部HTMLタグの含まれるテキストのみiOSとAndroidで文字列を分けました。


## iOSにおけるアプリの言語設定

iOSでは、端末の言語設定に基づいてアプリの表示言語が自動的に切り替わる仕組みがあります。

### 端末の言語設定の反映

端末の言語設定で、自動的にアプリの言語設定が決定されます。
言語設定は複数設定できるので、その場合も少し挙動が変わります。

- 端末: 英語　→　アプリ: 英語
- 端末: ニュージーランド英語　→　アプリ: 英語(亜種的な言語でも、その言語の設定になる)
- 端末: イタリア語(アプリが対応していない言語)　→　アプリ: 日本語(アプリのデフォルト言語)
- 端末: イタリア語, 英語　→　アプリ: 英語(1番上の言語設定でなくても、対応言語が言語設定に入っていればその言語になる)

### アプリ内の言語変更について

端末の言語設定と別に、アプリの言語設定を変えることもできます。ただし、iOSではアプリ内から直接言語設定を変更することはできません。

言語設定を変更したい場合は、設定アプリの各アプリの設定から `Preferred Language` を切り替える必要があります。
アプリ内から以下リンクで、設定アプリの各アプリ設定に飛ばすのが良いです。

```swift
UIApplication.shared.open(URL(string: UIApplication.openSettingsURLString)!)
```

### アプリ内の言語変更が表示されないケース

端末の言語設定が1つの場合、なぜか設定にアプリの言語設定が表示されません。

iOS17以降であれば、端末の言語設定が1つの場合でも、info.plistで`UIPrefersShowingLanguageSettings`を`YES`に設定すると、アプリの言語設定が表示されます。

[Localization settings hidden if on… | Apple Developer Forums](https://forums.developer.apple.com/forums/thread/721302)

`UIPrefersShowingLanguageSettings`については、なぜかドキュメントがなくWWDC2024の動画で一言触れられたのみです。

[多言語対応アプリの構築 - WWDC24 - ビデオ - Apple Developer](https://developer.apple.com/jp/videos/play/wwdc2024/10185/)

iOS17未満の端末で端末の言語設定が1つの場合は、アプリ内の言語変更を行う方法はありません……。


以上
