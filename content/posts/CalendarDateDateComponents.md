---
title: "SwiftのDateとDateComponentsの違いの整理"
date: 2022-07-19
categories: ["Swift"]
tags: ["Swift"]
---

### `Date`と`DateComponents`の違い

`Date`はカレンダーやタイムゾーン情報を含まない絶対的な時間のことです。

`DateComponents`は年月日など単位のある構造体で、具体的な数字による時間を表します。

大まかに言えば、日本で2022/7/19 16:24の`Date`は、イギリスでは2022/7/19 07:24の`Date`になります。

2022/7/19 16:24の`DateComponents`は`year: 2022, month: 7, day: 19, hour: 16, minute: 24`というただの値のことです。


`DateComponents`は以下のようなinitを持っており、さまざまな情報を保つことができる構造体なのがわかると思います。

カレンダーもタイムゾーンも持つことはできますが、持っていないこともあります。
```swift
public init(
    calendar: Calendar? = nil, 
    timeZone: TimeZone? = nil, 
    era: Int? = nil, 
    year: Int? = nil, 
    month: Int? = nil, 
    day: Int? = nil, 
    hour: Int? = nil, 
    minute: Int? = nil, 
    second: Int? = nil, 
    nanosecond: Int? = nil, 
    weekday: Int? = nil, 
    weekdayOrdinal: Int? = nil, 
    quarter: Int? = nil, 
    weekOfMonth: Int? = nil, 
    weekOfYear: Int? = nil, 
    yearForWeekOfYear: Int? = nil
)
```

### `Calendar`と`Timezone`

`Calendar`により、カレンダー、例えば西暦とか和暦とかを指定できます。

```swift
let seireki = Calendar(identifier: .gregorian) // 西暦カレンダー
let wareki = Calendar(identifier: .japanese) // 和暦カレンダー
let tanmatsu = Calendar.current // 端末のカレンダー設定に依存
```

`Date`は`Calendar`が関係ありません。

なぜなら、西暦2022年7月19日でも令和4年7月19日でもどちらも同じ時間のことを指していますから、どちらも同じものとして扱えるからです。

`TimeZone`は、地域による時差を指定できます。

上で言った日本とかイギリスとかの時間の違いは`TimeZone`による違いです。

`Calendar.timeZone`に入れることで特定のカレンダーの特定のタイムゾーンを指定できます。

```swift
var japanCalendar = Calendar(identifier: .japanese)
japanCalendar.timeZone = TimeZone(identifier: "JST")!

var ukCalendar = Calendar(identifier: .gregorian)
ukCalendar.timeZone = TimeZone(identifier: "GMT")!

var currentCalendar = Calendar.current
currentCalendar.timeZone = TimeZone.current // これは指定しなくても端末のタイムゾーン設定に依存する
```


### `Date`と`DateComponents`の変換


`Date`は`Calendar`と`Timezone`を指定することで`DateComponents`を取り出すことができます。

絶対的な時間`Date`にカレンダーとタイムゾーンを指定して、そのカレンダーのその地点の時間を切り出して`DateComponents`に入れているイメージです。

```swift
let date = Date() // Date()で現在の時間が入る
print(date) // 2022-07-19 08:20:30 +0000

var japanCalendar = Calendar(identifier: .japanese)
japanCalendar.timeZone = TimeZone(identifier: "JST")!
let dateComponents = japanCalendar.dateComponents([.year, .month, .day, .hour, .minute], from: date)
print(dateComponents) // year: 4 month: 7 day: 19 hour: 17 minute: 20 isLeapMonth: false
```
逆に、`DateComponents`の値に`Calendar`と`Timezone`を指定して`Date`に変換もできます。

`DateComponents`の表す時間が、どのカレンダーのどこのタイムゾーンかを教えてあげることで、絶対時間を計算して`Date`にしてくれるイメージです。

```swift
let dateComponents = DateComponents(year: 4, month: 7, day: 19, hour: 16, minute: 24)
print(dateComponents) // year: 4 month: 7 day: 19 hour: 16 minute: 24 isLeapMonth: false

var japanCalendar = Calendar(identifier: .japanese)
japanCalendar.timeZone = TimeZone(identifier: "JST")!
let date = japanCalendar.date(from: dateComponents)!
print(date) // 2022-07-19 07:24:00 +0000
```

