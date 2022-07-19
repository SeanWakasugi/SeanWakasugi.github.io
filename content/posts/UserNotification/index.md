---
title: "UserNotificationを使ってローカル通知をするシンプルなアプリを作成する"
date: 2022-07-12
categories: ["iOSアプリ"]
tags: ["Swift", "UIKit", "UserNotifications"]
---

`UserNotification`による簡単なiOSの通知ができるアプリケーションを作成しました。

<table>
<td><img src=./screenshot.png></td>
<td><img src=./screenshotnotified.png></td>
</table>

通知のテストボタンを押すと1秒後に通知が鳴り、  
日付と時間を設定して通知日時設定ボタンを押すと指定した時間に通知が鳴ります。

```
import UIKit
import UserNotifications

class ViewController: UIViewController {
    
    private let testNotificationButton = UIButton(type: .system)
    private let setNotificationButton = UIButton(type: .system)
    private let datePicker = UIDatePicker()
    
    private let stackView = UIStackView()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        // 通知の許可を得る
        let center = UNUserNotificationCenter.current()
        center.delegate = self
        center.requestAuthorization(options: [.alert, .sound], completionHandler: { (granted, error) in })
        
        view.addSubview(stackView)
        stackView.translatesAutoresizingMaskIntoConstraints = false
        stackView.axis = .vertical
        stackView.alignment = .center
        stackView.spacing = 20
        stackView.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor, constant: 100).isActive = true
        stackView.leadingAnchor.constraint(equalTo: view.safeAreaLayoutGuide.leadingAnchor, constant: 30).isActive = true
        stackView.trailingAnchor.constraint(equalTo: view.safeAreaLayoutGuide.trailingAnchor, constant: -30).isActive = true
        
        stackView.addArrangedSubview(testNotificationButton)
        testNotificationButton.setTitle("通知のテスト", for: .normal)
        testNotificationButton.configuration = UIButton.Configuration.plain()
        testNotificationButton.addTarget(self, action: #selector(testNotification(sender:)), for: .touchUpInside)
        
        stackView.addArrangedSubview(datePicker)
        datePicker.preferredDatePickerStyle = .inline
        
        stackView.addArrangedSubview(setNotificationButton)
        setNotificationButton.configuration = UIButton.Configuration.filled()
        setNotificationButton.setTitle("通知日時設定", for: .normal)
        setNotificationButton.addTarget(self, action: #selector(setNotification(sender: )), for: .touchUpInside)
    }
    
    /// 0.5秒後に通知
    @objc private func testNotification(sender: UIButton) {
        let trigger = UNTimeIntervalNotificationTrigger.init(timeInterval: 0.5, repeats: false)
        showNotification(trigger: trigger)
    }
    
    /// datePickerの日時に通知
    @objc private func setNotification(sender: UIButton) {
        // datePickerで取れるのはDate型なので、必要なパラメータをdateComponentsに変換
        let dateComponents = Calendar.current.dateComponents([.calendar, .year, .month, .day, .hour, .minute], from: datePicker.date)
        let trigger = UNCalendarNotificationTrigger.init(dateMatching: dateComponents, repeats: false)
        showNotification(trigger: trigger)
    }
    
    /// 通知の設定
    private func showNotification(trigger: UNNotificationTrigger) {
        let content = UNMutableNotificationContent()
        content.title = "Notification"
        content.body = "This is a notification"
        content.sound = UNNotificationSound.default
        let trigger = trigger
        let uuidString = UUID().uuidString
        let request = UNNotificationRequest.init(identifier: uuidString, content: content, trigger: trigger)
        let center = UNUserNotificationCenter.current()
        center.add(request) { (error) in
            guard let error = error else { return }
            print(error)
        }
        print(trigger)
    }
}

extension ViewController: UNUserNotificationCenterDelegate {
    /// アプリがフォアグラウンド時にも通知を表示する
    func userNotificationCenter(_ center: UNUserNotificationCenter, willPresent notification: UNNotification, withCompletionHandler completionHandler: @escaping (UNNotificationPresentationOptions) -> Void) {
        completionHandler([.banner, .sound])
    }
}
```

### triggerの付け方

