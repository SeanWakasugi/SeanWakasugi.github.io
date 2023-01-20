---
title: "FlutterでiOSのネイティブ画面を表示する"
date: 2023-01-19
categories: ["Flutter"]
tags: ["Flutter", "iOS", "Swift"]
---

Flutterで作成したアプリから部分的にiOSで作成した画面を呼ぶ上で必要になったことをまとめました。

## Method Channel

Method Channelを利用して、Flutterからネイティブのメソッドを呼び出すことができます。

今回は、動画と説明ページを表示するための`VideoViewController`をネイティブで用意しました。

この`VideoViewController`にビデオのURLと説明ページのURLを引数として渡し、モーダルとして表示してみます。

参考
[DartからAndroid / iOS等のメソッドを呼び出したい（Flutter MethodChannel）](https://zenn.dev/mukkun69n/articles/7438bae5082ed5)

公式ドキュメント
[Writing custom platform-specific code](https://docs.flutter.dev/development/platform-integration/platform-channels)

### Flutter側

main.dart

```dart
import 'package:flutter/services.dart';

class _MyHomePageState extends State<MyHomePage> {

  Future showVideoScreen() async {
    MethodChannel methodChannel = const MethodChannel('jp.wakasugi.chatFlutter/main');
    try {
      final arguments = {
        'contentURL':
            'https://bitdash-a.akamaihd.net/content/MI201109210084_1/m3u8s/f08e80da-bf1d-4e3d-8899-f0f6155f6efa.m3u8',
        'infoURL': 'https://google.com/'
    };
      final String result =
        await methodChannel.invokeMethod('showVideoScreen', arguments);
      print(result);
    } on PlatformException catch (e) {
      print(e);
    }
  }

  @override
  Widget build(BuildContext context) {...
```

ネイティブのコードを呼びたいところで`showVideoScreen()`のメソッドを非同期で呼ぶと、ネイティブコードが実行されます。

`methodChannel = const MethodChannel('jp.wakasugi.chatFlutter/main');`でMethod Channelの識別用のStringを設定します。

一意である必要があるため、'アプリパッケージ名/チャンネル名'とするのが一般的とのことです。

`methodChannel.invokeMethod('showVideoScreen', arguments);`でMethod Channelを通して`'showVideoScreen'`のメソッドが呼び出されます。

`arguments`にはネイティブ側に渡したい引数を渡します。StringやDictionary( = DartのMap)で渡すことができます。

以下のURLから渡すときに使える型と変換について確認できます。

[codec](https://flutter.dev/docs/development/platform-integration/platform-channels#codec)

### iOSネイティブ側

AppDelegate.swift

```swift
import UIKit
import Flutter

@UIApplicationMain
@objc class AppDelegate: FlutterAppDelegate {
    private let methodChannelName = "jp.wakasugi.chatFlutter/main"
    
    private var flutterViewController: FlutterViewController {
        return self.window.rootViewController as! FlutterViewController
    }
    
    override func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?
    ) -> Bool {
        
        let methodChannel = FlutterMethodChannel(name: methodChannelName, binaryMessenger: flutterViewController.binaryMessenger)
        methodChannel.setMethodCallHandler { [weak self] methodCall, result  in
            if methodCall.method == "showVideoScreen" {
                let arguments = methodCall.arguments as! [String: String]
                self?.showVideoScreen(contentURL: arguments["contentURL"]!, infoURL: arguments["infoURL"]!)
            } else {
                result(FlutterError(code: "ErrorCode", message: "ErrorMessage", details: nil))
            }
        }
        GeneratedPluginRegistrant.register(with: self)
        return super.application(application, didFinishLaunchingWithOptions: launchOptions)
    }
    
    func showVideoScreen(contentURL: String, infoURL: String) {
        let video = UIStoryboard(name: "Video", bundle: nil).instantiateInitialViewController()! as! VideoViewController
        video.contentURL = URL(string: contentURL)
        video.infoURL = URL(string: infoURL)
        flutterViewController.present(video, animated: true)
    }
}
```
`application(.. didFinishLaunchingWithOptions ..)`内で`methodChannel.setMethodCallHandler(..)`でMethod Channelを受け取る場所を作成します。

Flutter側と

* Method Channelの名前
* メソッド名
* argumentsの型

この3つが正しければ、正常に受け取りができるかと思います。

どうしても型を手動でキャストしなければいけないのが微妙だなぁと思っていたら、↓のような方法もあるようです。

[【Flutter】型安全に MethodChannel でネイティブとデータ通信を行う](https://zenn.dev/kotaro666/articles/method-channel-with-pigeon)

### できたこと

どこから呼び出してもiOS側ではflutterViewControllerが実行していることになるので、ネイティブ側で複数画面を順番に呼び出したり、TabBarやNavigationControllerなどを使うこともできそうです。

今回は`present()`でモーダルを表示していますが、モーダルを閉じるとそのままFlutter側の画面に戻ってきて、Flutterで作った画面で動作が継続できました。通常のモーダルと同じように使えます。

## iOSとAndroid(やその他)で処理を変える

Method ChannelはiOSでもAndroidでも利用可能な方法ですが、

iOS → ネイティブ呼び出し

Android → Flutterのまま実行

という動作にしたい場合などは環境によって実行を変える必要があります。

```dart
if (Platform.isIOS) {
  // MethodChannelによる処理
} else {
  // Flutterによる処理
}
```

[【Flutter】実行環境がiOSかAndroidか判断](https://www.automation-technology.info/index.php/2021/07/10/post-359/)

Method Channelだけで実装するのではなく、Flutterでも実装しておくとAndroidだけではなくWebアプリやMacなどに展開する時にとりあえず動くので、良さそうです。