---
title: "オブジェクト指向でなぜつくるのか 4章感想"
date: 2022-09-27
categories: ["本まとめ"]
tags: ["オブジェクト指向でなぜつくるのか", "swift"]
---

この記事では、SwiftでiOSアプリのプログラミングをする初学者として、

「オブジェクト指向でなぜつくるのか 第3版 知っておきたいOOP、設計、アジャイル開発の基礎知識」

https://www.amazon.co.jp/dp/4296000187

の感想、学びになった点を書いていきます。

## オブジェクト指向の3つの仕組み

オブジェクト指向の基礎は知っていたので、この本で改めて知ったことや整理できたことを書いていきます。

### クラス

一般的にはオブジェクト指向の仕組みとしてはカプセル化が挙げられますが、著者としてはそれはクラスの1つの特徴に過ぎないので、"クラス"としているそうです。

クラスとは、まとめて隠して（カプセル化）、たくさん作る（インスタンス化）仕組み。

### ポリモーフィズム

共通のメインルーチンを作る仕組み。

共通のメインルーチンという言葉がしっくりこなかったので、調べました。

プログラムでは、メインルーチンがサブルーチンを呼び出す、という構造を取ります。

同じサブルーチンを何回も使うような場合、共通サブルーチンを使って効率化を図ります → `関数`

では、同じメインルーチンを何回も使うためには、共通メインルーチンを作ります → `ポリモーフィズム`

全く異なるメインルーチンからでも、共通内容を呼び出せる`関数`に対して、  
全く異なるサブルーチンでも、共通の呼び出し方で呼び出せるのが`ポリモーフィズム`です。

### 継承

クラスの共通部分を別クラスにしてまとめる仕組み。

## 進化したオブジェクト指向の仕組み

### 型
クラスを型として使うことができる。

型を使うメリット（オブジェクト指向に限らず）

* メモリの確保領域が確定する
* プログラムのエラーを防止する

エラー防止の型チェックをいつ行うかによって2つに分けられる。

静的型付け  
コンパイル時点で型チェックをする

動的型付け  
プログラム実行時に型チェックをする

Swiftは静的型付けの言語だが、Objective-Cは動的型付けの言語という大きな差がある。

### 例外
メソッドにおけるエラー時の特別な処理。

Swiftにも`try-catch`と呼ばれる例外処理がありますが、メリットについて整理しきれていなかったので、勉強になりました。

通常メソッド（関数）は処理が終わると戻り値を返しますが、その代わりに別の形式でエラーを返すのが例外処理です。

メリット
* エラーの受け取りを書き忘れることがない  
  例外を処理するコードを書かないとプログラムがエラーになってくれる
* 連鎖するメソッドで簡単にエラーを受け渡すことができる  
  エラーの処理を行わずにより上位のメソッドに伝えるだけなら、例外が発生することを明示する(`throws`)だけで良い

### ガベージコレクション

自動でインスタンスをメモリから削除してくれる機能。

参照が0になると消える（詳細は5章の感想で）

メモリリーク問題を解決してくれる。