上のコードで実際に通知を登録しているのは、以下の部分です。
```
let request = UNNotificationRequest.init(identifier: uuidString, content: content, trigger: trigger)
let center = UNUserNotificationCenter.current()
center.add(request) { (error) in }
```
`UNNotificationRequest`に通知内容`content`と通知の発生条件`trigger`を入れて、
`UNUserNotificationCenter.current()`に`add(request)`で登録しています。

この`trigger`にはいくつか種類があり、今回は`UNTimeIntervalNotificationTrigger`と`UNCalendarNotificationTrigger`を使用しています。


#### UNTimeIntervalNotificationTrigger

参考にしたサイト  
http://faboplatform.github.io/SwiftDocs/1.uikit/008_uinotification/

こちらのサイトのコードでは、即時と10秒後に通知を送るボタンを使っています。

`UNTimeIntervalNotificationTrigger.init(timeInterval: 0.5, repeats: false)`  
のtimeInterval(Double型)に入れた秒数後に通知がきます。timeIntervalに0を入れると怒られます。

  
#### UNCalendarNotificationTrigger

公式ドキュメント
https://developer.apple.com/documentation/usernotifications/uncalendarnotificationtrigger

`UNCalendarNotificationTrigger.init(dateMatching: dateComponents, repeats: false)`  
のdateMatching(DateComponents型)に入れた時間に通知が来ます。

具体的な時間の他に、特定の曜日なども入ります。

今回の実装では、`UIDatePicker.date`から取れるのは`Date`型だったため、`DateComponents`型に変換する必要がありました。

素直に変換してみます。

参考  
https://llcc.hatenablog.com/entry/2017/08/31/230000

```
let dateComponents = Calendar.current.dateComponents(in: TimeZone.current, from: datePicker.date)
```
このように変換すると、通知は届きませんでした。

この時、dateComponentsの中身は以下のようになります。

```
dateComponents: <NSDateComponents: 0x60000029c860> {
    Calendar: <CFCalendar 0x6000023a8dc0 [0x1dee09ae0]>{identifier = 'gregorian'}
    TimeZone: Asia/Tokyo (JST) offset 32400
    Era: 1
    Calendar Year: 2022
    Month: 7
    Leap Month: 0
    Day: 12
    Hour: 17
    Minute: 55
    Second: 0
    Nanosecond: 0
    Quarter: 0
    Year for Week of Year: 2022
    Week of Year: 29 // 年通算何周目か
    Week of Month: 3 // 月の何周目か
    Weekday: 3 // 1(日曜日)~7(土曜日)で表される曜日、3なら火曜日
    Weekday Ordinal: 2 // 第何火曜日か
```
欲しい`年/月/日 時:分`以外に、下の余分な情報が入っています。

逆に言えば、`何曜日`等の指定でも通知設定できるということですが、今回は条件が重複するせい?でおかしくなっています。

欲しいパラメータのみを設定することで正しく取得できました。

```
let dateComponents = Calendar.current.dateComponents([.calendar, .year, .month, .day, .hour, .minute], from: datePicker.date)
```

### フォアグラウンドの時の通知処理

デフォルトではそのアプリが前面にない時（バックグラウンド時）のみに通知がくるようになっています。

アプリの起動中（フォアグラウンド）でも通知がくるようにするためには、`UNUserNotificationCenterDelegate`を書く必要があります。

参考サイト  
https://dev.classmethod.jp/articles/wwdc-2016-user-notifications-8/

`UNUserNotificationCenter.current()`にdelegateを設定します。

今回は同じクラスのextensionに書いているので`=self`としています。

```
        let center = UNUserNotificationCenter.current()
        center.delegate = self
```

`UNUserNotificationCenterDelegate`は`userNotificationCenter(_willPresent:_)`の中に通知を表示する処理を書きます。

`completionHandler`に通知表示の処理が入っているので、それを呼び出すだけです。

```
extension ViewController: UNUserNotificationCenterDelegate {
    /// アプリがフォアグラウンド時にも通知を表示する
    func userNotificationCenter(_ center: UNUserNotificationCenter, willPresent notification: UNNotification, withCompletionHandler completionHandler: @escaping (UNNotificationPresentationOptions) -> Void) {
        completionHandler([.banner, .sound])
    }
}
```