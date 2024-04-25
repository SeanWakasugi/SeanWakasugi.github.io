---
title: "iOSのTOTP(ワンタイムパスワード)まとめ"
date: 2024-04-25
categories: ["Swift"]
tags: ["ワンタイムパスワード", "Swift", "totp"]
---
TOTP(Time-based One-Time Password algorithm)とは、時間ベースで生成されるワンタイム(= 一回きりの)パスワードです。
iOSでは、Google AuthenticatorやMicrosoft Authenticatorなどのアプリがよく利用されています。
この記事では、iOSアプリでTOTPを利用する際の主な連携方法について説明します。
技術的な実装詳細は省略します。

## 結論

URL Scheme `otpauth` は認証アプリを直接開ける便利な方法ですが、サポートOSバージョンと、iOS標準の「パスワード」に対応するハードルがあります。

また、ユーザーが使いやすく利用するには、現状 `otpauth` 以外のQRコードやシークレットキーのコピペなどの方法を用意することが推奨されます。

## URL Scheme `otpauth`

以下の形式のURL Schemeを開くことで、TOTP対応アプリを開くことができます。

`otpauth://totp/Example:foo?secret=JBSWY3DPEHPK3PXP&issuer=Example`

書式詳細:
[Key Uri Format · google/google-authenticator Wiki · GitHub](https://github.com/google/google-authenticator/wiki/Key-Uri-Format)

Apple公式ドキュメント:
[Securing Logins with iCloud Keychain Verification Codes | Apple Developer Documentation](https://developer.apple.com/documentation/authenticationservices/securing_logins_with_icloud_keychain_verification_codes)

具体的にSwiftでは、以下のコードで開けます。

```swift
UIApplication.shared.open(URL(string: "otpauth://totp/Example:foo?secret=JBSWY3DPEHPK3PXP&issuer=Example")!)
```

ただし、動作はOSバージョンによって差があります。

## `otpauth` のOSバージョン別動作

### iOS15

(iOS15未満は検証していませんが、iOS15とも16以降とも動作が異なるようです)

`otpauth` を叩くと、必ずiOS標準の「パスワード」が開きます。

`otpauth` を利用してGoogle AuthenticatorなどのTOTPアプリを開くことはできません。
一般的なTOTPアプリをサポートするにはQRコードからの登録や、シークレットキーの入力による登録など、他の登録方法が必要です。

### iOS16~

(検証時点での最新OSはiOS17)

`otpauth` を叩くと、設定アプリ→パスワード→パスワードオプション→「次を使用して検証コードを設定:」で選択されているアプリが開きます。デフォルトだとiOS標準の「パスワード」になっています。
TOTPアプリを選択するUIを表示したり、この設定画面を開いたり、コードから選択したりすることはできません。ユーザーに自分で設定アプリから設定してもらう必要があります。

## iOS16以降での対応アプリ

`otpauth` のリンクは対応していないアプリでは開けません。設定画面で「次を使用して検証コードを設定:」選択肢に出てくるアプリのみ利用できます。

`otpauth` 対応アプリ

* パスワード(iOS設定アプリ)
* Google Authenticator
* Microsoft Authenticator
* Twilio Authy
* 1Password

`otpauth` 非対応アプリ

* Salesforce Authenticator(QRコードによる登録のみ)
* LastPass Authenticator(QRコード、シークレットキーによる登録が可能)

非対応アプリは、QRコードからの登録や、シークレットキーの入力による登録など、他の登録方法が必要です。

### 特定アプリを開く

`otpauth` を `apple-otpauth` に変更することで、設定アプリで選択されているアプリを無視して、パスワード(iOS設定アプリ)を開くことができます。

他のアプリの同様の方法は見つかりませんでした。

## 「パスワード」のTOTP発行機能

すでに「パスワード」に登録されているドメイン・アカウント名・パスワードのセットに紐つける形で設定できます。
他のTOTPアプリとは異なり、すでにあるアカウントに紐つけずにTOTP単体で設定することはできません。

bitbankさんの2段階認証のサポートページが参考になります。

[iOSの設定にあるパスワード機能が表示され、二段階認証が設定できない - bitbank](https://support.bitbank.cc/hc/ja/articles/4412605007257-iOS%E3%81%AE%E8%A8%AD%E5%AE%9A%E3%81%AB%E3%81%82%E3%82%8B%E3%83%91%E3%82%B9%E3%83%AF%E3%83%BC%E3%83%89%E6%A9%9F%E8%83%BD%E3%81%8C%E8%A1%A8%E7%A4%BA%E3%81%95%E3%82%8C-%E4%BA%8C%E6%AE%B5%E9%9A%8E%E8%AA%8D%E8%A8%BC%E3%81%8C%E8%A8%AD%E5%AE%9A%E3%81%A7%E3%81%8D%E3%81%AA%E3%81%84)

`otpauth` を開くと、すでに「パスワード」に登録されているアカウントの一覧が表示され、どのアカウントに紐つけるかの選択を促されます。
この時、`otpauth` リンクの `issuer` クエリに入っているドメインと同一のアカウントが優先的に表示されます。

ただし、`otpauth` を開いた時、必ずしもここにアカウントが登録されている状態にあるとは限りません。
保存されているのは、事前にユーザーがブラウザ等でログインなどを行い、ログイン時にパスワードを保存することを同意した場合のみです。
登録されていない場合は、ユーザーが+ボタンからドメイン、アカウント、パスワードを手動で登録する必要があり、ユーザーに親切とは言えません。

Webの場合は事前にログインをしたり、アプリの場合はAssociated Domainsを設定した上でログインフローを先に通すことで、ユーザーにアカウント情報を「パスワード」に登録することを促すことはできますが、登録するかどうかはユーザーの同意に委ねられるため、同様の問題が残ります。

[About the Password AutoFill workflow | Apple Developer Documentation](https://developer.apple.com/documentation/security/password_autofill/about_the_password_autofill_workflow)

## 結論

URL Scheme `otpauth` は認証アプリを直接開ける便利な方法ですが、サポートOSバージョンと、iOS標準の「パスワード」に対応するハードルがあります。

また、ユーザーが使いやすく利用するには、現状 `otpauth` 以外のQRコードやシークレットキーのコピペなどの方法を用意することが推奨されます。
