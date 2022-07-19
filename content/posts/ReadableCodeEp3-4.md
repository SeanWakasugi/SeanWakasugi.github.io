---
title: "リーダブルコード3章、4章まとめ"
date: 2022-07-19
categories: ["本まとめ"]
tags: ["リーダブルコード"]
---

## 3章: 誤解されない名前

積極的に「誤解」を探して曖昧な名前を変えると良い。

複数の名前を検討して、何が起きるかを明確に表していてかつ誤解を生む可能性が低い言葉を選ぶといい。

filter()  
選択するのか除外するのか分かりにくい  
選択する→select()  
除外する→exclude()

限界値を含めるときはminとmaxを使う。  
CART_TOO_BIG_LIMITでは、条件がCART_TOO_BIG_LIMIT未満ならOKなのか以下ならOKなのかわからない。MAX_ITEMS_IN_CARTなら以下であると分かる。

似た規則
* min <= x <= max
* first <= x <= last
* begin <= x < end

ブール値は、is, has, can, shouldを先頭につけるとわかりやすい。  
否定系は混乱しやすいので肯定系に。  
disable→use

get~は軽量な動作を想起させるので、処理が大きいならcomputeなどにした方がいい。


## 4章: 美しさ
美しい、目に優しい、読みやすいソースコードを作るための3つの原則

* 読み手が慣れているパターンと一貫性のあるレイアウトを使う。
* 似ているコードは似ているように見せる。
* 関連するコードをまとめてブロックにする。


似ているコードは、改行を使って同じ「シルエット」にすると似ている処理であることが一目でわかる。

重複したり複雑な処理はメソッドにして分離することで、コードが簡潔になり、何のための処理であるかなどの大切な部分が見やすくなる。

列を揃える
```
CheckFullName("Doug Adams", "Mr. Douglas Adams", ""); 
CheckFullName(" Jake Brown ", "Mr. Jacob Brown III", ""); 
CheckFullName("No Such Guy", "", "no match found"); 
CheckFullName("John", "", "more than one result"); 
```
上を、下のように整形すると読みやすい。
```
CheckFullName("Doug Adams",     "Mr. Douglas Adams",    "");
CheckFullName(" Jake Brown ",   "Mr. Jake Brown III",   "");
CheckFullName("No Such Guy" ,   "" ,                    "no match found");
CheckFullName("John" ,          "" ,                    "more than one result");
```
（という意図らしいが、できればやりたくないと思った。
見やすいのは確かだが、保守コストとかが高すぎる気がしている。）

どんな順番でも処理が変わらないようなものでも、意味のある順番を選んで、常にその順番を守ること。例えば使う順番とか、出力する時の並び順とか。

大きな処理の塊を空の行やコメントで区切って書くこと。