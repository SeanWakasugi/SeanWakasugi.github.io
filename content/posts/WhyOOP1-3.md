---
title: "オブジェクト指向でなぜつくるのか 1-3章感想"
date: 2022-09-27
categories: ["本まとめ"]
tags: ["オブジェクト指向でなぜつくるのか", "swift"]
---

この記事では、SwiftでiOSアプリのプログラミングをする初学者として、

「オブジェクト指向でなぜつくるのか 第3版 知っておきたいOOP、設計、アジャイル開発の基礎知識」

https://www.amazon.co.jp/dp/4296000187

の感想、学びになった点を書いていきます。

## オブジェクト指向が現れた歴史的経緯

私はプログラミングを初めて日が浅く、扱える言語も少ないため3章のプログラミング言語に関する記述がとても勉強になりました。

理解を軽くまとめてみます。

#### 「人間に親しみやすい言語を」の進化

機械語 → アセンブリ言語 → 高級言語(FORTRAN, COBOL)

##### 高級言語の課題
* ソフトウェアの需要が爆発的に増加した
* プログラムの寿命が長くなった（毎回新たに作るのではなく、修正を加えながら長く作るようになった）

→ プログラミングを理解しやすく、修正しやすくしたい

#### 「保守性の高い言語を」の進化

##### 構造化言語(C言語)
* 構造化プログラミング  
  GOTO文を使わず「順次実行」「条件分岐」「繰り返し」の基本3構造だけで表現する
* ローカル変数と引数の値渡しの登場  
  グローバル変数は影響範囲が広く修正しづらかったので、範囲が限定的なローカル変数を用いるようになった

##### 構造化言語の課題
* グローバル変数問題  
  複数のサブルーチンで利用する場合はグローバル変数が必要だったが、影響範囲が広いため、修正しづらい
* 貧弱な再利用  
  再利用できるのはサブルーチンだけだった。

→ さらに修正しやすく、再利用できる言語へ

#### オブジェクト指向言語へ

##### オブジェクト指向言語(Javaなど)
* クラス
* ポリモーフィズム
* 継承

オブジェクト指向言語の詳細は4書の記事で。