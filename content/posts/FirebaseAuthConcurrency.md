---
title: "Firebase Auth MFA x Swift Concurrency"
date: 2022-12-27
categories: ["Swift"]
tags: ["Swift", "Firebase"]
---

2022年7月にFirebase Authenticationで多要素認証(MFA)が利用可能になりました。

また、2022年5月のGoogle I/OでFirebaseはSwift Concurrencyに完全対応することが発表されています。

> We’re updating Firebase to fully embrace the modern Swift language
[https://firebase.blog/posts/2022/05/whats-new-at-google-io](https://firebase.blog/posts/2022/05/whats-new-at-google-io)

つまり、Firebaseでの多要素認証をasync/awaitを利用してシンプルに書くことが可能になりました。

しかし、多要素認証のチュートリアルは依然としてcompletionを利用したものになっています。

[https://firebase.google.com/docs/auth/ios/multi-factor](https://firebase.google.com/docs/auth/ios/multi-factor)

これをasync/await、そしてthrowを用いて書き直します。

Firebaseバージョンは10.2.0です。


## アカウント登録

Firebaseコンソールから多要素認証をオンにするのを忘れずに。

### メールアドレスとパスワードの登録

通常のアカウント作成と同じです。
アカウントを作成し、メールアドレス確認用メールを送信します。

```swift
func register(email: String, password: String) async throws {
    let result = try await Auth.auth().createUser(withEmail: email, password: password)
    try await result.user.sendEmailVerification()
}
```

`sendEmailVerification(with: )`を使って、メールでの確認後にアクションコードを走らせることができます（メールでの確認後にアプリに戻ってくる等）

このメソッドが正常に終了すると、ログイン状態となります。

### 多要素認証用の電話番号の登録

電話番号の登録には、以下の3つの条件が必要です。
1. ログイン状態であること(= `Auth.auth().currentUser`がnilではないこと)
2. メールアドレスが確認されていること(= `Auth.auth().currentUser.isEmailVerified`がtrueであること)
3. 10分以内に認証されていること(ログインから10分経過していなければOK)

メアド・パスワードの登録をした直後に電話番号登録をする場合は、メールアドレスの確認をユーザーに促しましょう。
メールアドレスが確認されたことをプッシュ等で受け取ることはできないので、適当なタイミングで`Auth.auth().currentUser.reload`で更新して`.isEmailVerified`を確認するのが良いです。

また、10分以内のログインが必要なため、電話番号登録直前に再度ログインさせるのが無難です。
ログインから時間が経過しすぎると以下の`verifyPhoneNumber(...)`を行った際にエラーとして`.requiresRecentLogin`が返ってくるので、それを拾って再ログインに誘導することもできます。

電話番号の登録は以下のメソッドです。入力した電話番号にSMSコードを送信するところまでやってくれます。
返り値は`verificationID`として次の認証コードの確認に利用します。

```swift
func registerPhone(phoneNumber: String) async throws -> String {
    let session = try await Auth.auth().currentUser?.multiFactor.session()
    return try await PhoneAuthProvider.provider().verifyPhoneNumber(phoneNumber, uiDelegate: nil, multiFactorSession: session)
}
```

### 電話番号について

上の`PhoneAuthProvider.provider().verifyPhoneNumber(phoneNumber, uiDelegate: nil, multiFactorSession: session)`で使う`phoneNumber`についてです。

この電話番号は、`+819012345678`のような形式の文字列で入力しなくてはいけません。国番号付きの国際電話番号です。

確実に国内のみでしか使えないサービスであれば、ユーザーの入力した電話番号に`+81`をつけるだけでもいいのですが、国際対応するなら国番号の取得が必要になります。

[SKCountryPicker](https://github.com/SURYAKANTSHARMA/CountryPicker)というライブラリにUIまで含めて国番号選択をやってもらいました。
詳細は省きますが、選択された`country: Country`を取得して、`country.dialingCode`を取ると、`+81`のような文字列が取れます。

```swift
if phoneNumber.text!.first! == "0" {
    phoneNumber.removeFirst()
}
let verificationId = try await registerPhone(phoneNumber: country.dialingCode! + phoneNumber)
```

なお、`+819012345678`でも`+8109012345678`でも一応`verifyPhoneNumber(...)`は動きました（検証数少なめ）。

### 認証コードの確認

`verifyPhoneNumber(...)`でSMSに認証コードが送信されるので、ユーザーにそれを入力してもらいます(`verificationCode`)。
`verificationID`は`verifyPhoneNumber(...)`の返り値です。

```swift
func verifyCodeWhileRegistration(verificationID: String, verificationCode: String) async throws {
    let credential = PhoneAuthProvider.provider().credential(withVerificationID: verificationID, verificationCode: verificationCode)
    let assertion = PhoneMultiFactorGenerator.assertion(with: credential)
    try await Auth.auth().currentUser?.multiFactor.enroll(with: assertion, displayName: "phone number")
}
```
`Auth.auth().currentUser?.multiFactor.enroll(with: assertion, displayName: "phone number")`の`displayName`は、多要素認証の認証方法の名前です。
複数の多要素認証の登録を可能にする場合は、ユーザーにこの名前を設定させることでログイン時にどの方法で認証するか選択しやすくなります。
登録後は、`Auth.auth().currentUser?.multiFactor.enrolledFactors[0].displayName`で登録した名前が取れます。


## ログイン

### メールアドレスとパスワードの入力

Firebase Authenticationでは、すべてのアカウントで多要素認証必須、という設定はできません。
もし全てのアカウントで多要素認証必須としたければ、アプリ側で多要素認証が有効でないアカウントをブロックしたり、多要素認証登録のフローに誘導したりする必要があります。

下のコードでは、多要素認証設定済みかどうかで`enum LoginType`を返すようにしています。

```swift
enum LoginType {
    case noMultiFactor
    case withMultiFactor(resolver: MultiFactorResolver)
}

func login(email: String, password: String) async throws -> LoginType {
    do {
        try await Auth.auth().signIn(withEmail: email, password: password)
        guard isEmailVerified() else {
            throw AuthErrorCode.Code.unverifiedEmail
        }
        // 多要素認証が未設定のアカウント
        return .noMultiFactor
    } catch let error as NSError {
        switch AuthErrorCode(_nsError: error).code {
        case .secondFactorRequired:
            // 多要素認証設定済みのアカウント
            let resolver = error.userInfo[AuthErrorUserInfoMultiFactorResolverKey] as? MultiFactorResolver
            return .withMultiFactor(resolver: resolver!)
        case .unverifiedEmail:
            // ログイン時にメールアドレスの確認が済んでいなかった場合はメールアドレス確認メールを再送信
            try await Auth.auth().currentUser?.sendEmailVerification()
            throw error
        default:
            throw error
        }
    }
}
```

多要素認証が一度登録されると、多要素認証なしではログインできなくなります。
`signIn(...)`を実行すると、エラーとして`.secondFactorRequired`がthrowされます。
エラーから`MultiFactorResolver`を生成して、次の認証コードの確認に利用します。

### 認証コードの送信

登録時とは違い、自動で認証コードSMSは送信されません。
先述の`MultiFactorResolver`を引数に、SMSを送信します。
返り値は、`verificationID`です。

```swift
func sendSMSWhileLogin(resolver: MultiFactorResolver) async throws -> String {
    let multiFactorInfo = resolver.hints[0] as? PhoneMultiFactorInfo
    return try await PhoneAuthProvider.provider().verifyPhoneNumber(with: multiFactorInfo!, uiDelegate: nil, multiFactorSession: resolver.session)
}
```
多要素認証方法を複数登録している場合は、`resolver.hints[0]`のindexでどの方法を利用するか指定できます。
それぞれのhintに対し、下4桁のみの電話番号`phoneNumber`、登録時に入力した`displayName`や登録日`enrollmentDate`を取得できるので、選択肢としてこれらを表示して選んでもらうことができます。

### 認証コードの確認

ユーザーに認証コード`verificationCode`を入力してもらい、確認します。
メールアドレスとパスワード入力時に取得できる`MultiFactorResolver`とSMS送信時に取得できる`verificationID`の両方が必要です。

```swift
func verifyCodeWhileLogin(resolver: MultiFactorResolver, verificationID: String, verificationCode: String) async throws {
    let credential = PhoneAuthProvider.provider().credential(withVerificationID: verificationID, verificationCode: verificationCode)
    let assertion = PhoneMultiFactorGenerator.assertion(with: credential)
    try await resolver.resolveSignIn(with: assertion)
}
```

これで多要素認証を用いたログイン完了です。

次回は、多要素認証を導入するにあたって修正が必要になったことを書きます。