---
title: "リーダブルコード1章、2章まとめ"
date: 2022-07-11
categories: ["本まとめ"]
tags: ["リーダブルコード"]
---

## 1章: 理解しやすいコード

読みやすさの基本定理

**コードは他の人が最短時間で理解できるように書かなければいけない。**

これが全て。

## 2章: 名前に情報を詰め込む

まずは命名の話。

### 明確な単語を選ぶ 
getは明確ではない。
どこからgetするかがわからないから。  
fetchなら、インターネットから手に入れるのだとわかる。


sizeは何のサイズを指すのか明確ではない。  
HeightとかNumNodesとかMemoryBytesとかに変えれば、何のサイズなのかが明確になる。


Stopという言葉もどう止まるのかが明確じゃない。  
Kill（取り消しができない重い操作）を使ったり、
Pause（後からResumeできる操作）を使ったりすると、明確になる。


捉え方が複数ある単語を使う前に、シソーラスで調べてより合った言葉がないか探すこと。


### 汎用的な名前を避ける(あるいは、使う状況を選ぶ) 

retvalなど汎用的な命名を避けて、変数の値や目的を示す変数名へ変えた方がいい。

汎用的な名前も悪いわけではなく、例えば一時的な保管が目的で生存期間が短いならtmpでも良い。

i, j, kなどもイテレータであることがわかりやすくて良いが、イテレータが複数あるときは説明的な名前を付けると良い。例えば、clubs_i, members_jなど。

少しでも時間を使っていい名前を考える習慣を。

### 抽象的な名前よりも具体的な名前を使う
名前の強さよりも、許可していないものを明確にする方が大事。  
DISALLOW_EVIL_CONSTRUCTORS → DISALLOW_COPY_AND_ASSIGN 

—-run-locallyがローカルのデータベースを使って、かつデバッグログを出してもらうためのオプションなら、  
—-use-local-databaseと—-extra-loggingで分けてわかりやすくする。

### 接尾辞や接頭辞を使って情報を追加する 

値の単位を入れる
* delay_secs
* max_kbps

重要な属性を追加する
* plaintext_password: 暗号化前のパスワード
* unescaped_comment: 表示する前にエスケープする必要があるcomment

### 名前の長さを決める 
名前は短ければ短いほどいいわけでも、長く情報を詰め込めばいいわけでもない。  
状況ごとに適切な長さを設定して命名する必要がある。

スコープの大きさ（その名前が使われる範囲の広さ）で長さを決めると良い。  
例えば、スコープが小さければ短い簡潔な名前でも良い。
なぜなら、その変数を使う場所が近く何の変数かすぐわかるから。

省略語は構わないがプロジェクト固有の省略後はだめ。
新しいチームメイトが名前の意味を理解できるかで判断する。  
BackEndManger → BEManger はだめで、
string → strは良い。

不要な単語を捨てたほうが良い。  
ConvertToString → ToString

### 名前のフォーマットで情報を伝える 
クラス名はCamelCaseで変数名はlower_separatedのようなルールを持つことで、一目でそれが何なのかわかる。

言語ごと、プロジェクトごとに決まりが違うので、プログラム、チーム内で一貫性を持つことが大事。