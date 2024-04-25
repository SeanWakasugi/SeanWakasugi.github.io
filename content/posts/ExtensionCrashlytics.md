---
title: "ExtensionでCrashlyticsを利用"
date: 2024-04-25
categories: ["Swift"]
tags: ["Crashlytics", "Swift", "Firebase"]
---

Widget(ウィジェット)やShare(共有)などのApp Extensionや、VPNなどのNetwork Extensionなど、Extensionでもクラッシュやエラーの追跡は重要です。
ExtensionでのCrashlyticsの設定方法をまとめました。

なお、Firebase公式はApp Extension等でのFirebaseの動作をサポートしていません。

[Add Firebase to iOS app extension · Issue #6605 · firebase/firebase-ios-sdk · GitHub](https://github.com/firebase/firebase-ios-sdk/issues/6605)

ただ、上記Issueの中で開発メンバーは

> Crashlytics should just work.

と述べており、Crashlytics程度の利用であれば問題ないと考えられます。(実際私が導入したアプリでもクラッシュやエラーの送信は行えています)

## Firebaseコンソールのセットアップ

Extensionをアプリ本体とは別のアプリとして登録する必要があります。

Firebaseコンソールで「プロジェクトの設定」→「アプリを追加」と進み、「AppleバンドルID」の欄にExtensionのバンドルIDを入れます。

その後生成される `GoogleService-Info.plist` をExtensionのフォルダ下に置いてください。

公式ドキュメント:
[Firebase を Apple プロジェクトに追加する  |  Firebase for Apple platforms](https://firebase.google.com/docs/ios/setup?hl=ja)

試してはいませんが、複数のExtensionでも同じように追加できそうです。

## Firebaseライブラリのインストール

Firebase CrashlyticsとAnalyticsをCocoaPodsや、Swift PMを利用してインストールしてください。

通常、アプリ本体のターゲットのみに追加されるので、Extensionにも追加されるようにしてください。

Swift PMの場合は、該当Extensionの「General」の「Frameworks and Libraries」からFirebase CrashlyticsとAnalyticsを追加します。

## 開発/本番の動的なGoogleService-Info.plistの使い分け

開発と本番で別のFirebaseプロジェクトに向ける場合は、ビルドのConfigurationに応じてファイルを置き換える処理が必要です。

Extensionのフォルダのルート下の `Config/` に `GoogleServiceProd.plist` と `GoogleServiceDev.plist` を置き、ExtensionのBuild Phasesから新たなRun Scriptを追加してください。入力ファイル、出力ファイルの設定は不要です。

```bash
if [ "${CONFIGURATION}" == "Release" ]; then
  cp "${PROJECT_DIR}/${TARGET_NAME}/Config/GoogleServiceProd.plist" "${BUILT_PRODUCTS_DIR}/${WRAPPER_NAME}/GoogleService-Info.plist"
else
  cp "${PROJECT_DIR}/${TARGET_NAME}/Config/GoogleServiceDev.plist" "${BUILT_PRODUCTS_DIR}/${WRAPPER_NAME}/GoogleService-Info.plist"
fi
```

このスクリプトが行っている動作は、Extension動作時にFirebaseライブラリは、Extensionの直下の `GoogleService-Info.plist` というファイルを読みにいきます。

このスクリプトは、ビルドされたプロダクトの位置 `${BUILT_PRODUCTS_DIR}` の TARGETとなるExtensionファイル `${WRAPPER_NAME}` の直下に `GoogleService-Info.plist` という名前でコピーしています。

`${WRAPPER_NAME}` は動作時に `{Extensionの名前}.appex` のような名前として解決されます。

このコードはExtensionに限らず、アプリ本体でも同じコードが使えます。

アプリ本体の場合は `${WRAPPER_NAME}` が `{Appの名前}.app` というような名前に置き換わるので、正常な位置にコピーが行われます。

## CrashlyticsサーバにdSYMファイルを送信するスクリプト

[Firebase Crashlytics を使ってみる](https://firebase.google.com/docs/crashlytics/get-started?hl=ja&platform=ios#set-up-dsym-uploading)

チュートリアルと完全に同じScriptで大丈夫です。
TARGETに応じてビルド時に解決されるようなコードになっているので、ExtensionのdSYMも正常に送信してくれます。

## Firebaseの初期化とエラーの送信

Firebase Crashlyticsを有効化するには、どこかで `FirebaseApp.configure()` を呼ばなくてはなりません。Extensionで使うクラスの `init()` など、Extensionが実行される前に一度だけ必ず実行される場所に書いておくことが推奨されます。

クラッシュ時は自動的にエラーが送信されますが、それ以外の非致命的なエラーを送りたい場合は、`Crashlytics.crashlytics().record(error: Error)` が便利です。

アプリと合わせて動くタイプのExtensionの場合は、なんらかの識別できるIDをセットしておくと `Crashlytics.crashlytics().setUserID(String)` アプリのエラーとの紐付けを行いやすいかもしれません。
