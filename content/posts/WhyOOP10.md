---
title: "オブジェクト指向でなぜつくるのか 10章感想"
date: 2022-10-25
categories: ["本まとめ"]
tags: ["オブジェクト指向でなぜつくるのか", "swift"]
---

この記事では、SwiftでiOSアプリのプログラミングをする初学者として、

「オブジェクト指向でなぜつくるのか 第3版 知っておきたいOOP、設計、アジャイル開発の基礎知識」

https://www.amazon.co.jp/dp/4296000187

の感想、学びになった点を書いていきます。

## 10章: 設計

### 保守性と再利用性

設計の目標は、

第一に、要求仕様通りに正しく動かすこと。

第二は、以前は実行効率だったが、現在は**保守性や再利用性**。

保守に強く再利用しやすいソフトウェアとは

1. 重複を排除
2. 部品の独立性を高める
3. 依存関係を循環させない

### 重複を排除
同じ部分をコピペではなく、関数、クラスやポリモーフィズムを使って共通化をすること

### 部品の独立性を高める
ソフトウェアを一枚岩にするのではなく、複数のサブシステム、部品から作ること  
さらにそれぞれが独立性が高いこと

凝集度: 個々の部品の機能のまとまり度合い。強いほど良い。  
結合度: 部品間の結びつき度合い。弱いほど良い。

#### 独立性を高めるコツ
一言で表現する名前をつける: 適切な名前をつけれるなら、そのクラスは機能がまとまっている。

秘密をたくさん作る: クラスが外部に公開する情報を最小限にする

小さく作る: 1つのクラスやメソッドに詰め込みすぎると全体として何をしているか理解できない。

擬人化: オブジェクトが役割を分担して、まるで人のように相互にメッセージを送り合うように設計する。

### 依存関係を循環させない
依存関係が循環していると、その部品は修正したり再利用するときに循環している部品全てを修正したりまとめて利用しなくてはならなくなる

依存関係を最低限で一方通行にすれば、独立した部品が作られて修正や再利用の影響範囲が狭まる

