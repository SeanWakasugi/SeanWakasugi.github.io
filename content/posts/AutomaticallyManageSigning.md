---
title: "Automatically manage signingのススメ"
date: 2024-03-07
categories: ["Xcode", "iOS"]
tags: ["Xcode"]
---

Xcode開発でSigning周りは本当に面倒ですが、Automatically manage signingは全てを解決してくれます。

一部参考にした記事:
[Xcode14時代の証明書管理のベストプラクティス #iOS - Qiita](https://qiita.com/ykyouhei/items/af728232ce1950b2c2de)

小さい開発チームが作成するアプリは全てAutomatically manage signingに良いと考えています。

## 新しいアプリを作るとき

超簡単です。もう [https://developer.apple.com/](https://developer.apple.com/) を開く必要はありません。

1. Automatically manage signingチェックボックスをオンする
2. TeamとBundle Identifierを設定する

これで終わりです。

面倒な

* AppIDの設定
* Provisioning Profileの作成
* Provisioning Profileにメンバー、端末の割り当て

全部要りません。

## 既存のアプリをAutomatically manage signingに変更するとき

変更する場合は少し面倒です。

1. Automatically manage signingチェックボックスをオンする
2. TeamとBundle Identifierを設定する

ここまでで自動管理になるように見えますが、少し落とし穴があります。
キャッシュがある限り、既存の手動作成のProvisioning Profileを使用していることがあることです。
そこで、キャッシュを消します。

> ・SigningのProvisioning profile Xcode Managed Profile の右にある i をクリックします
>
> ・Prov アイコンをデスクトップにドラッグ＆ドロップする等してファイル名を確認します
>
> ・cd ~/Library/MobileDevice/Provisioning\ Profilesでプロビジョニングファイルがあるディレクトリに移動します
>
> ・rm で先ほど調べたファイル名のファイルを削除します
> 
> ・Xcode Managed Profile の右にある i をクリックし、Provisioning profile の作成日が正しいことを確認します

引用元(一部抜粋、修正)
[Provisioning profileにデバイスを追加したのに XCode で反映されない｜conocode](https://conocode.com/troubleshooting/provisioning-profile-reload/)

また、手動作成したProvisioning Profileも不要になりますので、チームにAutomatically manage signingに切り替えることを共有の上 [https://developer.apple.com/](https://developer.apple.com/) から削除しましょう。

## 本番証明書について

今までは本番証明書をローカルで作成し、Xcodeに登録する必要がありましたが、
Automatically manage signingを使用すると、本番証明書も自動で管理されます。
正確には、Cloud Signingと呼ばれるクラウド上での認証に切り替わります。

本番ビルドが作成できるのは `クラウド管理配布証明書へのアクセス` があるメンバーのみです。(DeveloperでもOK)
そのため、ビルドの作成メンバーを絞ることは今まで通り可能です。
[Apple Developer Programにおける役割](https://developer.apple.com/jp/support/roles/)

## デメリット

### 開発Certificateが複数つくられてしまうことがある

Automatically manage signingを使用すると、すでにCertificateがあるのにも関わらずデベロッパーメンバーごとのCertificateが自動で生成されることがあります。
今後新規作成する場合は自動生成に任せて、手動で作成したりしないのが良いと思います。
既存のTeamで自動生成されてしまった場合は、手動で作成した方を削除した方がいいとは思います。ただ、基本的に自動管理になるように進んでいるので、大抵の場合は気にしなくても大丈夫だと思います。
