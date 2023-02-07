---
title: "CocoaPodsからSwiftPackageManagerに移行する方法"
date: 2022-02-03
categories: ["Swift"]
tags: ["Swift", "CocoaPods", "SwiftPM"]
---

CocoaPodsを利用していたプロジェクトをSwiftPMに移行したので手順のメモを残しておきます。

1. `pod deintegrate`
2. `~.xcodeproj`を開き、`podfile`を見ながら、同じライブラリをSwift Package Managerで追加
   ライブラリ名で検索してヒットしなくても、githubのURLを探して入力すれば対応しているライブラリが多かったです。
3. `podfile`、`podfile.lock`、`~.xcworkspace`、`Pods`フォルダを削除
4. Build Phasesの書き換え
   私の場合はFirebase Crashlyticsを利用していたので書き換える必要がありました。
   [Firebase公式](https://firebase.google.com/docs/crashlytics/get-started?hl=ja&platform=ios)の方法で対応しました。

## 対応していなかったライブラリ

有名なライブラリの中では、[LicensePlist](https://github.com/mono0926/LicensePlist)がSwiftPMに対応していませんでした。

代わりに[License**L**ist](https://github.com/cybozu/LicenseList)を導入しました。

シンプルで使いやすかったです。

License**L**istがLicense**P**listと違う点
* 全てのライブラリがSwiftPMで管理されている必要がある
* 設定アプリ内ではなく、アプリ内にライセンス情報を置くことになる
* カスタムでライセンス情報を追加できない。

## 移行してのメリットデメリット

メリット
* ブランチを移動しても`pod install`が必要ない
* Xcode Cloudのビルドが速い？(`brew install cocoapods`と`pod install`が必要ないから速いはず)
* プライベートなライブラリを作るのが楽。`Package.swift`を追加するだけ

デメリット
* 起動してから数秒、ライブラリの依存関係解決をしているためビルドができない

終わり