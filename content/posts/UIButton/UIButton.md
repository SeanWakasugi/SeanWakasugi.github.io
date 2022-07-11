---
title: "Storyboardで作成できるUIButtonと同じものをコードで作る"
date: 2022-07-11
categories: ["UIKit"]
tags: ["Storyboard"]
---

Storyboard上でボタンを設置しようとすると、以下のような選択肢がある。

![StoryboardのButton選択肢](./buttonsInStroyboard.png)

{{<figure src="./buttonsInStoryboard.png" alt="StoryboardのButton選択肢" width="75%">}}

![StoryboardのButton選択肢](./buttonsInStroyboard.jpg)

{{<figure src="./buttonsInStoryboard.jpg" alt="StoryboardのButton選択肢" width="75%">}}

{{<figure src="./image.jpeg" alt="美ヶ原からの景色" width="75%">}}

これらと同じものをコードで実装しようとすると少しずつ違うものができがちである。

StroyboardのデフォルトはAppleの推奨であると(私は)考えているので、Storyboardのデフォルトボタンをコードで実装しながらAppleの推奨のボタンの実装方法について考えたい。

### UIButtonのstyle