---
title: "SwiftUIで起動時の画面の出しわけ"
date: 2024-04-24
categories: ["Swift"]
tags: ["SwiftUI", "iOS", "Swift"]
---

SwiftUIを使用してアプリ起動時にユーザーステータスに応じて画面を表示し分ける個人的ベストプラクティスを説明します。

前提として、SwiftUIで画面遷移に使われるNavigationViewは親Viewから任意に画面遷移を管理したりするのが困難です。(iOS16~で使えるNavigationStackではある程度解決していますが、Viewの外から画面を書き換えるような構造でないことは変わりません)
そのため、ユーザーステータスなどに左右される画面遷移をコードで制御するのにNavigationViewを使うのはあまり適していません。

この記事は、起動時の画面出し分けと、起動後ユーザーの操作によって画面遷移していく場合に同じロジックや状態管理を使うための実装です。
NavigationViewは画面を単純に積み上げていく場合のみ使い、ユーザーステータスに応じて応じて画面を表示し分けるのは `@State` などによる表示内容の変更を使う、というのがメインの趣旨です。

## 最小実装

ユーザーがログインしているかどうかに基づき、メイン画面またはログイン画面を切り替えるというアプリを例に説明します。

### 全体構造

`ContentView` では、`@AppStorage` を使用してユーザーのログイン状態を管理します。`@AppStorage` は UserDefaults に保存され、アプリを閉じても値が保持されます。これにより、アプリが起動するたびにユーザーのログイン状態に基づいて適切な画面が表示されます。

```swift
struct ContentView: View {
    // @Stateと同じように変化するとViewの更新を発火する。
    // 裏側はUserDefaultsに保存されているので、アプリを閉じても残る
    @AppStorage("isLoggedIn") private var isLoggedIn = false

    var body: some View {
        if isLoggedIn {
            // ログイン中ならMainView表示
            NavigationStackOrView {
                MainView(isLoggedIn: $isLoggedIn)
            }
        } else {
            // ログアウト中ならLoginView表示
            NavigationStackOrView {
                LoginView(isLoggedIn: $isLoggedIn)
            }
        }
    }
}
```

注: `NavigationStackOrView` はiOS ~15では `NavigationView`、 iOS 16~では `NavigationStack` を使うようなラッパーです。

### ログイン・ログアウト処理

ログイン画面とメイン画面では、`@Binding` を使用して `isLoggedIn` フラグをバインドし、ログインボタンとログアウトボタンのアクションでこのフラグを切り替えることができます。

`isLoggedIn` は `@AppStorage` がついているので、 `@State` と同じように値の変化によりViewの変更が発火します。
ログインボタンやログアウトボタンで `isLoggedIn` を変更することで、起動時の画面出し分けと同じロジックで、メイン画面やログイン画面へ遷移することができます。

```swift
struct LoginView: View {
    @Binding var isLoggedIn: Bool

    var body: some View {
        // ログインボタンを押すとMainViewに遷移
        Button("ログイン") {
            // @Bindingで受け取っているので変更すると@AppStorageも変更される
            isLoggedIn = true
        }
    }
}

struct MainView: View {
    @Binding var isLoggedIn: Bool

    var body: some View {
        // ログアウトボタンを押すとLoginViewに遷移
        Button("ログアウト") {
            // @Bindingで受け取っているので変更すると@AppStorageも変更される
            isLoggedIn = false
        }
    }
}
```

## アニメーションの追加

`isLoggedIn` を変更することで、メイン画面やログイン画面へ遷移することができるのですが、この遷移は標準ではアニメーションがありません。
アプリ内でプッシュ遷移を主に使っている場合は、メイン画面に遷移するときだけアニメーションがないのは違和感があるでしょう。
`ContentView` に `.animation` と `.transition` を追加し、プッシュ遷移風のアニメーションを実現します。

```swift
struct ContentView: View {
    @AppStorage("isLoggedIn") private var isLoggedIn = false

    var body: some View {
        ZStack {
            if isLoggedIn {
                NavigationStackOrView {
                    MainView(isLoggedIn: $isLoggedIn)
                }
                .transition(pushTransition())
            } else {
                NavigationStackOrView {
                    LoginView(isLoggedIn: $isLoggedIn)
                }
                .transition(pushTransition())
            }
        }
        // animationを行う必要があるプロパティを指定
        .animation(.default, value: isLoggedIn)
    }
    /// プッシュ風のTransition
    private func pushTransition() -> AnyTransition {
        return .asymmetric(
            // 挿入時は右から移動アニメーション
            insertion: .move(edge: .trailing),
            // 削除時は左に1/3だけ移動するアニメーション
            removal: .offset(CGSize(width: -UIScreen.main.bounds.width / 2, height: 0))
        )
    }
}
```

`pushTransition()` は標準のプッシュ遷移のアニメーションを真似して作った自作の `AnyTransition` です。
iOS16〜のサポートですが、`push(from:)` という `AnyTransition` も用意されているので、そちらを利用する方法もあります。
(私が試した範囲では、自作の `pushTransition()` の方がプッシュ遷移のアニメーションに似ていたように思いますが)

[push(from:) | Apple Developer Documentation](https://developer.apple.com/documentation/SwiftUI/AnyTransition/push(from:))

### EnvironmentObjectで値の受け渡しをシンプルに

紹介した方法では、各画面に `isLoggedIn` のバインドを渡す必要があり、出し分けをする画面の種類が増えたり、ログアウトボタンなど `isLoggedIn` を変更するViewの階層が深くなった際にバインドの受け渡しが面倒です。

`EnvironmentObject` を使うと、アプリケーション全体の状態を管理し、異なるビュー間でデータを共有するのが簡単になります。親ビューから子ビューへとデータを直接継承することなく、必要なビューで直接データにアクセスできるようになります。

アプリのログイン状態を管理するために、例えば `AuthManager` というクラスを定義し、`isLoggedIn` をプロパティとして持たせることができます。これを `EnvironmentObject` として親View(今回は最も親であryContentView)に注入することで、その子Viewである全てのアプリのViewからアクセスできるようになります。

```swift
class AuthManager: ObservableObject {
    @AppStorage("isLoggedIn") var isLoggedIn = false
    // 永続化しない場合は
    // @Published var isLoggedIn = false
}

@main
struct MyApp: App {
    @StateObject private var authManager = AuthManager()

    var body: some Scene {
        WindowGroup {
            ContentView()
                .environmentObject(authManager)
        }
    }
}

struct ContentView: View {
    @EnvironmentObject var authManager: AuthManager

    var body: some View {
        if authManager.isLoggedIn {
            NavigationStackOrView {
                MainView()
            }
        } else {
            NavigationStackOrView {
                LoginView()
            }
        }
    }
}

struct LoginView: View {
    @EnvironmentObject var authManager: AuthManager

    var body: some View {
        Button("ログイン") {
            authManager.isLoggedIn = true
        }
    }
}

struct MainView: View {
    @EnvironmentObject var authManager: AuthManager

    var body: some View {
        Button("ログアウト") {
            authManager.isLoggedIn = false
        }
    }
}
```

この例では、`LoginView` や `MainView` で `AuthManager` を利用していますが、LoginViewの子Viewや、NavigationLink等で遷移した先(これも子Viewとして扱われます)であっても `AuthManager` を受け渡しなしで利用できます。

### まとめ

NavigationViewは画面を単純に積み上げていく場合のみ使い、ユーザーステータスに応じて応じて画面を表示し分けるのは `@State` などによる表示内容の変更を行いましょう。
